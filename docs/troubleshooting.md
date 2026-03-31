# Troubleshooting Guide

## VMs no arrancan

```bash
VBoxManage --version   # verificar VirtualBox instalado
vagrant status         # ver estado de las VMs
vagrant up --debug     # arrancar con debug detallado

# Si hay conflicto de red host-only:
# VirtualBox → File → Host Network Manager
# Crear red: 192.168.56.1 / 255.255.255.0
```

## Ansible no conecta

```bash
vagrant status
vagrant ssh web                          # probar SSH manualmente
ansible all_managed -m ping              # test conectividad Ansible
ansible all_managed -m ping -vvv         # debug detallado
vagrant reload web                       # reiniciar una VM
```

## SELinux bloquea un servicio

```bash
# Ver qué está bloqueando
sudo ausearch -c httpd --raw | audit2allow

# Crear módulo permisivo y aplicar
sudo ausearch -c httpd --raw | audit2allow -M httpd_custom
sudo semodule -i httpd_custom.pp

# Ver contexto de un fichero
ls -lZ /var/www/html/

# Restaurar contexto por defecto
sudo restorecon -Rv /var/www/html/

# Cambiar contexto manualmente
sudo chcon -t httpd_sys_content_t /var/www/html/index.html
```

## firewalld bloquea tráfico

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --get-services        # ver servicios disponibles
```

## DNS no resuelve

```bash
dig @192.168.56.30 web.lab.local
nslookup web.lab.local 192.168.56.30
sudo journalctl -u named -f
sudo named-checkconf /etc/named.conf
sudo named-checkzone lab.local /var/named/lab.local.zone
```

## Apache no arranca

```bash
sudo systemctl status httpd
sudo journalctl -u httpd -f
sudo apachectl configtest            # verificar sintaxis
sudo httpd -t                        # alternativa
# Comprobar SELinux y puertos con ausearch
```

## MariaDB — acceso denegado

```bash
# Verificar servicio
sudo systemctl status mariadb
sudo journalctl -u mariadb

# Probar conexión local
mysql -u root -p
mysql -u labuser -p labdb

# Reset root (emergencia)
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &
mysql -u root -e "FLUSH PRIVILEGES; ALTER USER 'root'@'localhost' IDENTIFIED BY 'nueva';"
```

## NFS — no monta en cliente

```bash
showmount -e 192.168.56.30
sudo mount -t nfs 192.168.56.30:/srv/nfs/shared /mnt/test
sudo journalctl -u nfs-server -f
exportfs -v
```

## Podman — contenedor no arranca

```bash
podman ps -a                          # ver todos los contenedores
podman logs portfolio-nginx           # ver logs del contenedor
systemctl --user status container-portfolio-nginx
podman inspect portfolio-nginx
# Comprobar SELinux en volúmenes (etiqueta :Z)
ls -lZ /srv/podman/html/
```

## Playbook falla a mitad

```bash
# Dry-run para ver qué cambiaría
ansible-playbook playbooks/site.yml --check --diff

# Ejecutar solo desde la tarea que falló
ansible-playbook playbooks/site.yml --start-at-task "nombre de la tarea"

# Ejecutar solo un tag
ansible-playbook playbooks/site.yml --tags dns

# Debug de variables
ansible web.lab.local -m debug -a "var=hostvars[inventory_hostname]"

# Modo verbose
ansible-playbook playbooks/site.yml -vvv
```
