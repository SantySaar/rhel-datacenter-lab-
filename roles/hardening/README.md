# Role: hardening

**Curso:** RH134 — Red Hat System Administration II

## Qué hace
Hardening de seguridad basado en CIS Benchmark para RHEL:
- SELinux en modo enforcing (política targeted)
- firewalld activo y habilitado
- SSH hardening: sin root login, sin password auth, banner, timeouts
- auditd para registro de actividad
- Política de contraseñas con pwquality (min 12 chars, complejidad)

## Variables requeridas
Ninguna.

## Tags disponibles
`hardening`, `selinux`, `firewall`, `ssh`, `audit`, `password`

## Uso
```bash
ansible-playbook playbooks/hardening.yml
ansible-playbook playbooks/site.yml --tags selinux
```
