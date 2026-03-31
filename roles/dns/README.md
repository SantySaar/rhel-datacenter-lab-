# Role: dns

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala BIND9 (named + bind-utils)
- Despliega named.conf con zona autoritativa para lab.local
- Crea zona forward con registros A y CNAME para todas las VMs
- Abre puerto DNS en firewalld
- Habilita y arranca named

## Variables requeridas
```yaml
lab_domain: lab.local
lab_network: "192.168.56.0/24"
```

## Verificación
```bash
dig @192.168.56.30 web.lab.local
named-checkconf /etc/named.conf
named-checkzone lab.local /var/named/lab.local.zone
```

## Tags disponibles
`dns`, `firewall`
