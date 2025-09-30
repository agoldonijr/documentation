# BeeGFS Installation


### Base packages

```bash
hostnamectl hostname andromeda
dnf update -y 
dnf install tar pciutils bind-utils vim wget
```

### ZFS packages

```bash
dnf install epel-release 
dnf install https://zfsonlinux.org/epel/zfs-release-2-8$(rpm --eval "%{dist}").noarch.rpm
dnf install -y epel-release
dnf install -y kernel-devel
dnf install -y dkms
dnf install -y zfs
dkms autoinstall
reboot
modprobe zfs
lsmod | grep zfs
```

## Metadata pool

Creating metadata pool:
```bash
zpool create -f -m /data/meta001 meta001 mirror sdd sdb
```
## Storage pool

Creating storage pool:

```bash
zpool create -m /data/storage001 storage001 raidz2 /dev/sdc /dev/sde /dev/sdf /dev/sdg
```


## Install Beegfs

Configuring repositories:

```bash
rpm --import https://www.beegfs.io/release/beegfs_7.4.6/gpg/GPG-KEY-beegfs
wget https://www.beegfs.io/release/beegfs_7.4.6/dists/beegfs-rhel9.repo -O /etc/yum.repos.d/beegfs.repo
```

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
dd if=/dev/random of=/etc/beegfs/connauthfile bs=128 count=1
chown root:root /etc/beegfs/connauthfile
chmod 400 /etc/beegfs/connauthfile
```
You need to edit the /etc/beegfs/beegfs-mgmtd.conf for Management and Meta.

Management:
```bash
vim /etc/beegfs/beegfs-mgmtd.conf 
connAuthFile                           = /etc/beegfs/connauthfile
```
Meta:
```bash
vim /etc/beegfs/beegfs-meta.conf 
connAuthFile                           = /etc/beegfs/connauthfile
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
connAuthFile                           = /etc/beegfs/connauthfile
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
connAuthFile                 = /etc/beegfs/connauthfile


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

