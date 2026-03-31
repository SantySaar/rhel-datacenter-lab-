# Role: common

**Curso:** RH124 — Red Hat System Administration I

## Qué hace
Configuración base aplicada a todos los servidores del lab:
- Hostname correcto
- /etc/hosts con todos los nodos
- Paquetes esenciales (vim, git, bind-utils, chrony...)
- Sincronización de tiempo con chronyd
- Timezone Europe/Madrid

## Variables requeridas
Ninguna. Usa valores del inventario.

## Uso
```yaml
- hosts: all_managed
  roles:
    - common
```
