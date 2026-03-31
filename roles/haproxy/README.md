# Role: haproxy

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala HAProxy
- Despliega configuración con Jinja2 y valida con `haproxy -c` antes de aplicar
- Configura frontend en puerto 8080 y backend apuntando a Apache
- Habilita stats dashboard en puerto 8404/stats
- Abre puertos 8080 y 8404 en firewalld
- Habilita y arranca HAProxy

## Verificación
```bash
# Stats dashboard:
curl http://192.168.56.20:8404/stats

# Test balanceo:
curl http://192.168.56.20:8080

# Estado del servicio:
systemctl status haproxy
haproxy -c -f /etc/haproxy/haproxy.cfg
```

## Tags disponibles
`haproxy`, `firewall`
