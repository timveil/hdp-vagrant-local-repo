# Local YUM Repository for Hortonworks Data Platform (HDP) using Vagrant

## Overview
This project provides a quick and easy way to build a small, local YUM repo for the Hortonworks Data Platform (HDP) using Vagrant.  This fairly closely mirrors the official Hortonworks Documentation found [here](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.0.1/bk_ambari-installation/content/getting_started_setting_up_a_local_repository.html).  Building and referring to a local YUM repository can be very useful if you have limited bandwidth or an unreliable internet connection.  It can also significantly reduce your data plan consumption if you are frequently building Hadoop clusters like I am.  This can be easily paired with my [HDP Vagrant Generator](https://github.com/timveil/hdp-vagrant-generator) project by updating that projects `application.properties` in that project.  For example, updating these values in `application.properties` would allow you to generate your Vagrant image using your freshly built local YUM Repository.

```dosini
# Custom Base URL for HDP Repo (ex. "http://repo.hdp.local/hdp/centos7/HDP-2.5.0.0")
hdp.repo.base=

# Custom Base URL for HDP Utils Repo (ex. "http://repo.hdp.local/hdp/centos7/HDP-UTILS-1.1.0.21")
hdp.repo.utils.base=

```

By default, the Vagrant image built by this project will have the following configuration, but can be easily changed by updating the `Vagrantfile`

Property | Value
------------ | -------------
Hostaname | repo.hdp.local
IP Address | 192.168.7.201
RAM | 1024 MB
CPUs | 1

## How to Run
Assuming, Vagrant is properly installed you can create the the local repo image by running `vagrant up`.  The process usually takes about 20 minutes based on hardware and network connectivity.  Keep in mind that you will need to update your Hosts file on your host environment to properly recognize the hostname.

## How to Customize
The `Vagrantfile` uses the following properties to control the content of the local repo.  You may adjust as necessary.

```rb
OS=centos7
HDP_VERSION=2.5.0.0
AMBARI_VERSION=2.4.0.1
HDP_UTILS_VERSION=1.1.0.21
REPO_BASE=repo.hdp.local
```

Below is the full `Vagrantfile` for reference:

```rb
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

    config.vm.provision "Update /etc/hosts File", type: "shell", inline: $hostsFile
    config.vm.provision "Create Local Repo", type: "shell", inline: $createRepo

end

$hostsFile = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.7.201   repo.hdp.local
EOF
SCRIPT

$createRepo = <<SCRIPT

OS=centos7
HDP_VERSION=2.5.0.0
AMBARI_VERSION=2.4.0.1
HDP_UTILS_VERSION=1.1.0.21
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
```