# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.box = "timveil/centos7-hdp-base"
    config.vm.box_check_update = true

    config.vm.hostname = 'repo.hdp.local'
    config.vm.network "private_network", ip: '192.168.7.201'
    config.vm.provider "virtualbox" do |v|
      v.name = 'repo.hdp.local'
      v.memory = 1024
      v.cpus = 1
    end

    config.vbguest.auto_update = true
    config.vbguest.no_remote = true
    config.vbguest.no_install = false

    config.vm.provision "Create Local Repo", type:  "shell", inline: $createRepo

end

$createRepo = <<SCRIPT

OS=centos7
HDP_VERSION=2.4.2.0
AMBARI_VERSION=2.2.2.0
HDP_UTILS_VERSION=1.1.0.20
REPO_BASE=repo.hdp.local

AMBARI_REPO_URL=http://public-repo-1.hortonworks.com/ambari/$OS/2.x/updates/$AMBARI_VERSION/ambari.repo
HDP_REPO_URL=http://public-repo-1.hortonworks.com/HDP/$OS/2.x/updates/$HDP_VERSION/hdp.repo

yum update -y -q

wget -nv $AMBARI_REPO_URL -O /etc/yum.repos.d/ambari.repo
wget -nv $HDP_REPO_URL -O /etc/yum.repos.d/hdp.repo

yum install yum-utils createrepo httpd -y -q

systemctl start httpd
systemctl enable httpd

cd /var/www/html
mkdir -p ambari/$OS
cd ambari/$OS
reposync -r Updates-ambari-*

cd /var/www/html
mkdir -p hdp/$OS
cd hdp/$OS
reposync -r HDP-*

createrepo /var/www/html/hdp/$OS/HDP-$HDP_VERSION
createrepo /var/www/html/hdp/$OS/HDP-UTILS-$HDP_UTILS_VERSION
createrepo /var/www/html/ambari/$OS/Updates-ambari-$AMBARI_VERSION

cd /var/www/html
mkdir -p repos/$OS/hdp/$HDP_VERSION
mkdir -p repos/$OS/ambari/$AMBARI_VERSION

cat > /var/www/html/repos/$OS/hdp/$HDP_VERSION/hdp.repo <<EOL
[HDP-${HDP_VERSION}]
name=HDP Version - HDP-${HDP_VERSION}
baseurl=http://${REPO_BASE}/hdp/$OS/HDP-${HDP_VERSION}
enabled=1
priority=1
gpgcheck=0

[HDP-UTILS-${HDP_UTILS_VERSION}]
name=Hortonworks Data Platform Utils Version - HDP-UTILS-${HDP_UTILS_VERSION}
baseurl=http://${REPO_BASE}/hdp/$OS/HDP-UTILS-${HDP_UTILS_VERSION}
enabled=1
priority=1
gpgcheck=0
EOL

cat > /var/www/html/repos/$OS/ambari/$AMBARI_VERSION/ambari.repo <<EOL
[Updates-ambari-${AMBARI_VERSION}]
name=ambari-${AMBARI_VERSION} - Updates
baseurl=http://${REPO_BASE}/ambari/$OS/Updates-ambari-${AMBARI_VERSION}
enabled=1
priority=1
gpgcheck=0
EOL

echo "Ambari repo url = http://$REPO_BASE/repos/$OS/ambari/$AMBARI_VERSION/ambari.repo"
echo "HDP repo url = http://$REPO_BASE/repos/$OS/hdp/$HDP_VERSION/hdp.repo"

SCRIPT