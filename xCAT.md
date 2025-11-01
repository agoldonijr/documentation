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
wget https://raw.githubusercontent.com/xcat2/xcat-core/master/xCAT-server/share/xcat/tools/go-xcat -O - > ~/go-xcat
chmod +x /tmp/go-xcat
/tmp/go-xcat -x devel install
source /etc/profile.d/xcat.sh
```
### Network management interface
```bash
chdef -t site dhcpinterfaces="xcatmn|<interface_interna>"
chdef -t site domain=andromeda
```

### Create image

Download the OS imagem 
```bash
wget https://download.rockylinux.org/pub/rocky/10/isos/x86_64/Rocky-10.0-x86_64-minimal.iso 
copycds Rocky-10.0-x86_64-minimal.iso
```

To verify if the image has been created, use:
```bash
lsdef -t osimage
```
The output should be somthing like:
```bash
rocky9.6-x86_64-install-compute  (osimage)
rocky9.6-x86_64-install-service  (osimage)
rocky9.6-x86_64-netboot-compute  (osimage)
rocky9.6-x86_64-stateful-mgmtnode  (osimage)
rocky9.6-x86_64-statelite-compute  (osimage)
```

