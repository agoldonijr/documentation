# First steps to install xCAT and openHPC

## Rocky Linux 8.10

### Requirements
Install Rocky Linux 8.10
Please, do not change the minor version!

### Base Packages
```bash
dnf install -y tar pciutils bind-utils vim wget
hostnamectl set-hostname andromeda
```
### Repos
```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
wget -P /etc/yum.repos.d http://xcat.org/files/xcat/repos/yum/devel/xcat-dep/rh8/x86_64/xcat-dep.repo
wget -P /etc/yum.repos.d https://xcat.org/files/xcat/repos/yum/2.17/xcat-core/xcat-core.repo
```

### Installing 
```bash
wget https://raw.githubusercontent.com/xcat2/xcat-core/master/xCAT-server/share/xcat/tools/go-xcat -O - > ~/go-xcat
chmod +x /tmp/go-xcat
/tmp/go-xcat -x devel install
source /etc/profile.d/xcat.sh

```

## Rocky Linux 9.6
### Requirements
Install Rocky Linux 9.6
Please, do not change the minor version!

### Base Packages
```bash
dnf install -y tar pciutils bind-utils vim wget initscripts
hostnamectl hostname andromeda
```

### Repos
```bash
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
wget -P /etc/yum.repos.d http://xcat.org/files/xcat/repos/yum/devel/xcat-dep/rh9/x86_64/xcat-dep.repo
wget -P /etc/yum.repos.d https://xcat.org/files/xcat/repos/yum/2.17/xcat-core/xcat-core.repo
dnf config-manager --set-enabled crb
```

### Installing 
```bash
wget https://raw.githubusercontent.com/xcat2/xcat-core/master/xCAT-server/share/xcat/tools/go-xcat -O - > /tmp/go-xcat
chmod +x /tmp/go-xcat
/tmp/go-xcat -x devel install
source /etc/profile.d/xcat.sh
```
## Network management interface
```bash
chdef -t site dhcpinterfaces="xcatmn|<interface_interna>"
chdef -t site domain=andromeda
```

## Create image

Download the OS imagem 
```bash
wget https://dl.rockylinux.org/vault/rocky/9.6/isos/x86_64/Rocky-9.6-x86_64-minimal.iso
copycds Rocky-9.6-x86_64-minimal.iso
```

To verify if the image has been created, use:
```bash
lsdef -t osimage
```
The output should be something like:
```bash
rocky9.6-x86_64-install-compute  (osimage)
rocky9.6-x86_64-install-service  (osimage)
rocky9.6-x86_64-netboot-compute  (osimage)
rocky9.6-x86_64-stateful-mgmtnode  (osimage)
rocky9.6-x86_64-statelite-compute  (osimage)
```

### Customizing image

Adding OHPC Repo:

```bash

chdef -t osimage rocky9.6-x86_64-netboot-compute pkgdir=/install/rocky9.6/x86_64/,https://repos.openhpc.community/OpenHPC/3/EL_9/,https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/,http://dl.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/,http://dl.rockylinux.org/pub/rocky/9/AppStream/x86_64/os/

```

If Nvidia driver will be necessary:
```bash

chdef -t osimage rocky9.6-x86_64-netboot-compute -p pkgdir=https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/
```


Create the directory and the file for the custom packages:
```bash
mkdir -p /install/common
touch /install/common/common.pkglist
```

Add the packages you need:
```bash
vim /install/common/common.pkglist

ohpc-base-compute
lmod-ohpc
chrony
ohpc-slurm-client
autofs
pciutils
vim
htop
slurm-libpmi-ohpc
```

Addint the package list to the image:
```bash
chdef -t osimage -o rocky9.6-x86_64-netboot-compute -p pkglist=/install/common/common.pkglist
```

Creating sync list file:

```bash
vim /install/common/compute.synclist

/etc/hosts -> /etc/hosts
/etc/passwd -> /etc/passwd
/etc/group -> /etc/group
/etc/shadow -> /etc/shadow
/etc/munge/munge.key -> /etc/munge/munge.key

```
Adding sync list file to the image:
```bash
chdef -t osimage -o rocky9.6-x86_64-netboot-compute -p synclists=/install/common/compute.synclist
```

Generate image:
```bash
genimage rocky9.6-x86_64-netboot-compute
packimage rocky9.6-x86_64-netboot-compute
```


### Adding nodes
```bash
mkdef -t node cn1 --template x86_64-template ip=192.168.0.3 mac=42:3d:0a:05:27:0c bmc=10.4.40.254 bmcusername=USERID bmcpassword=PASSW0RD
makehosts cn1
makedns -n
rpower cn1 on
rpower cn1 state
```

#### Start the Diskful OS Deployment
```bash
rinstall cn1 osimage=rocky9.6-x86_64-install-compute
makegocons cn1
rcons cn1
```



