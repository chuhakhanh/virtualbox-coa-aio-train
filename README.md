# virtualbox-coa-aio-train

## virtualbox setup
vboxmanage modifyvm "coa-aio-train" --nested-hw-virt on

## virtual machine
## interface NIC0
cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
DEVICE=enp0s3
ONBOOT=yes
IPADDR=10.0.2.50
PREFIX=24
GATEWAY=10.0.2.1

## interface NIC1
IPADDR=192.168.56.50
PREFIX=24


hostnamectl set-hostname coa-aio-train
systemctl disable --now firewalld NetworkManager
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
echo “nameserver 8.8.8.8” > /etc/resolv.conf
yum -y install centos-release-openstack-train
yum install -y openstack-packstack
packstack --gen-answer-file /root/answers.cfg