# virtualbox-coa-aio-train

## virtualbox setup
vboxmanage modifyvm "coa-aio-train" --nested-hw-virt on

## virtual machine
## interface NIC0 configured as br-ex
cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
DEVICE=enp0s3
ONBOOT=yes
IPADDR=10.0.2.50
PREFIX=24
GATEWAY=10.0.2.1

## interface NIC1 configure as API
DEVICE=enp0s8
ONBOOT=yes
IPADDR=192.168.56.50
PREFIX=24

hostnamectl set-hostname coa-aio-train
systemctl disable --now firewalld NetworkManager
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
echo “nameserver 8.8.8.8” > /etc/resolv.conf
yum -y install centos-release-openstack-train
yum install -y openstack-packstack

packstack --allinone \
--default-password=myrootpassword \
--os-controller-host=192.168.56.50 \
--os-cinder-install=y \
--os-ceilometer-install=n \
--os-trove-install=n \
--os-ironic-install=n \
--os-swift-install=y \
--os-aodh-install=n \
--os-heat-install=n \
--os-neutron-ml2-mechanism-drivers=openvswitch \
--os-neutron-ml2-tenant-network-types=vxlan \
--os-neutron-ml2-type-drivers=vxlan,flat,vlan \
--os-neutron-ml2-vlan-ranges=physnet1:298:299 \
--os-neutron-ovs-bridge-mappings=physnet1:br-ex \
--os-neutron-l2-agent=openvswitch \
--os-neutron-ovs-bridge-interfaces=br-ex:enp0s3 \
--provision-demo=n 

source /root/keystonerc_admin
openstack image create "cirros" --file /root/cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
openstack flavor create --id 1 --ram 1024 --disk 1  --vcpu 1 tiny
openstack flavor create --id 2 --ram 4096 --disk 10 --vcpu 2 small
openstack flavor create --id 4 --ram 8096 --disk 50 --vcpu 2 medium



openstack network create --provider-network-type flat --provider-physical-network physnet1 --external --share external-flat-network
openstack router set --external-gateway external-flat-network --enable-snat cusA-router1

openstack server create --flavor 1 --image cirros --nic net-id=96448280-519e-4173-a198-ee0b18d66f02 inst1
openstack server create --flavor 1 --image cirros --nic net-id=$net-id-pro-vlan111 inst1