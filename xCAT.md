# Firsts steps to install xCAT and openHPC

# Requerments
Install Rocky Linux 9.3
Please, do not change the minor version!

# Base Packages
```bash
dnf install tar pciutils bind-utils vim wget
hostnamectl hostname andromeda
```

# Repos
```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
wget -P /etc/yum.repos.d http://xcat.org/files/xcat/repos/yum/devel/xcat-dep/rh9/x86_64/xcat-dep.repo
wget -P /etc/yum.repos.d https://xcat.org/files/xcat/repos/yum/2.17/xcat-core/xcat-core.repo
dnf config-manager --set-enabled crb
```

# Installing 
```bash
dnf install xCAT
```
Now, you can update the system:
```bash
dnf update -y
xcatconfig --initialinstall --credentials --sshnodehostkeys
```

# Create image
```bash
wget https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.6-x86_64-minimal.iso
```

