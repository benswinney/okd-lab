# Install - lab.okd.example.com

< Prerequisite: [Install Centos 7.8](00_install_centos.md)

* * *

Typically this is a one time installation. Not to much automation here, just follow step by step and __copy&paste__.

## Clone `okd-lab`

Clone this project to your home desktop user directory where you start your journey.

```bash
[home@home]

cd ~/
git clone https://github.com/disposab1e/okd-lab.git

```

## Prepare guide (optional)

To follow the __copy&paste__ style of this guide it might be useful to set the IP address of your host to some commands.

```bash
[home@home]

# Replace 100.100.100.100 with your public IP address

sed -i 's/YO.UR.I.P/100.100.100.100/g' ~/okd-lab/docs/01_install_lab.md

```

Or just use your favorite tool and replace `YO.UR.I.P` with your public IP address.

## SSH and user configuration

### Generate new SSH keys

Accept all settings and use NO passphrase.

```bash
[home@home]

ssh-keygen -f ~/.ssh/okd_lab_id_rsa

```

### Authorize public key

```bash
[home@home]

ssh-copy-id -i ~/.ssh/okd_lab_id_rsa.pub lab@YO.UR.I.P

```

### Configure SSH config

```bash
[home@home]

vi ~/.ssh/config

# Add to your existing file:

Host YO.UR.I.P
  HostName YO.UR.I.P
  IdentityFile ~/.ssh/okd_lab_id_rsa
  User lab

Host 10.0.0.2
  HostName 10.0.0.2
  ProxyJump YO.UR.I.P
  User root
  ForwardAgent yes

```

## Configurations and installations

### SSH to `lab`

```bash
[home@home]

ssh lab@YO.UR.I.P

```

### Set sudo timeout to 120min (optional)

```bash
[lab@lab]

sudo visudo

Change line:
Defaults        env_reset

To:
Defaults        env_reset,timestamp_timeout=120
```

### Restrict sshd

```bash
[lab@lab]

sudo sed -i "s/#PermitRootLogin yes/PermitRootLogin no/g" /etc/ssh/sshd_config
sudo sed -i "s/#PubkeyAuthentication yes/PubkeyAuthentication yes/g" /etc/ssh/sshd_config
sudo sed -i "s/PasswordAuthentication yes/PasswordAuthentication no/g" /etc/ssh/sshd_config
sudo systemctl restart sshd

```

### Change password for user lab

```bash
[lab@lab]

sudo passwd lab

```

### Change password for user root

```bash
[lab@lab]

sudo passwd root

```

### Disable IPv6

```bash
[lab@lab]

sudo sed -i "s/quiet/quiet ipv6.disable=1/g" /etc/default/grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

```

### Back to `lab`

```bash
[home@home]

ssh lab@YO.UR.I.P

```

### Disable dhcpv6-client in firewall

```bash
[lab@lab]

sudo firewall-cmd --permanent --zone=public --remove-service=dhcpv6-client
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all

```

### Install and configure Cockpit

```bash
[lab@lab]

sudo yum -y install cockpit cockpit-dashboard cockpit-machines cockpit-networkmanager cockpit-packagekit cockpit-storaged
sudo systemctl enable --now cockpit.socket
sudo systemctl restart cockpit.socket

sudo firewall-cmd --permanent --zone=public --add-service=cockpit
sudo firewall-cmd --reload
```

### Install Virtualization Clients

```bash
[lab@lab]

sudo yum -y install wget git net-tools bind-utils bash-completion nfs-utils rsync qemu-kvm libvirt libvirt-python libguestfs-tools virt-install iscsi-initiator-utils 
sudo yum -y install @virtualization-client

```

### Add lab user to libvirt group

```bash
[lab@lab]

sudo usermod -aG libvirt lab
newgrp libvirt

```

### Update KVM installation

```bash
[lab@lab]

sudo bash -c 'cat << EOF > /etc/yum.repos.d/kvm-common.repo
[kvm-common]
name=KVM Common
baseurl=http://mirror.centos.org/centos/7/virt/x86_64/kvm-common/
gpgcheck=0
enabled=1
EOF'
sudo yum -y update

```

### Install Ansible, Git and lxml

```bash
[lab@lab]

sudo yum -y install epel-release
sudo yum -y install ansible git python-lxml

```

## Finally :-)

```bash
[lab@lab]

sudo reboot -h now

```

* * *

Next > [Provision infrastructure](02_provision_infrastructure.md)
