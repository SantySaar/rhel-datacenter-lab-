# Lab Journal

Diario de errores reales encontrados durante el despliegue del lab y cómo se resolvieron. Documentado como referencia propia y como prueba de troubleshooting real sobre Rocky Linux 9 + Ansible.

---

## Día 1 — 21/04/2026
**Tiempo dedicado:** 6 horas
**Avances:**
- Lab levantado de cero a producción
- 3 VMs Rocky Linux 9 desplegadas con Vagrant
- Los 11 roles Ansible ejecutados en verde (`failed=0`)
- Portfolio web, HAProxy stats, contenedor Podman y todos los servicios verificados manualmente
- Screenshots capturados y subidos al repo

**Errores encontrados:**

### 1. Disco VDI huérfano bloquea `vagrant up`
- **Error:** `VBoxManage.exe: error: VDI: cannot create image 'web-sdb.vdi' (VERR_ALREADY_EXISTS)`
- **Causa:** Un intento previo de `vagrant up` se cortó a medias. El disco `web-sdb.vdi` quedó registrado en VirtualBox aunque el fichero ya no existía en disco. Vagrant intentaba crearlo de nuevo y VirtualBox lo rechazaba.
- **Solución:**
```bash
  VBoxManage list hdds                          # localizar el disco inaccessible
  VBoxManage unregistervm <UUID_VM> --delete    # borrar la VM huérfana
  VBoxManage closemedium disk <UUID> --delete   # desregistrar el disco
  vagrant up web                                # recrear limpiamente
```

### 2. `ansible.cfg` ignorado por permisos world-writable en /vagrant
- **Error:** `[WARNING]: Ansible is being run in a world writable directory (/vagrant), ignoring it as an ansible.cfg source`
- **Causa:** La carpeta compartida `/vagrant` (sincronizada con Windows) tiene permisos 777, y Ansible por seguridad no lee `ansible.cfg` ni inventario desde rutas world-writable.
- **Solución:** copiar el proyecto a la home del usuario vagrant dentro de la VM control:
```bash
  cp -r /vagrant ~/rhel-datacenter-lab
  cd ~/rhel-datacenter-lab
```

### 3. Clave SSH inexistente al ejecutar `ansible all_managed -m ping`
- **Error:** `no such identity: /home/vagrant/.vagrant.d/insecure_private_key: No such file or directory`
- **Causa:** El inventario referenciaba la clave SSH de Vagrant del host Windows, que no existe dentro de la VM control.
- **Solución:** generar una clave propia en control, distribuirla a web y services con `ssh-copy-id` (usando `sshpass` con la contraseña por defecto `vagrant`), y actualizar el inventario:
```bash
  ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
  sshpass -p vagrant ssh-copy-id -o StrictHostKeyChecking=no vagrant@192.168.56.20
  sshpass -p vagrant ssh-copy-id -o StrictHostKeyChecking=no vagrant@192.168.56.30
  sed -i 's|~/.vagrant.d/insecure_private_key|~/.ssh/id_ed25519|' inventory/hosts.yml
```

### 4. Apache httpd no arranca — certificado `localhost.crt` inexistente
- **Error:** `SSLCertificateFile: file '/etc/pki/tls/certs/localhost.crt' does not exist or is empty`
- **Causa:** El rol `webserver` genera un certificado autofirmado en `/etc/pki/tls/certs/lab.crt`, pero el fichero `/etc/httpd/conf.d/ssl.conf` por defecto de mod_ssl sigue apuntando a `localhost.crt`. Apache carga ambos y el ssl.conf por defecto rompe el arranque.
- **Solución:** añadir una tarea al rol que reemplace las rutas del ssl.conf por defecto para que apunten al certificado real del lab:
```yaml
  - name: Update default ssl.conf to use lab certificate
    ansible.builtin.replace:
      path: /etc/httpd/conf.d/ssl.conf
      regexp: "{{ item.old }}"
      replace: "{{ item.new }}"
    loop:
      - { old: '^SSLCertificateFile /etc/pki/tls/certs/localhost.crt', new: 'SSLCertificateFile /etc/pki/tls/certs/lab.crt' }
      - { old: '^SSLCertificateKeyFile /etc/pki/tls/private/localhost.key', new: 'SSLCertificateKeyFile /etc/pki/tls/private/lab.key' }
    notify: restart httpd
```

### 5. HAProxy no puede bindear al puerto 8404 (SELinux)
- **Error:** `Binding [/etc/haproxy/haproxy.cfg:28] for proxy stats: cannot bind socket (Permission denied) for [0.0.0.0:8404]`
- **Causa:** SELinux en modo enforcing solo permite a HAProxy bindear a puertos conocidos. 8404 no está en esa lista.
- **Solución:** activar el booleano `haproxy_connect_any` que permite a HAProxy escuchar en cualquier puerto:
```yaml
  - name: Allow HAProxy to bind to any port (SELinux)
    ansible.posix.seboolean:
      name: haproxy_connect_any
      state: true
      persistent: true
```

### 6. `podman-compose` no disponible en los repos de Rocky 9
- **Error:** `No package podman-compose available`
- **Causa:** Rocky Linux 9 no incluye `podman-compose` en sus repos base. Además, el rol no lo necesita porque usa `podman generate systemd` para la integración con systemd, no Compose.
- **Solución:** quitar `podman-compose` de la lista de paquetes del rol `podman`.

### 7. Pull de imagen desde Docker Hub cuelga la VM
- **Error:** `dial tcp 172.64.66.1:443: connect: connection refused` (Cloudflare CDN rechaza conexiones)
- **Causa:** Rate limiting agresivo de Cloudflare contra la IP pública de la conexión al intentar descargar `docker.io/library/nginx:alpine`.
- **Solución:** usar una imagen equivalente desde Quay.io (registro de Red Hat, más fiable para entornos RHEL):
```yaml
  image: quay.io/nginx/nginx-unprivileged:stable-alpine
  ports: "8090:8080"  # nginx-unprivileged escucha en 8080 internamente, no en 80
```

### 8. "Empty reply from server" con Podman rootless + pasta
- **Error:** `curl: (52) Empty reply from server` al acceder al contenedor, aunque el puerto 8090 estaba escuchando.
- **Causa:** El networking rootless de Podman usa `pasta` por defecto en Rocky 9, y tiene un bug al enrutar peticiones externas hacia el contenedor.
- **Solución:** forzar el uso de `slirp4netns` en la definición del contenedor:
```yaml
  network: slirp4netns
```

### 9. Variables del Vault no cargadas por Ansible
- **Error:** `'vault_mariadb_root_password' is undefined`
- **Causa:** El fichero `inventory/group_vars/vault.yml` no lo carga Ansible automáticamente porque espera nombres de grupos (`all.yml`, `webservers.yml`, etc.), no ficheros arbitrarios.
- **Solución:** convertir `group_vars/all.yml` en una carpeta que contenga `vars.yml` (variables normales) y `vault.yml` (secretos):
```bash
  mkdir -p inventory/group_vars/all
  mv inventory/group_vars/all.yml inventory/group_vars/all/vars.yml
  mv inventory/group_vars/vault.yml inventory/group_vars/all/vault.yml
```

### 10. MariaDB root password falla en ejecuciones repetidas del playbook
- **Error:** `Access denied for user 'root'@'localhost' (using password: NO)` en la tarea "Set MariaDB root password"
- **Causa:** Tras la primera ejecución, root ya tiene password puesto. El módulo `mysql_user` intenta conectarse sin credenciales y falla, rompiendo la idempotencia del playbook.
- **Solución:** desplegar `/root/.my.cnf` con las credenciales antes de cualquier operación MariaDB. Así root siempre puede autenticarse tanto en la primera ejecución como en las siguientes:
```yaml
  - name: Deploy root .my.cnf for passwordless admin login
    ansible.builtin.copy:
      dest: /root/.my.cnf
      content: |
        [client]
        user=root
        password={{ mariadb_root_password }}
      mode: '0600'
```

### 11. Redirect HTTP→HTTPS rompe con port forwarding de Vagrant
- **Error:** En el navegador, `http://localhost:8080` devolvía `ERR_SSL_PROTOCOL_ERROR` tras redirigir a HTTPS.
- **Causa:** El vhost hacía `RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]`. Con el port forwarding de VirtualBox (host:8080 → guest:80), `HTTP_HOST` llega como `localhost:8080`, y el redirect manda al navegador a `https://localhost:8080/` — un puerto HTTP hablando HTTPS → error SSL. Además Chrome cachea el 301 con HSTS, perpetuando el fallo.
- **Solución:** quitar el redirect automático del vhost *:80 y servir HTTP directamente. El usuario accede manualmente a `https://localhost:8443` si quiere TLS. En Chrome, limpiar HSTS en `chrome://net-internals/#hsts`.

**Aprendizajes clave:**
- VirtualBox mantiene un registro interno de discos independiente del filesystem; limpiar un `.vdi` huérfano requiere `VBoxManage closemedium`, no solo `rm`.
- Ansible ignora `ansible.cfg` en directorios world-writable por seguridad; siempre trabajar fuera de `/vagrant` dentro de las VMs.
- En Rocky 9, preferir `quay.io` sobre `docker.io` para imágenes de contenedores — menos rate limits y más estable.
- `slirp4netns` es más fiable que `pasta` para networking rootless en Podman 4.x.
- La estructura correcta para Vault + vars normales es `group_vars/<grupo>/vars.yml` + `group_vars/<grupo>/vault.yml`, no ficheros sueltos.
- `/root/.my.cnf` es el patrón estándar para hacer idempotentes los playbooks de MariaDB.
- El port forwarding de Vagrant puede romper redirects HTTP→HTTPS porque `HTTP_HOST` incluye el puerto del host.

---


