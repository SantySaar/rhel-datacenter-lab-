# Role: webserver

**Curso:** RH358 — Red Hat Services Management and Automation

## Qué hace
- Instala Apache (httpd) con mod_ssl
- Genera certificado SSL self-signed con OpenSSL
- Despliega virtual host HTTP→HTTPS con redirect
- Sirve el portfolio web como template Jinja2 dinámico
- Abre puertos 80/443 en firewalld

## Variables requeridas
```yaml
web_server_name: "web.lab.local"
web_document_root: /var/www/html
```

## Tags disponibles
`webserver`, `ssl`, `firewall`

## Uso
```bash
ansible-playbook playbooks/site.yml --tags webserver
```
