# Installing Nvidia driver

```bash
chmod +x NVIDIA-Linux-x86_64-xxx.xx.xx.run
bash ./NVIDIA-Linux-x86_64-xxx.xx.xx.run
￼
reboot
```
Make sure that it is installed￼
```bash
nvidia-smi
```
￼
# Installing Docker
```bash
dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker
```
Testing the Docker installation

```bash
docker run hello-world
```

# Install NVIDIA Container Toolkit packages
```bash
dnf install -y curl
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | tee /etc/yum.repos.d/nvidia-container-toolkit.repo
dnf-config-manager --enable nvidia-container-toolkit-experimental

export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.1-1
dnf install -y \
      nvidia-container-toolkit-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1-${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

# Configuration
### Prerequisites
You installed a supported container engine (Docker, Containerd, CRI-O, Podman).
You installed the NVIDIA Container Toolkit.

```bash
nvidia-ctk runtime configure --runtime=docker
```
The nvidia-ctk command modifies the /etc/docker/daemon.json file on the host. The file is updated so that Docker can use the NVIDIA Container Runtime

```bash
systemctl restart docker
```
#Rootless mode

```bash
mkdir -p /var/local/docker/
nvidia-ctk runtime configure --runtime=docker --config=/var/local/docker/daemon.json
systemctl --user restart docker
```

Configure /etc/nvidia-container-runtime/config.toml by using the nvidia-ctk command:
```bash
nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
```

# Configuring CRI-O

By default, the nvidia-ctk command creates a /etc/crio/conf.d/99-nvidia.toml drop-in config file. The drop-in file ensures that CRI-O can use the NVIDIA Container Runtime.
```bash
nvidia-ctk runtime configure --runtime=crio
systemctl restart crio

===================================

Install xCat or other cluster manager

===================================

# Verifying the Installation of NVIDIA Container Toolkit
```bash
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
