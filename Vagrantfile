# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box
$script = <<-SCRIPT

# Install kubernetes
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# sed -i '/ExecStart=/a Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# Get the IP address that VirtualBox has given this VM
IPADDR=`ifconfig eth1 | grep Mask | awk '{print $2}'| cut -f2 -d:`
echo This VM has IP address $IPADDR

# Set up Kubernetes
NODENAME=$(hostname -s)
kubeadm init --apiserver-cert-extra-sans=$IPADDR  --node-name $NODENAME

# Set up admin creds for the vagrant user
echo Copying credentials to /home/vagrant...
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
echo Completed credentials configuration.
exit 0
SCRIPT

Vagrant.configure("2") do |config|
  # define and name vm
	config.vm.define "kube_local"
	config.vm.box = "bento/ubuntu-16.04"

  config.vm.network "private_network" , type: "dhcp"

  config.vm.provider :virtualbox do |v|
    # enable VirtualBox DNS Proxy Mode to use host's resolver
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.name = "kube_local"
    v.memory = 4096
    v.cpus = 4
  end

  config.vm.provision "docker"

  config.vm.provision "shell", inline: $script
end