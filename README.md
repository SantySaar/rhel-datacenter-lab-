# rhel-datacenter-lab 🏗️

> Datacenter empresarial completo automatizado con Ansible sobre Rocky Linux 9 (compatible RHEL 9).
> Demuestra la formación oficial Red Hat: **RH124 · RH134 · RH358 · RH294/AU374/AU467 · EX188**

![Rocky Linux](https://img.shields.io/badge/Rocky_Linux-9-10B981?logo=rockylinux&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white)
![Podman](https://img.shields.io/badge/Podman-Rootless-892CA0?logo=podman&logoColor=white)
![SELinux](https://img.shields.io/badge/SELinux-Enforcing-red)
![License](https://img.shields.io/badge/License-MIT-blue)

---

## 📸 Screenshots

<!-- Añade aquí las capturas cuando las tengas -->
<!-- ![Portfolio web](docs/img/portfolio.png) -->
<!-- ![HAProxy stats](docs/img/haproxy-stats.png) -->
<!-- ![Ansible playbook](docs/img/ansible-run.png) -->

---

## 🎯 Qué demuestra este proyecto

Despliegue automatizado de un **datacenter de 3 nodos Rocky Linux 9** con:

- **12 roles Ansible** modulares y reutilizables.
- **6 playbooks** para despliegues parciales y totales.
- **7 servicios de red** en producción (Apache/TLS, HAProxy, DNS, NFS, Samba, MariaDB, Podman)
- **Hardening completo**: SELinux enforcing, firewalld, SSH hardening, auditd, pwquality
- **Secretos cifrados** con Ansible Vault.
- **Todo reproducible** con un solo comando: `ansible-playbook site.yml`

---
## 🛠️ Stack técnico

**Sistemas:** Rocky Linux 9, RHEL 9, systemd, LVM, XFS, Bash

**Automatización:** Ansible (Playbooks, Roles, Vault, Jinja2, Handlers, Tags, Collections, AAP 2.x)

**Servicios:** Apache, mod_ssl, HAProxy, BIND9, NFS v4, Samba, MariaDB

**Seguridad:** SELinux, firewalld, SSH hardening, auditd, Ansible Vault, pwquality

**Contenedores:** Podman rootless, systemd units, volúmenes persistentes

**Herramientas:** Git, Vagrant, VirtualBox, ansible-lint

---

## 🏛️ Arquitectura

```
  Windows Host (VirtualBox)
  ┌─────────────────────────────────────────────┐
  │                                             │
  │  ┌──────────────┐                           │
  │  │  control     │  ← nodo Ansible           │
  │  │  .56.10      │    ansible-core + git     │
  │  └──────┬───────┘                           │
  │         │ SSH (Ansible managed)             │
  │    ┌────┴──────────────────┐                │
  │    │                       │                │
  │  ┌─▼────────────┐  ┌───────▼──────────┐     │
  │  │  web         │  │  services        │     │ 
  │  │  .56.20      │  │  .56.30          │     │
  │  │  Apache/TLS  │  │  DNS + NFS       │     │
  │  │  HAProxy     │  │  Samba + MariaDB │     │
  │  │  Podman      │  │                  │     │
  │  └──────────────┘  └──────────────────┘     │
  │                                             │
  │  Red host-only: 192.168.56.0/24             │
  └─────────────────────────────────────────────┘
```

---

## 🎓 Formación Red Hat demostrada

| Curso | Contenido | Roles / Ficheros |
|---|---|---|
| **RH124** — System Administration I | Shell, usuarios, SSH, DNF, systemd, permisos, NetworkManager | `common`, `users` |
| **RH134** — System Administration II | LVM, SELinux, firewalld, Grub2, scripting, tuning, troubleshooting | `hardening`, `storage` |
| **RH358** — Services Management | DNS/BIND9, Apache, MariaDB, NFS, Samba, iSCSI, HAProxy | `webserver`, `haproxy`, `dns`, `nfs`, `samba`, `mariadb` |
| **RH294 / AU374 / AU467** — Ansible Automation | Playbooks, roles, Vault, Collections, Jinja2, AAP 2.x | Toda la estructura Ansible del repo |
| **EX188** — Podman | Rootless containers, pods, volúmenes, systemd units | `podman` |

---

## 🚀 Servicios desplegados

| Servicio | VM | Role Ansible | Curso |
|---|---|---|---|
| Apache + TLS | web (192.168.56.20) | `webserver` | RH358 |
| HAProxy load balancer | web (192.168.56.20) | `haproxy` | RH358 |
| Podman + systemd | web (192.168.56.20) | `podman` | EX188 |
| DNS autoritativo BIND9 | services (192.168.56.30) | `dns` | RH358 |
| NFS server v4 | services (192.168.56.30) | `nfs` | RH358 |
| Samba SMB/CIFS | services (192.168.56.30) | `samba` | RH358 |
| MariaDB | services (192.168.56.30) | `mariadb` | RH358 |
| Hardening + SELinux | all_managed | `hardening` | RH134 |
| LVM + XFS + backup | all_managed | `storage` | RH134 |
| Usuarios + SSH keys | all_managed | `users` | RH124 |

---

## 📁 Estructura del proyecto

```
rhel-datacenter-lab/
├── README.md                        ← este fichero
├── ansible.cfg                      ← configuración Ansible
├── requirements.yml                 ← collections (RH294)
├── .gitignore                       ← excluye vault sin cifrar
│
├── docs/
│   ├── course-mapping.md            ← qué curso cubre cada fichero
│   ├── troubleshooting.md           ← solución de problemas comunes
│   └── lab-journal.md               ← diario de errores y aprendizajes
│
├── inventory/
│   ├── hosts.yml                    ← grupos: webservers, services
│   └── group_vars/
│       ├── all.yml                  ← variables globales
│       └── vault.yml                ← secretos cifrados (Vault · RH294)
│
├── playbooks/
│   ├── site.yml                     ← entry point: todo el stack
│   ├── hardening.yml                ← solo hardening (RH134)
│   ├── storage.yml                  ← solo LVM/XFS/backup (RH134)
│   ├── users.yml                    ← solo usuarios (RH124)
│   └── podman.yml                   ← solo contenedores (EX188)
│
├── roles/
│   ├── common/        ← RH124: paquetes, hostname, chrony
│   ├── hardening/     ← RH134: SELinux, firewalld, SSH, auditd
│   ├── users/         ← RH124: usuarios, grupos, sudoers, SSH keys
│   ├── storage/       ← RH134: LVM, XFS, fstab, rsync, cron
│   ├── webserver/     ← RH358: Apache, mod_ssl, TLS, portfolio web
│   ├── haproxy/       ← RH358: load balancer + stats
│   ├── dns/           ← RH358: BIND9 + zona lab.local
│   ├── nfs/           ← RH358: NFS server v4 + exports
│   ├── samba/         ← RH358: Samba SMB/CIFS + shares
│   ├── mariadb/       ← RH358: MariaDB + databases + users
│   └── podman/        ← EX188: Podman rootless + systemd
│
└── vagrant/
    └── Vagrantfile    ← define las 3 VMs
```

---

## ⚡ Inicio rápido

### Requisitos previos
1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) + Extension Pack
2. [Vagrant](https://developer.hashicorp.com/vagrant/install)
3. [Git for Windows](https://git-scm.com/download/win) (usa Git Bash)

### Paso a paso

```bash
# 1. Clonar el repositorio
git clone https://github.com/SantySaar/rhel-datacenter-lab-.git
cd rhel-datacenter-lab

# 2. Levantar las 3 VMs (primera vez descarga ~1.5GB)
cd vagrant
vagrant up

# 3. Verificar que las VMs están corriendo
vagrant status

# 4. Acceder al nodo control
vagrant ssh control

# 5. Dentro del control, clonar el proyecto
git clone https://github.com/SantySaar/rhel-datacenter-lab-.git
cd rhel-datacenter-lab

# 6. Instalar collections
ansible-galaxy collection install -r requirements.yml

# 7. Verificar conectividad
ansible all_managed -m ping

# 8. Desplegar todo el stack
ansible-playbook playbooks/site.yml
```

### Verificar el resultado
Desde tu navegador en Windows:
- **Portfolio web** → http://localhost:8080
- **HAProxy stats** → http://localhost:8404/stats
- **Podman container** → http://localhost:8090

---

## 🔧 Comandos útiles

```bash
# Ejecutar solo un rol
ansible-playbook playbooks/site.yml --tags webserver
ansible-playbook playbooks/site.yml --tags dns
ansible-playbook playbooks/site.yml --tags podman

# Ejecutar en un host concreto
ansible-playbook playbooks/site.yml --limit web.lab.local

# Dry-run (ver cambios sin aplicar)
ansible-playbook playbooks/site.yml --check --diff

# Listar todas las tareas
ansible-playbook playbooks/site.yml --list-tasks

# Ad-hoc: comprobar espacio en disco
ansible all_managed -m command -a "df -h"

# Ad-hoc: reiniciar un servicio
ansible web.lab.local -m service -a "name=httpd state=restarted" --become

# Gestión del Vault
ansible-vault encrypt inventory/group_vars/vault.yml
ansible-vault edit inventory/group_vars/vault.yml
ansible-playbook playbooks/site.yml --ask-vault-pass
```

---

## 📚 Documentación adicional

- [`docs/course-mapping.md`](docs/course-mapping.md) — Mapeo detallado curso → fichero
- [`docs/troubleshooting.md`](docs/troubleshooting.md) — Problemas comunes y soluciones
- [`docs/lab-journal.md`](docs/lab-journal.md) — Diario de errores reales resueltos

---

## 👤 Autor

**Santiago Alameda Recuero** — Linux SysAdmin Junior · Madrid, España
[LinkedIn](https://www.linkedin.com/in/santiagoalamedarecuero322/) · [GitHub](https://github.com/SantySaar)

---

*Portfolio técnico — Linux & Ansible · Madrid, España*
