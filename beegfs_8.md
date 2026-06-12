# BeeGFS Installation


### Base packages

```bash
[root@andromeda ~]# hostnamectl hostname andromeda
[root@andromeda ~]# dnf update -y 
[root@andromeda ~]# dnf -y install tar pciutils bind-utils vim wget
[root@andromeda ~]# dnf install kernel-devel

```

## Metadata pool

Creating metadata pool:
```bash
[root@andromeda ~]# zpool create -f -m /data/meta001 meta001 mirror sdd sdb
```
## Storage pool

Creating storage pool:

```bash
[root@andromeda ~]# zpool create -m /data/storage001 storage001 raidz2 /dev/sdc /dev/sde /dev/sdf /dev/sdg
```


## Install Beegfs

Configuring repositories:

```bash
[root@andromeda ~]# wget -O /etc/yum.repos.d/beegfs-rhel9.repo http://www.beegfs.com/release/beegfs_8.0.0/dists/beegfs-rhel9.repo

```
[root@andromeda ~]# firewall-cmd --zone=public --permanent --add-port=8000-8008/tcp --add-port=8000-8008/udp
[root@andromeda ~]# firewall-cmd --zone=public --permanent --add-port=8010/tcp --add-port=8010/udp
[root@andromeda ~]# firewall-cmd --reload

[root@andromeda ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk
sdc           8:32   0   10G  0 disk

[root@andromeda ~]# parted /dev/sdb mklabel gpt && parted /dev/sdb mkpart primary 0% 100%
[root@andromeda ~]# parted /dev/sdc mklabel gpt && parted /dev/sdc mkpart primary 0% 50% && parted /dev/sdc mkpart primary 50% 100%
[root@andromeda ~]# mkfs.xfs -K /dev/sdb1
[root@andromeda ~]# mkfs.xfs -K /dev/sdc1

[root@andromeda beegfs]# mkdir -p /data/beegfs/beegfs_meta
[root@andromeda beegfs]# mkdir -p /data/beegfs/beegfs_storage

[root@andromeda beegfs]# mount /dev/sdc1 /data/beegfs/beegfs_meta
[root@andromeda beegfs]# mount /dev/sdb1 /data/beegfs/beegfs_storage

[root@andromeda beegfs]# lsblk 
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part 
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk 
└─sdb1        8:17   0   10G  0 part /data/beegfs/beegfs_storage
sdc           8:32   0   10G  0 disk 
├─sdc1        8:33   0    5G  0 part /data/beegfs/beegfs_meta
└─sdc2        8:34   0    5G  0 part 
sr0          11:0    1 1024M  0 rom  

[root@andromeda ~]# dnf install -y beegfs-mgmtd libbeegfs-license

[root@andromeda ~]# dd if=/dev/random of=/etc/beegfs/conn.auth bs=128 count=1
[root@andromeda beegfs]# chown root:root /etc/beegfs/conn.auth
[root@andromeda beegfs]# chmod 400 /etc/beegfs/conn.auth


[root@andromeda ~]# cd /etc/beegfs/
[root@andromeda ~]# vim /etc/beegfs/san.cnf
[ req ]
default_bits = 2048
distinguished_name = req_distinguished_name
req_extensions = req_ext
x509_extensions = v3_ca # The extensions to add to the self signed cert

[ req_distinguished_name ]
commonName =  andromeda(eg, fully qualified host name)
commonName_default = andromeda

[ req_ext ]
subjectAltName = @alt_names

[ v3_ca ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = localhost
IP.1 = 127.0.0.1

[root@andromeda beegfs]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -config /etc/beegfs/san.cnf
[root@andromeda ~]# dnf install -y sqlite


[root@andromeda beegfs]# vim /etc/beegfs/beegfs-mgmtd.toml 
auth-file = "/etc/beegfs/conn.auth"

[root@andromeda beegfs]# /opt/beegfs/sbin/beegfs-mgmtd --init
Created new database version 3 at "/var/lib/beegfs/mgmtd.sqlite".


[root@andromeda beegfs]# systemctl start beegfs-mgmtd ; systemctl status beegfs-mgmtd
● beegfs-mgmtd.service - BeeGFS Management Server
     Loaded: loaded (/usr/lib/systemd/system/beegfs-mgmtd.service; disabled; preset: disabled)
     Active: active (running) since Wed 2026-04-15 17:21:45 -03; 15ms ago
       Docs: https://doc.beegfs.io
   Main PID: 1539 (beegfs-mgmtd)
      Tasks: 10 (limit: 10652)
     Memory: 4.4M (peak: 4.6M)
        CPU: 8ms
     CGroup: /system.slice/beegfs-mgmtd.service
             └─1539 /opt/beegfs/sbin/beegfs-mgmtd --log-target=journald

Apr 15 17:21:45 andromeda systemd[1]: Starting BeeGFS Management Server...
Apr 15 17:21:45 andromeda beegfs-mgmtd[1539]: Initializing licensing library failed. Licensed features will be unavailable: Reading certificate file "/etc/beegfs/license.pem" failed
Apr 15 17:21:45 andromeda systemd[1]: Started BeeGFS Management Server.
Apr 15 17:21:45 andromeda beegfs-mgmtd[1539]: Waiting for shutdown signal ...



[root@andromeda beegfs]# dnf install -y beegfs-meta

[root@andromeda data]# /opt/beegfs/sbin/beegfs-setup-meta -p /data/beegfs/beegfs_meta -s 2 -m andromeda
Preparing storage directory: /data/beegfs/beegfs_meta
 * Creating format.conf file...
 * Creating server numeric ID file: /data/beegfs/beegfs_meta/nodeNumID
Updating config file: /etc/beegfs/beegfs-meta.conf
 * Setting management host: andromeda
 * Setting storage directory in config file...
 * Disabling usage of uninitialized storage directory in config file...
 * Fetching the underlying device...
Underlying device detected: /dev/sdc1
Fetching UUID of the file system on that device...
Found UUID d70a3d30-0c72-4819-bb57-a8e42872a164
Writing UUID to config file...
 * Setting usage of extended attributes to: true
All done.



[root@andromeda ~]# dnf install -y  beegfs-storage libbeegfs-ib
[root@andromeda ~]# vim /etc/beegfs/beegfs-storage.conf 
auth-file = "/etc/beegfs/conn.auth"

[root@andromeda ~]# /opt/beegfs/sbin/beegfs-setup-storage -f -p /data/beegfs/beegfs_storage -s 3 -i 301 -m andromeda
[root@andromeda data]# systemctl restart beegfs-storage ; systemctl status beegfs-storage
● beegfs-storage.service - BeeGFS Storage Server
     Loaded: loaded (/usr/lib/systemd/system/beegfs-storage.service; enabled; preset: disabled)
     Active: active (running) since Wed 2026-04-15 18:01:16 -03; 37ms ago
       Docs: http://www.beegfs.com/content/documentation/
   Main PID: 2010 (beegfs-storage/)
      Tasks: 1 (limit: 10652)
     Memory: 556.0K (peak: 808.0K)
        CPU: 19ms
     CGroup: /system.slice/beegfs-storage.service
             └─2010 /opt/beegfs/sbin/beegfs-storage cfgFile=/etc/beegfs/beegfs-storage.conf runDaemonized=false

Apr 15 18:01:16 andromeda systemd[1]: Started BeeGFS Storage Server.
Apr 15 18:01:16 andromeda beegfs-storage[2010]: Main [App] >> Built with NVFS RDMA support.


[root@andromeda ~]# dnf install -y kernel-devel-$(uname -r)
[root@andromeda ~]# mkdir -p /mnt/beegfs
[root@andromeda ~]# dnf install -y beegfs-client beegfs-tools beegfs-utils
[root@andromeda ~]# vim /etc/beegfs/beegfs-client.conf 
connAuthFile                  = /etc/beegfs/conn.auth

[root@andromeda ~]# /opt/beegfs/sbin/beegfs-setup-client -m andromeda
Updating config file: /etc/beegfs/beegfs-client.conf
 * Setting management host: andromeda
All done.


[root@andromeda beegfs]# systemctl start beegfs-client

[root@andromeda beegfs]# systemctl enable --now beegfs-mgmtd beegfs-meta beegfs-storage beegfs-client 

===============================================
Monitoramento

Influx DB 2
```bash
curl --silent --location -O https://repos.influxdata.com/influxdata-archive.key
gpg --show-keys --with-fingerprint --with-colons ./influxdata-archive.key 2>&1 \
| grep -q '^fpr:\+24C975CBA61A024EE1B631787C3D57159FC2F927:$' \
&& cat influxdata-archive.key \
| gpg --dearmor \
| tee /etc/pki/rpm-gpg/RPM-GPG-KEY-influxdata > /dev/null
```


```bash
[root@andromeda ~]# 
cat <<EOF | tee /etc/yum.repos.d/influxdata.repo
[influxdata]
name = InfluxData Repository - Stable
baseurl = https://repos.influxdata.com/stable/\${basearch}/main
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-influxdata
EOF
```
[root@andromeda ~]# dnf install -y influxdb2
[root@andromeda ~]# systemctl enable --now influxdb



[root@andromeda ~]# vim /etc/default/influxdb2

INFLUXD_CONFIG_PATH=/etc/influxdb/config.toml
ARG1="--http-bind-address :8087"
ARG2="--storage-wal-fsync-delay=15m"

[root@andromeda ~]# vim /lib/systemd/system/influxdb.service
ExecStart=/usr/lib/influxdb/scripts/influxd-systemd-start.sh $ARG1 $ARG2

[root@andromeda ~]# firewall-cmd --zone=public --permanent --add-port=8086/tcp --add-port=8086/udp
[root@andromeda ~]# firewall-cmd --reload

http://192.168.0.101:8086/
username: admin
password: 123qweasd
initial organizazion: beegfs_mon
initial bucketa: beegfs_mon
Token: YEzRDIhJiaxzdcyCarOT0ke4EbKnRBwMgd0_ECBkGMM4gamxZvy4prWvpY4iaLR6zB0Ra1OnWez331ldxf1okA==


Grafana

[root@andromeda ~]# wget -q -O gpg.key https://rpm.grafana.com/gpg.key
[root@andromeda ~]# rpm --import gpg.key
[root@andromeda ~]# vim /etc/yum.repos.d/grafana.repo

[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt

[root@andromeda ~]# dnf install -y grafana

[root@andromeda ~]# firewall-cmd --zone=public --permanent --add-port=3000/tcp --add-port=3000/udp
[root@andromeda ~]# firewall-cmd --reload
[root@andromeda ~]# systemctl enable --now grafana-server


http://192.168.0.101:8086/
username: admin
senha: admin

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/influxdata.repo
[influxdata]
name = InfluxData Repository - Stable
baseurl = https://repos.influxdata.com/stable/\$basearch/main
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-influxdata
EOF
[influxdata]
name = InfluxData Repository - Stable
baseurl = https://repos.influxdata.com/stable/$basearch/main
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-influxdata
```


Telegraf
[root@andromeda ~]# dnf install -y telegraf
[root@andromeda ~]#  vim /etc/telegraf/telegraf.d/beegfs_mon_telegraf.conf

[[outputs.influxdb_v2]]
urls = ["http://localhost:8086"] # Replace with the actual InfluxDB URL
token = "YEzRDIhJiaxzdcyCarOT0ke4EbKnRBwMgd0_ECBkGMM4gamxZvy4prWvpY4iaLR6zB0Ra1OnWez331ldxf1okA==" # Replace with your InfluxDB 2.x token
organization = "beegfs_mon" # Replace with your InfluxDB 2.x organization
bucket = "beegfs_mon" # Replace with your InfluxDB 2.x bucket

[[inputs.cpu]]
percpu = true
totalcpu = true
collect_cpu_time = false
report_active = false
core_tags = false

[[inputs.disk]]
ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.system]]

[root@andromeda ~]# systemctl enable --now telegraf
[root@andromeda ~]# dnf install -y beegfs-mon

[root@andromeda ~]# vim /etc/beegfs/beegfs-mon.auth
username = admin
password = 123qweasd
organization = beegfs-db
token = SqzDBU752P_QeSck9vTK2gXvfy_T1yxDlbiwDf8AylApgnWVrRKLKnCYQBlRe3fqc9HefhrhCyiQQENTDjuwnQ==


[root@andromeda ~]# vim /etc/beegfs/beegfs-mon.conf 
sysMgmtdHost                 = andromeda

dbType                       = influxdb2
dbHostName                   = 192.168.0.101
dbHostPort                   = 8086
dbAuthFile                   = /etc/beegfs/beegfs-mon.auth

dbBucket                     = beegfs-db

[root@andromeda ~]# systemctl enable --now beegfs-mon
[root@andromeda ~]# systemctl status beegfs-mon


[root@andromeda ~]# dnf install -y beegfs-mon-grafana
[root@andromeda ~]# cd /opt/beegfs/scripts/grafana
[root@andromeda grafana]# ./import-dashboards default





















### Management and Meta

Install packages:
```bash
dnf  install beegfs-mgmtd 
dnf install beegfs-meta libbeegfs-ib 

```

Management configuration:
```bash
/opt/beegfs/sbin/beegfs-setup-mgmtd -p /data/mgmtd
```

Meta configuration:
```bash
/opt/beegfs/sbin/beegfs-setup-meta -p /data/meta001 -s 2 -m andromeda
```
### Authentication 
Creating ConnAuthentication File:

```bash
dd if=/dev/random of=/etc/beegfs/conn.auth bs=128 count=1
chown root:root /etc/beegfs/conn.auth
chmod 400 /etc/beegfs/conn.auth
```
You need to edit the /etc/beegfs/beegfs-mgmtd.conf for Management and Meta.

Management:
```bash
vim /etc/beegfs/beegfs-mgmtd.conf 
connAuthFile                           = /etc/beegfs/conn.auth
```
Meta:
```bash
vim /etc/beegfs/beegfs-meta.conf 
connAuthFile                           = /etc/beegfs/conn.auth
```


### Storage

Install packages:

```bash
dnf install beegfs-storage libbeegfs-ib 
```

Storage configuration:

```bash
/opt/beegfs/sbin/beegfs-setup-storage -p /data/storage001 -s 3 -i 301 -m andromeda
```

Storage ConnAuthentication: 

```bash
vim /etc/beegfs/beegfs-storage.conf
connAuthFile                           = /etc/beegfs/conn.auth
```


### Client

Install packages:

```bash
dnf install beegfs-client beegfs-helperd beegfs-utils
```

Storage configuration:
```bash
/opt/beegfs/sbin/beegfs-setup-client -m andromeda

```


## Monitoring

### InfluxDB

Installing InfluxDB OSS:
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 0
gpgkey = https://repos.influxdata.com/influxdata-archive.key
EOF

dnf install influxdb

cd /var/lib/influxdb
chown -R influxdb: influxdb
systemctl start influxdb
systemctl status influxdb
```
Creating a user and password:
```bash
influx
CREATE USER beegfs_mon WITH PASSWORD '123qwe' WITH ALL PRIVILEGES
quit
```
To test the user you just created, type:

```bash
influx -username 'beegfs_mon' -password '123qwe'
CREATE DATABASE beegfs_mon
```

Now that InfluxDB is running, we need to install and configure beegfs-mon.


```bash
dnf install beegfs-mon

sysMgmtdHost                 = andromeda
dbHostName                   = andromeda
connAuthFile                 = /etc/beegfs/conn.auth


vim /etc/beegfs/beegfs-mon.auth

username = beegfs_mon
password = 123qwe

```
### Grafana

If you want to use the prebuilt Grafana panels, you also need Grafana.

```bash
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
rpm --import gpg.key
cat <<EOF | sudo tee /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
 EOF

dnf install grafana

systemctl enable --now grafana-server.service

```
To access Grafana, type `<IP>:3000` in a browser. The defautl user and password are "admin" and "admin".


### Telegraf

Install Telegraf to collect the data:

```bash
dnf search telegraf

vim /etc/telegraf/telegraf.d/beegfs_mon_telegraf.conf

[[outputs.influxdb]]
urls = ["http://localhost:8086"] # Replace with the actual InfluxDB URL
database = "beegfs_mon" # Replace with your desired InfluxDB database name
username = "beegfs_mon" # Replace with your InfluxDB username
password = "123qwe" # Replace with your InfluxDB password

[[inputs.cpu]]
percpu = true
totalcpu = true
collect_cpu_time = false
report_active = false
core_tags = false

[[inputs.disk]]
ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.system]]

```

After this configuration, start the service:

```bash
systemctl start telegraf
systemctl start beegfs-mon
```
A set of Grafana panels for use with BeeGFS is provided by the beegfs-mon-grafana package. Once it is installed they can be imported using the script /opt/beegfs/scripts/grafana/import-dashboards. For the out-of-the-box setup with InfluxDB and Grafana being on the same host, just use:


```bash
dnf install beegfs-mon-grafana
cd /opt/beegfs/scripts/grafana
./import-dashboards default


Select an option:
1. Using BeeGFS Monitoring with Telegraf
2. Using BeeGFS Monitoring without Telegraf
Enter your Option: 1
Please select influxdb version:
1) Influxdb 1.x
2) Influxdb 2.x
Enter your influxdb Verion: 1
Enter Database Name: beegfs_mon
Enter Database User: beegfs_mon

```

