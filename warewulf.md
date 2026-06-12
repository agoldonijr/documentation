# Warewult installation

## Base packages

```bash
dnf install -y vim git tar pciutils bind-utils wget
dnf config-manager --set-enabled crb
```

## Disable firewall

```bash
systemctl stop firewalld
systemctl disable firewalld
```

## Installing
```bash
dnf install https://github.com/warewulf/warewulf/releases/download/v4.6.4/warewulf-4.6.4-1.el9.x86_64.rpm

```

## Configuration
```bash
cp /etc/warewulf/warewulf.conf /etc/warewulf/warewulf.conf_orig
```

## Node image

```bash
wwctl image import docker://ghcr.io/warewulf/warewulf-rockylinux:9 rockylinux-9 --build
wwctl profile set default --image rockylinux-9

```

