# Mapa Formación → Código del Proyecto

Cada fichero del proyecto está vinculado al curso Red Hat que lo respalda.

---

## RH124 — Red Hat System Administration I

| Concepto | Fichero |
|---|---|
| Gestión de paquetes con DNF | `roles/common/tasks/main.yml` → `ansible.builtin.dnf` |
| Hostname y /etc/hosts | `roles/common/tasks/main.yml` → `hostname`, `lineinfile` |
| Usuarios y grupos | `roles/users/tasks/main.yml` → `ansible.builtin.user` |
| SSH y authorized_keys | `roles/users/tasks/main.yml` → `ansible.posix.authorized_key` |
| systemd (start/enable) | Todos los roles → `ansible.builtin.service` |
| Permisos de ficheros | `roles/webserver`, `roles/nfs`, `roles/samba` → `ansible.builtin.file` |
| Timezone | `roles/common/tasks/main.yml` → `community.general.timezone` |

---

## RH134 — Red Hat System Administration II

| Concepto | Fichero |
|---|---|
| SELinux enforcing + contextos | `roles/hardening/tasks/main.yml` → `ansible.posix.selinux` |
| firewalld (servicios, rich rules) | `roles/hardening`, `roles/webserver`, `roles/dns`, `roles/nfs`... |
| LVM (VG + LV) | `roles/storage/tasks/main.yml` → `community.general.lvg/lvol` |
| XFS filesystem | `roles/storage/tasks/main.yml` → `community.general.filesystem` |
| /etc/fstab (montaje persistente) | `roles/storage/tasks/main.yml` → `ansible.posix.mount` |
| Cron / tareas programadas | `roles/storage/tasks/main.yml` → `ansible.builtin.cron` |
| Shell scripting | `roles/storage/templates/backup.sh.j2` |
| SSH hardening | `roles/hardening/tasks/main.yml` → `lineinfile sshd_config` |
| auditd | `roles/hardening/tasks/main.yml` |
| Política de contraseñas | `roles/hardening/tasks/main.yml` → `pwquality.conf` |

---

## RH358 — Red Hat Services Management and Automation

| Concepto | Fichero |
|---|---|
| Apache (httpd) + mod_ssl | `roles/webserver/tasks/main.yml` |
| Virtual hosts | `roles/webserver/templates/vhost.conf.j2` |
| TLS/SSL self-signed | `roles/webserver/tasks/main.yml` → `openssl req` |
| DNS autoritativo BIND9 | `roles/dns/tasks/main.yml` + templates |
| Zonas DNS | `roles/dns/templates/zone.lab.local.j2` |
| NFS server v4 | `roles/nfs/tasks/main.yml` + `exports.j2` |
| Samba SMB/CIFS | `roles/samba/tasks/main.yml` + `smb.conf.j2` |
| MariaDB databases + users | `roles/mariadb/tasks/main.yml` |
| HAProxy load balancer | `roles/haproxy/tasks/main.yml` + `haproxy.cfg.j2` |

---

## RH294 / AU374 / AU467 — Linux Automation with Ansible

| Concepto | Fichero |
|---|---|
| Inventario YAML con grupos | `inventory/hosts.yml` |
| Variables de grupo | `inventory/group_vars/all.yml` |
| Ansible Vault (secretos) | `inventory/group_vars/vault.yml` |
| Playbooks multi-play | `playbooks/site.yml` |
| Roles con estructura estándar | `roles/*/` (10 roles) |
| Templates Jinja2 | `roles/*/templates/*.j2` |
| Handlers | `roles/*/handlers/main.yml` |
| Tags para ejecución selectiva | Todos los roles |
| Collections externas | `requirements.yml` |
| ansible.cfg | `ansible.cfg` |

---

## EX188 — Red Hat Certified Specialist in Containers (Podman)

| Concepto | Fichero |
|---|---|
| Instalación Podman rootless | `roles/podman/tasks/main.yml` |
| Pull de imagen | `roles/podman/tasks/main.yml` → `containers.podman.podman_image` |
| Crear contenedor con volumen y puerto | `roles/podman/tasks/main.yml` → `containers.podman.podman_container` |
| Integración con systemd | `roles/podman/tasks/main.yml` → `podman generate systemd` |
| Lingering para rootless persistente | `roles/podman/tasks/main.yml` → `loginctl enable-linger` |
| Collection containers.podman | `requirements.yml` |
