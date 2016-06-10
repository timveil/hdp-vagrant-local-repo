# -*- mode: ruby -*-
# vi: set ft=ruby :

HOSTNAME = "repo.hdp.local"
IP = "192.168.66.10"

$hostsFile = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

$1  $2
EOF
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.box_check_update = true

  config.vbguest.auto_update = true

  config.vbguest.no_remote = true

  config.vbguest.no_install = false

  config.multihostsupdater.aliases = {IP => [HOSTNAME]}

  config.vm.provision "hosts", type: "shell", inline: $hostsFile, args: [IP, HOSTNAME]

  config.vm.provision "base", type: "shell", path: "provision-base.sh", args: [HOSTNAME]

  config.vm.provision "yum", type: "shell", path: "provision-yum.sh", args: [HOSTNAME]

  config.vm.hostname = HOSTNAME

  config.vm.network "private_network", ip: IP

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
    v.name = HOSTNAME
  end
end