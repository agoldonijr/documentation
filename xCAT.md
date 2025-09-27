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
dnf install xCAT
source /etc/profile.d/xcat.sh
```

## Rocky Linux 9.6
### Requirements
Install Rocky Linux 9.6
Please, do not change the minor version!

### Base Packages
```bash
dnf install -y tar pciutils bind-utils vim wget
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
dnf install -y xCAT
```
Now, you can update the system:
```bash
dnf update -y
xcatconfig --initinstall --credentials --sshnodehostkeys
```
