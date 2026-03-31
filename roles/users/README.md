# Role: users

**Curso:** RH124 — Red Hat System Administration I

## Qué hace
Gestión centralizada de usuarios en todos los servidores:
- Crea usuarios con grupos y shell definidos en variables
- Configura sudoers para el grupo wheel
- Despliega authorized_keys para acceso SSH sin contraseña

## Variables requeridas (group_vars/all.yml)
```yaml
lab_users:
  - name: sysadmin
    groups: wheel
    sudo: true
    shell: /bin/bash
```

## Uso
```bash
ansible-playbook playbooks/users.yml
ansible-playbook playbooks/site.yml --tags users
```
