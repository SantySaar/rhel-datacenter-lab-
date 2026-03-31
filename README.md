# rhel-datacenter-lab 🏗️

> Portfolio técnico: datacenter completo automatizado con Ansible sobre Rocky Linux 9 (compatible RHEL 9).
> Demuestra la formación oficial Red Hat completa: **RH124 · RH134 · RH358 · RH294/AU374/AU467 · EX188**

---

## ¿Qué hace este proyecto?

Despliega automáticamente un entorno de datacenter con **3 VMs** y los siguientes servicios:

| Servicio | VM | Role Ansible | Curso |
|---|---|---|---|
| Apache + TLS | web (192.168.56.20) | `webserver` | RH358 |
| HAProxy load balancer | web (192.168.56.20) | `haproxy` | RH358 |
| Podman + systemd | web (192.168.56.20) | `podman` | EX188 |
| DNS autoritativo BIND9 | services (192.168.56.30) | `dns` | RH358 |
| NFS server v4 | services (192.168.56.30) | `nfs` | RH358 |
| Samba SMB/CIFS | services (192.168.56.30) | `samba` | RH358 |
| MariaDB | services (192.168.56.30) | `mariadb` | RH358 |
| CIS Hardening (todos) | all\_managed | `hardening` | RH134 |
| LVM + XFS + backup | all\_managed | `storage` | RH134 |
| Usuarios + SSH keys | all\_managed | `users` | RH124 |

El **portfolio web** se sirve desde Apache como template Jinja2 con datos reales del servidor — desplegado por Ansible.

---

## Arquitectura

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
  │  ┌─▼────────────┐  ┌───────▼──────────┐    │
  │  │  web         │  │  services        │    │
  │  │  .56.20      │  │  .56.30          │    │
  │  │  Apache/TLS  │  │  DNS + NFS       │    │
  │  │  HAProxy     │  │  Samba + MariaDB │    │
  │  │  Podman      │  │                  │    │
  │  └──────────────┘  └──────────────────┘    │
  │                                             │
  │  Red host-only: 192.168.56.0/24            │
  └─────────────────────────────────────────────┘
```

---

## Estructura del proyecto

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

## Inicio rápido en Windows

### Requisitos previos
1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) + Extension Pack
2. [Vagrant](https://developer.hashicorp.com/vagrant/install)
3. [Git for Windows](https://git-scm.com/download/win) (usa Git Bash)

### Paso a paso

```bash
# 1. Clonar el repositorio
git clone https://github.com/tuusuario/rhel-datacenter-lab.git
cd rhel-datacenter-lab

# 2. Levantar las 3 VMs (primera vez descarga ~1.5GB)
cd vagrant
vagrant up

# 3. Verificar que las VMs están corriendo
vagrant status

# 4. Acceder al nodo control
vagrant ssh control

# 5. Clonar el proyecto dentro del control
git clone https://github.com/tuusuario/rhel-datacenter-lab.git
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
- **Portfolio web**: http://localhost:8080
- **HAProxy stats**: http://localhost:8404/stats
- **Podman container**: http://localhost:8090

---

## Comandos útiles

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

# Gestión del vault
ansible-vault encrypt inventory/group_vars/vault.yml
ansible-vault edit inventory/group_vars/vault.yml
ansible-playbook playbooks/site.yml --ask-vault-pass
```

---

## Formación que respalda este proyecto

| Curso | Contenido | Roles |
|---|---|---|
| RH124 | Shell, usuarios, SSH, DNF, systemd | `common`, `users` |
| RH134 | SELinux, firewalld, LVM, scripting | `hardening`, `storage` |
| RH358 | DNS, Apache, NFS, Samba, MariaDB, HAProxy | `webserver`, `haproxy`, `dns`, `nfs`, `samba`, `mariadb` |
| RH294/AU374/AU467 | Ansible: playbooks, roles, Vault, Jinja2, Collections | Toda la estructura |
| EX188 | Podman rootless, pods, systemd integration | `podman` |

---

*Portfolio técnico — Linux & Ansible Engineer · Madrid, España*
