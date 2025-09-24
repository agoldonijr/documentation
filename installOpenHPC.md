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
dnf install http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/ohpc-release-3-1.el9.x86_64.rpm
dnf config-manager --set-enabled crb
```

# Installing 
```bash
dnf install ohpc-base
```
Now, you can update the system:
```bash
dnf update -y
xcatconfig --initialinstall --credentials --sshnodehostkeys
```

