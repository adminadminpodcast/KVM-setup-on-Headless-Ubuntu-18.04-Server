
# KVM setup on Headless Ubuntu 18.04 Server

This guide show you how to setup KVM on a headless ubuntu 18.04 Server and setup Kimchi, a HTML5 Client to manage the VMs.

## Prerequisites

 A bare metal machine with ubuntu 18.04 Server instaleld on it, Fully patched and SSH enabled. My Server is called KVM and has a static IP address of 192.168.127.200

## Install KVM
Run the following commands to install the base KVM packages via SSHing in the server.

``` 
sudo apt-get install qemu-kvm libvirt-bin virtinst cpu-checker 
```

Run the following Commands to check that your CPU support KVM

``` 
kvm-ok 
```

## Setup Network Bridge
To allow your VMs access to your network so that they appair that they have IP address of your  local lan network.

Edit you network config using netplan
```
sudo vi /etc/netplan/50-cloud-init.yaml
```

Below is my current static IP address setup configured by netplan

```
network:
    version: 2
    ethernets:
        enp0s25:
            dhcp4: no
            addresses: [192.168.127.200/24]
            gateway4: 192.168.127.10
            nameservers:
                addresses: [192.168.127.10]
```
Update the netplan config so that you enable a bridge on enp0s25 interface
```
network:
    version: 2
    ethernets:
        enp0s25:
            dhcp4: no
    bridges:
        br0:
            interfaces: [enp0s25]
            dhcp4: no
            addresses: [192.168.127.200/24]
            gateway4: 192.168.127.1
            nameservers:
                addresses: [192.168.127.10]
```
Apply the following command to apply the config
```
sudo netplan --debug  apply
```
You will proably get disconnected from the VM, SSH back in the KVM host to make sure the Network is working correctly

## Create your First Virtual Machine

We first need to download the lastest Ubuntu ISO to the KVM server.

```
sudo wget -P /var/lib/libvirt/images http://releases.ubuntu.com/18.04/ubuntu-18.04.2-live-server-amd64.iso /var/lib/libvirt/images
```

On your local Linux Desktop you can use Virtual Machine Manager to Create your first VM. Watch the youtube video (link at the top of the page) on what steps are need to create and install VMs.

Use the Follow comamd on your client to install the Virtual Machine Manager 

```
sudo apt-get install virt-manager
```

## Set Kimchi and Wok

We are now going to install Kimchi and Wok so that we can manage Virtual Machines via a HTML5 web browser.

SSH back on to the KVM server

Install dependencies required for Kimchi and Wok

``` 
sudo apt install python-paramiko python-pil novnc python-libvirt python-ethtool python-ipaddr python-guestfs libguestfs-tools spice-html5 spice-html5 python-magic keyutils libnfsidmap2 libtirpc1 nfs-common rpcbind python-configobj python-parted nginx -y
```

Download the wok and kimchi deb packages 
``` 
wget https://github.com/kimchi-project/kimchi/releases/download/2.5.0/wok-2.5.0-0.noarch.deb
```
```
wget https://github.com/kimchi-project/kimchi/releases/download/2.5.0/kimchi-2.5.0-0.noarch.deb
```

Install wok by running the following command
```
sudo dpkg -i wok-2.5.0-0.noarch.deb
```
Run the following command to install the missing dependencies for wok and allow apt to then install wok
```
sudo apt install -f -y
```
Install Kimchi by running the following command
```
sudo dpkg --ignore-depends=python-imaging -i kimchi-2.5.0-0.noarch.deb
```
Download the following patch to fix the iso images error when try to add templates in kimchi
```
sudo wget -q https://raw.githubusercontent.com/kimchi-project/kimchi/1ec059af4040c50b1a7b9a34253510a46ca09d3b/model/templates.py -O /usr/lib/python2.7/dist-packages/wok/plugins/kimchi/model/templates.py
```
Run the following command to allow kimchi to have access to ubuntu iso
```
sudo cp /var/lib/libvirt/images/ubuntu-18.04.2-live-server-amd64.iso /var/lib/kimchi/isos/ubuntu-18.04.2-live-server-amd64.iso
```
Reboot the server to apply changes
```
sudo shutdown -r now
```

Goto https://kvm:8001 to start managing VMs via HTML5 Web browsers