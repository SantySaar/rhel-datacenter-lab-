# Role: podman

**Curso:** EX188 — Red Hat Certified Specialist in Containers

## Qué hace
- Instala Podman, podman-compose, slirp4netns y fuse-overlayfs
- Habilita lingering para contenedores rootless persistentes
- Descarga imagen Nginx Alpine
- Crea contenedor rootless con volumen y port mapping
- Genera y habilita unit de systemd para arranque automático
- Abre el puerto en firewalld

## Variables requeridas (group_vars/all.yml)
```yaml
podman_containers:
  - name: portfolio-nginx
    image: docker.io/library/nginx:alpine
    ports:
      - "8090:80"
    volumes:
      - "/srv/podman/html:/usr/share/nginx/html:ro,Z"
    systemd: true
```

## Verificación
```bash
# Dentro de la VM web:
podman ps
systemctl --user status container-portfolio-nginx
curl http://localhost:8090
```

## Tags disponibles
`podman`, `firewall`

## Uso
```bash
ansible-playbook playbooks/podman.yml
ansible-playbook playbooks/site.yml --tags podman
```
