# Role: storage

**Curso:** RH134 — Red Hat System Administration II

## Qué hace
Gestión completa de almacenamiento en RHEL:
- Crea Volume Group `vg_lab` sobre disco adicional
- Crea Logical Volumes con tamaño configurable
- Formatea con XFS y monta persistentemente en /etc/fstab
- Despliega script de backup con rsync
- Programa backup diario a las 02:00 con cron

## Variables requeridas (group_vars/all.yml)
```yaml
lvm_volumes:
  - name: lv_data
    size: 2g
    mount: /data
backup_source_dirs: "/etc /var/named /srv/nfs"
```

## Tags disponibles
`storage`, `lvm`, `xfs`, `backup`, `cron`

## Uso
```bash
ansible-playbook playbooks/storage.yml
ansible-playbook playbooks/site.yml --tags lvm
```
