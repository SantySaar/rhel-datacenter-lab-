# Role: nfs

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala nfs-utils
- Crea directorios de exportación
- Despliega /etc/exports con Jinja2
- Abre puertos nfs, rpc-bind y mountd en firewalld
- Habilita y arranca nfs-server

## Variables requeridas
```yaml
nfs_exports:
  - path: /srv/nfs/shared
    clients: "192.168.56.0/24(rw,sync,no_root_squash)"
```

## Verificación
```bash
showmount -e 192.168.56.30
exportfs -v
```

## Tags disponibles
`nfs`, `firewall`
