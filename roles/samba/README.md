# Role: samba

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala samba, samba-client y samba-common
- Crea directorios de shares con SELinux context samba_share_t
- Despliega smb.conf con Jinja2 (shares configurables por variable)
- Valida config con testparm antes de aplicar
- Abre Samba en firewalld
- Habilita y arranca smb y nmb

## Variables requeridas
```yaml
samba_workgroup: LABGROUP
samba_shares:
  - name: public
    path: /srv/samba/public
    comment: "Shared public folder"
    public: "yes"
    writable: "yes"
```

## Verificación
```bash
testparm -s
smbclient -L //192.168.56.30 -N
```

## Tags disponibles
`samba`, `firewall`
