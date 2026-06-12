#!/bin/bash

echo Install basic Debian package build requirements:
apt install -y  build-essential fakeroot devscripts equivs

echo Installing prerequisistes

apt install -y freeipmi libfreeipmi-dev   > /dev/null 2>&1
apt install -y libibmad-dev libibumad-dev > /dev/null 2>&1
apt install -y h5utils libhdf5-dev        > /dev/null 2>&1
apt install -y libjwt-dev                 > /dev/null 2>&1
apt install -y libmunge-dev               > /dev/null 2>&1
apt install -y libnvidia-ml-dev           > /dev/null 2>&1
apt install -y libvpl-dev                 > /dev/null 2>&1
apt install -y librocm-smi-dev            > /dev/null 2>&1
apt install -y man2html                   > /dev/null 2>&1
apt install -y lua5.4                     > /dev/null 2>&1
apt install -y libpam-slurm-dev           > /dev/null 2>&1
apt install -y libpmix-dev                > /dev/null 2>&1
apt install -y libreadline-dev            > /dev/null 2>&1
apt install -y libhttp-parser-dev         > /dev/null 2>&1
apt install -y libjson-c-dev              > /dev/null 2>&1
apt install -y libyaml-dev                > /dev/null 2>&1
apt install -y libgtk-4-dev               > /dev/null 2>&1
apt install -y curl                       > /dev/null 2>&1
apt install -y libnuma-dev                > /dev/null 2>&1
apt install -y libhwloc-dev               > /dev/null 2>&1
apt install -y libbpf-dev libbpf-tools    > /dev/null 2>&1
apt install -y libdbus-1-dev              > /dev/null 2>&1


echo Missing...
echo cray-libcxi
echo s2n

wget https://github.com/SchedMD/slurm/releases/download/slurm-25-11-5-1/slurm-25.11.5.tar.bz2

tar -xaf slurm-25.11.5.tar.bz2
cd slurm-25.11.5

mk-build-deps -i debian/control
debuild -b -uc -us

