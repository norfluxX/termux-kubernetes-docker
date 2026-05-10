# Termux QEMU Alpine + Docker + Kubernetes Setup

This guide provides step-by-step instructions to run **Alpine Linux** inside **QEMU** on **Termux**, and set up **Docker** and **K3s (lightweight Kubernetes)**.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Install QEMU in Termux](#install-qemu-in-termux)  
3. [Download Alpine Linux ISO](#download-alpine-linux-iso)  
4. [Create Alpine Disk and Start VM](#create-alpine-disk-and-start-vm)  
5. [Install Alpine Linux](#install-alpine-linux)  
6. [Install Docker](#install-docker)  
7. [Install K3s (Kubernetes)](#install-k3s-kubernetes)  
8. [Configure Environment](#configure-environment)  
9. [Auto-start Services](#auto-start-services)  
10. [Shutdown and Restart VM](#shutdown-and-restart-vm)  

---

## Prerequisites

- Termux installed on Android  
- Storage permissions granted:  

```sh
termux-setup-storage

Sufficient free storage (15GB+)

Recommended 2–4 CPU cores and 4GB RAM for QEMU VM



---

Install QEMU in Termux

pkg update && pkg upgrade -y
pkg install qemu-system-x86-64 wget tar -y


---

Download Alpine Linux ISO

mkdir -p ~/alpine && cd ~/alpine
wget https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-virt-3.14.0-x86_64.iso


---

Create Alpine Disk and Start VM

1. Create a disk image:


pkg install qemu-utils
qemu-img create -f qcow2 alpine.qcow2 15G

2. Create a start script bootup-alpine.sh:



#!/data/data/com.termux/files/usr/bin/bash
qemu-system-x86_64 \
  -smp 2 \
  -m 4096 \
  -drive file=$HOME/alpine/alpine.qcow2,if=virtio \
  -cdrom $HOME/alpine/alpine-virt-3.14.0-x86_64.iso \
  -netdev user,id=net0 \
  -device virtio-net-pci,netdev=net0 \
  -boot d \
  -nographic

3. Make it executable:



chmod +x bootup-alpine.sh

4. Run VM:



./bootup-alpine.sh


---

Install Alpine Linux

1. Login as root when VM boots.


2. Run Alpine setup:



setup-alpine

Select network, hostname, root password, and package mirror.

Use disk vda → sys → confirm erase.

Complete setup and reboot.



---

Install Docker

1. Update Alpine packages:


ip link set eth0 up
udhcpc -i eth0
cat << EOF >  /etc/apk/repositories
> https://dl-cdn.alpinelinux.org/alpine/v3.14/main
> https://dl-cdn.alpinelinux.org/alpine/v3.14/community
> EOF

apk update && apk upgrade -y

2. Install Docker:



apk add docker

3. Enable Docker at boot:



rc-update add docker default
service docker start

4. Verify Docker:



docker version
docker run hello-world


---

Install K3s (Kubernetes)

1. Install prerequisites:



apk add curl bash

2. Install k3s lightweight Kubernetes:



curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik --disable=servicelb" sh -

3. Enable k3s at boot:



rc-update add k3s default
service k3s start

4. Set KUBECONFIG permanently:



echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> /etc/profile
source /etc/profile

5. Verify cluster:



kubectl get nodes
kubectl get pods -A


---

Configure Environment

Check CPU cores:


nproc

Check RAM:


cat /proc/meminfo | grep MemTotal

Verify disk usage:


df -h


---

Auto-start Services

Check runlevel services:


rc-status

Ensure docker and k3s are listed under default and started.



---

Shutdown and Restart VM

Shutdown Alpine safely:


poweroff

start Alphine with disk
qemu-system-x86_64 \
  -smp 8 -m 6500 \
  -drive file=$HOME/alpine/alpine.qcow2,if=virtio \
  -netdev user,id=n1,hostfwd=tcp::2222-:22 \
  -device virtio-net,netdev=n1 \
  -nographic


Stop QEMU process (if still running):


ps aux | grep qemu
kill <PID>

Restart VM:


./start-alpine.sh

After boot, services start automatically, and kubectl works with exported KUBECONFIG.
