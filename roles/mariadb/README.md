# Role: mariadb

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala mariadb-server y python3-PyMySQL (requerido por módulos Ansible)
- Configura password de root
- Crea databases y usuarios con privilegios
- Abre puerto MySQL solo para la red del lab (rich rule firewalld)
- Handler para reinicio si cambia configuración

## Variables requeridas
```yaml
mariadb_root_password: "{{ vault_mariadb_root_password }}"
mariadb_databases:
  - name: labdb
mariadb_users:
  - name: labuser
    password: "{{ vault_mariadb_app_password }}"
    priv: "labdb.*:ALL"
```

## Verificación
```bash
mysql -u root -p -e "SHOW DATABASES;"
mysql -u labuser -p labdb -e "SHOW TABLES;"
```

## Tags disponibles
`mariadb`, `firewall`
