# Lab Journal — rhel-datacenter-lab

> Diario de errores, soluciones y aprendizajes durante la construcción del lab.
> Este fichero es lo que demuestra a un entrevistador que realmente ejecutaste el proyecto.

---

## Formato de entrada


## Día X — DD/MM/YYYY
**Tiempo dedicado:** X horas
**Avances:**
- ...
**Errores encontrados:**
- Error: ...
  Causa: ...
  Solución: ...
**Aprendizajes clave:**
- ...
**Pendiente para mañana:**
- ...
---

## Día 1 — 30/03/2026
**Tiempo dedicado:** 6 horas

**Avances:**
- Instalado VirtualBox 7.2.6, Vagrant 2.4.9 y Git 2.53.0 en Windows
- 3 VMs Rocky Linux 9 (bento/rockylinux-9) corriendo: control, web, services
- Ansible Core 2.14.18 instalado en nodo control
- ansible all_managed -m ping — SUCCESS en ambos nodos
- ansible-playbook playbooks/site.yml ejecutado completamente
- Apache con TLS accesible desde Windows en https://localhost:8443

**Errores encontrados:**
- Error: vagrant box rockylinux/9 devuelve 404
  Causa: La versión 6.0.0 del box fue eliminada de Vagrant Cloud
  Solución: Cambiar a bento/rockylinux-9 en el Vagrantfile

Error: vagrant ssh control no conectaba — connection reset infinito
Causa: El bloque synced_folder estaba fuera del bloque do |ctl| en el Vagrantfile, corrompiendo la configuración SSH
Solución: Reescribir el bloque control correctamente con la indentación adecuada

Error: ansible all_managed -m ping — provided hosts list is empty
Causa: Ansible ignora ansible.cfg en directorios world-writable (/vagrant)
Solución: Copiar el proyecto a ~/rhel-datacenter-lab fuera de /vagrant

Error: ansible all_managed -m ping — Permission denied (publickey)
Causa: El inventario hosts.yml tenía hardcodeada la ruta ~/.vagrant.d/insecure_private_key que no existe con el box bento
Solución: Cambiar ansible_ssh_private_key_file a ~/.ssh/id_ed25519 en inventory/hosts.yml

Error: community.general.yaml callback deprecated
Causa: El callback stdout_callback = yaml fue eliminado en community.general 12.0.0
Solución: Cambiar a stdout_callback = default en ansible.cfg

Error: Rol common not found
Causa: roles_path no estaba configurado en ansible.cfg
Solución: Añadir roles_path = ~/rhel-datacenter-lab/roles en ansible.cfg

Error: Grupo developers does not exist al crear usuarios
Causa: El rol users creaba usuarios antes de crear los grupos
Solución: Añadir tarea Create lab groups antes de Create lab users en roles/users/tasks/main.yml

Error: Apache no arrancaba — localhost.crt not found
Causa: El /etc/httpd/conf.d/ssl.conf por defecto apuntaba a localhost.crt en vez de lab.crt
Solución: Añadir tarea en rol webserver para eliminar ssl.conf por defecto antes de arrancar httpd

Error: Apache solo escuchaba en puerto 80, no en 443
Causa: Al borrar ssl.conf también se eliminó la directiva Listen 443 https necesaria para SSL
Solución: Crear un ssl.conf mínimo con Listen 443 https y directivas SSL básicas

Error: Certificado generado como CA certificate (BasicConstraints: CA == TRUE)
Causa: El comando openssl req del rol no incluía -addext "basicConstraints=CA:FALSE"
Solución: Regenerar el certificado manualmente con el flag correcto

**Aprendizajes clave:**
- Ansible ignora ansible.cfg en directorios world-writable — nunca trabajar desde /vagrant
- El box bento/rockylinux-9 es más estable que rockylinux/9 en VirtualBox con Windows
- El orden de las tareas en los roles importa — grupos antes que usuarios
- Borrar ssl.conf de Apache elimina también Listen 443 — hay que recrearlo mínimo

**Pendiente para mañana:**
- Verificar HAProxy en localhost:8404
- Verificar DNS con dig desde control
- Verificar NFS con showmount
- Verificar MariaDB
- Arreglar rol storage (LVM requiere disco /dev/sdb — añadirlo en Vagrantfile)
- Subir proyecto a GitHub con commits reales


## Día 2 — 31/03/2026
**Tiempo dedicado:** 6 horas

**Avances:**
- Stack completo verificado y funcionando: Apache+TLS, HAProxy, DNS, NFS, Samba, MariaDB, Podman
- ansible-playbook site.yml con 0 fallos reales en ambos nodos
- Proyecto subido a GitHub: https://github.com/SantySaar/rhel-datacenter-lab-
- Ansible Vault configurado para secretos de MariaDB
- SSH key distribution automatizada en rol common
- Disco /dev/sdb añadido a VMs web y services para LVM

**Errores encontrados:**
- Error: HAProxy no arrancaba — Permission denied en puerto 8404
  Causa: SELinux no tenia el puerto 8404 como http_port_t
  Solucion: Añadir tarea community.general.seport en rol haproxy

- Error: BIND9 no escuchaba en 192.168.56.30
  Causa: ansible_default_ipv4.address devuelve IP NAT no la host-only
  Solucion: Cambiar a ansible_host en template named.conf.j2

- Error: vault_mariadb_root_password undefined
  Causa: vault.yml estaba en group_vars/ en vez de group_vars/all/
  Solucion: Mover vault.yml a inventory/group_vars/all/vault.yml

- Error: Permission denied al crear vault.yml
  Causa: Ficheros copiados con sudo tenian owner root
  Solucion: sudo chown -R vagrant:vagrant ~/rhel-datacenter-lab/

- Error: podman-compose not available
  Causa: El paquete no existe en repos de Rocky Linux 9
  Solucion: Eliminar podman-compose de la lista de paquetes del rol podman

- Error: Generate systemd unit fallaba en Podman
  Causa: Directorio ~/.config/systemd/user/ se creaba despues de usarlo
  Solucion: Reordenar tareas — crear directorio antes de generar unit

**Aprendizajes clave:**
- ansible_default_ipv4.address no es fiable con multiples interfaces — usar ansible_host
- Los ficheros de vars del vault deben estar en group_vars/all/ para cargarse automaticamente
- SELinux bloquea puertos no estandar aunque firewalld los tenga abiertos
- El orden de tareas en roles es critico — dependencias implicitas dan errores dificiles

**Pendiente para mañana:**
- Añadir capturas de pantalla al README
- Hacer commits incrementales por cada fix
- Publicar GitHub Pages.
