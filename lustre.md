# Lustre Installation

https://www.lustre.org/wp-content/uploads/architecting-lustre-storage-white-paper.pdf
Versoes aceitas:

https://wiki.lustre.org/Lustre_with_Virtualbox_install]


Server: 
Rocky Linux 8.10 + Lustre 2.15
Rocky Linux 9.7 + Lustre 2.17
### Base packages

```bash
hostnamectl hostname andromeda
bash
dnf update -y
dnf install -y createrepo_c

```

### ZFS packages

```bash
dnf install -y https://zfsonlinux.org/epel/zfs-release-2-8$(rpm --eval "%{dist}").noarch.rpm
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
dnf install -y dkms
dnf install -y zfs
dkms autoinstall
modprobe zfs
lsmod | grep zfs
```


Creating the repo:
```bash
cat >/etc/yum.repos.d/lustre-repo.conf <<EOF
[lustre-server]
name=lustre-server
baseurl=https://downloads.whamcloud.com/public/lustre/lustre-2.17.0/el9.7/server
# exclude=*debuginfo*
gpgcheck=0

[lustre-client]
name=lustre-client
baseurl=https://downloads.whamcloud.com/public/lustre/lustre-2.17.0/el9.7/client
# exclude=*debuginfo*
gpgcheck=0

[e2fsprogs-wc]
name=e2fsprogs-wc
baseurl=https://downloads.whamcloud.com/public/e2fsprogs/latest/el9
# exclude=*debuginfo*
gpgcheck=0
EOF
```
Download Lustre repo to the server:

```bash
mkdir -p /var/www/html/repo
cd /var/www/html/repo
reposync -c /etc/yum.repos.d/lustre-repo2.conf \
--repoid=lustre-server \
    --repoid=lustre-client \
    --repoid=e2fsprogs-wc \
    --newest-only \
--download-path=/var/www/html/repo
```

After the command, there are tree folders following bellow:
```bash
[root@andromeda ~]# cd /var/www/html/repo
[root@andromeda repo]# ls -l
total 0
drwxr-xr-x. 4 root root 31 Apr 11 01:19 e2fsprogs-wc
drwxr-xr-x. 4 root root 31 Apr 11 01:18 lustre-client
drwxr-xr-x. 4 root root 31 Apr 11 01:13 lustre-server

```
Creating repos:
```bash
[root@andromeda repo]# for i in e2fsprogs-wc lustre-client lustre-server; do
    createrepo_c -v $i/
done
[root@andromeda repo]# hn=`hostname --fqdn`
cat >/var/www/html/lustre.repo <<EOF
[lustre-server]
name=lustre-server
baseurl=https://$hn/repo/lustre-server
enabled=1
gpgcheck=0
proxy=_none_

[lustre-client]
name=lustre-client
baseurl=https://$hn/repo/lustre-client
enabled=1
gpgcheck=0

[e2fsprogs-wc]
name=e2fsprogs-wc
baseurl=https://$hn/repo/e2fsprogs-wc
enabled=1
gpgcheck=0
EOF

[root@andromeda repo]# ls -l /var/www/html/lustre.repo
-rw-r--r--. 1 root root 315 Apr 11 01:44 /var/www/html/lustre.repo

```
```bash
