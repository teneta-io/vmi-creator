# Requirements
```
# For Ubuntu host system:
sudo apt update -y
sudo apt install -y \
virtinst \
qemu \
qemu-kvm \
libvirt-daemon \
libvirt-clients \
bridge-utils \
virt-manager
```

# Create a cloud-init configuration so we can set the password and the hostname etc.
```
sudo echo "#cloud-config
system_info:
  default_user:
    name: ubuntu
    home: /home/ubuntu

password: $PASSWORD
chpasswd: { expire: False }
hostname: $VM_NAME

clouduser-ssh-key: ~/.ssh/id_rsa.pub


#Configure sshd to allow users logging in using password rather than just keys

ssh_pwauth: True
" | sudo tee cloud-init.cfg
```

# Create a cloud-init configuration so we can set the password, hostname, and some other things.
```
sudo echo "#cloud-config
hostname: $VM_NAME
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    lock_passwd: false
#    ssh-authorized-keys:
#      - <sshPUBKEY>

#Only cert auth via ssh (console access can still login)

ssh_pwauth: false
disable_root: false
password: $PASSWORD
chpasswd: { expire: False }

package_update: true
#packages:
#  - qemu-guest-agent
" | sudo tee cloud-init.cfg
```
# Optionally create netplan network configuration
```
sudo echo "#network_config
network:
    ethernets:
        ens2:
            dhcp4: true
    version: 2
" | sudo tee network_config.cfg
```
# Create the ISO file from the cloud config file we just created:
```
sudo cloud-localds cloud-init.iso cloud-init.cfg
```
## Or with network
```
sudo cloud-localds -v --network-config=network_config.cfg cloud-init.iso cloud-init.cfg
```

# Create VM with the cloud image and the cloud-init metadata
```
sudo virt-install \
  --name $VM_NAME \
  --memory $VM_MEM \
  --vcpus $VM_CPU \
  --disk jammy-server-cloudimg-amd64-disk-kvm.img,device=disk \
  --disk cloud-init.iso,device=cdrom \
  --os-variant ubuntu22.04 \
  --virt-type kvm \
  --graphics none \
  --network network:default
```
## Example value for variables:
```
#hostname of machine:
VM_NAME=Ubuntu-test-VM
echo VM_NAME=$VM_NAME

#user password
PASSWORD=ubuntu
echo PASSWORD=$PASSWORD

#Memory to allocate for guest instance in megabytes.
#If the hypervisor does not have enough free memory, it is usual for it to automatically take memory away from the host operating system to satisfy this allocation.
VM_MEM=8192
echo VM_MEM=$VM_MEM

#Number of virtual cpus to configure for the guest.
#If 'maxvcpus' is specified, the guest will be able to hotplug up to MAX vcpus while the guest is running, but will startup with VCPUS
VM_CPU=8
echo VM_CPU=$VM_CPU

```
