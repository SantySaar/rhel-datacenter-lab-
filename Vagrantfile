# =============================================================
#  Vagrantfile — rhel-datacenter-lab
#  3 VMs: control + web + services
#  Requisitos: VirtualBox + Vagrant en Windows
#
#  Uso:
#    vagrant up              # levanta las 3 VMs
#    vagrant ssh control     # accede al nodo Ansible
#    vagrant halt            # apaga las VMs
#    vagrant destroy -f      # elimina las VMs
# =============================================================

Vagrant.configure("2") do |config|

  config.vm.box = "bento/rockylinux-9"

  # ── VM 1: control (nodo Ansible) ───────────────────────────
  config.vm.define "control" do |ctl|
    ctl.vm.hostname = "control.lab.local"
    ctl.vm.network "private_network", ip: "192.168.56.10"
    ctl.vm.provider "virtualbox" do |vb|
      vb.name   = "lab-control"
      vb.memory = 1024
      vb.cpus   = 1
    end
  end

  # ── VM 2: web (Apache + HAProxy + Podman) ──────────────────
  config.vm.define "web" do |web|
    web.vm.hostname = "web.lab.local"
    web.vm.network "private_network", ip: "192.168.56.20"
    web.vm.network "forwarded_port", guest: 80,   host: 8080
    web.vm.network "forwarded_port", guest: 443,  host: 8443
    web.vm.network "forwarded_port", guest: 8090, host: 8090
    web.vm.network "forwarded_port", guest: 8404, host: 8404
    web.vm.provider "virtualbox" do |vb|
      vb.name   = "lab-web"
      vb.memory = 1024
      vb.cpus   = 1
    end
    web.vm.provision "shell", inline: <<-SHELL
      echo "192.168.56.10 control.lab.local control" >> /etc/hosts
      echo "192.168.56.20 web.lab.local web"         >> /etc/hosts
      echo "192.168.56.30 services.lab.local services" >> /etc/hosts
      dnf install -y python3 2>/dev/null
      echo "==> Web node listo"
    SHELL
  end

  # ── VM 3: services (DNS + NFS + Samba + MariaDB) ───────────
  config.vm.define "services" do |svc|
    svc.vm.hostname = "services.lab.local"
    svc.vm.network "private_network", ip: "192.168.56.30"
    svc.vm.provider "virtualbox" do |vb|
      vb.name   = "lab-services"
      vb.memory = 1024
      vb.cpus   = 1
    end
    svc.vm.provision "shell", inline: <<-SHELL
      echo "192.168.56.10 control.lab.local control" >> /etc/hosts
      echo "192.168.56.20 web.lab.local web"         >> /etc/hosts
      echo "192.168.56.30 services.lab.local services" >> /etc/hosts
      dnf install -y python3 2>/dev/null
      echo "==> Services node listo"
    SHELL
  end

end
