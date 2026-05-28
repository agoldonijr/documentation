
Listando imagens:
```bash
[root@bcmhead ~]# cmsh -c "softwareimage; list"
Name (key)           Path (key)                               Kernel version                Nodes   
-------------------- ---------------------------------------- ----------------------------- --------
default-image        /cm/images/default-image                 5.14.0-570.17.1.el9_6.x86_64  13      
[bcmhead->softwareimage]% exit
[bcmhead]% exit
```

Criando imagens:

```bash
[root@bcmhead ~]# cmsh -c "softwareimage; clone default-image login-image; commit"
[root@bcmhead ~]# cmsh -c "softwareimage; list"
Name (key)           Path (key)                               Kernel version                Nodes   
-------------------- ---------------------------------------- ----------------------------- --------
default-image        /cm/images/default-image                 5.14.0-570.17.1.el9_6.x86_64  13      
login-image          /cm/images/login-image                   5.14.0-570.17.1.el9_6.x86_64  0    
```
Instalando pacotes na imagem criada:

```bash
[root@bcmhead ~]# cm-chroot-sw-img /cm/images/login-image
[root@login-image /]# hostname
login-image
[root@login-image /]# dnf install -y htop tmux vim git pdsh rsync
[root@login-image /]# dnf install -y slurm24.11 slurm24.11-slurmd slurm24.11-slurmctld
[root@login-image /]# exit
[root@bcmhead ~]# cmsh -c "softwareimage; use login-image; commit"
```
Instalando módulos

Exemplo de instalação do modulo do CUDA 12.9:
```bash
[root@login-image /]# dnf install -y cm-cudnn9.15-cuda12.9
```

Exemplo de instalação do modulo do OpenBlas:
```bash
[root@login-image /]# dnf install -y cm-openblas
```
Administração de usuários:

Criando um usuário comum:
```bash
cmsh
user
add novo.usuario
set commonname Nome
set surname Sobrenome
set loginshell /bin/bash
set homedirectory /home/novo.usuario
set homedirectoryoperation yes
commit
exit
exit
```
```bash
cmsh
user
use novo.usuario
set password
commit
exit
exit
```
Para que as atualizações façam efeito:
```bash
systemctl restart cmd
```
Verificando as informações:
```bash
getent passwd novo.usuario
id novo.usuario
su - novo.usuario
id
exit
```

Para usuário administrador:
```bash
cmsh
group
use cluster-admins
append members novo.usuario
commit
exit
exit
```
Para que a mudança seja aplicada:
```bash
systemctl restart cmd
```
Validando informações:
```bash
id novo.usuario
su - novo.usuario
sudo -l
exit
```
Para remover admin:

```bash
cmsh
group
use cluster-admins
removefrom members novo.usuario
commit
exit
exit
```
Para que a mudança seja aplicada:
```bash
systemctl restart cmd
```
Validando informações:
```bash
id novo.usuario
sudo -l -U novo.usuario
```
