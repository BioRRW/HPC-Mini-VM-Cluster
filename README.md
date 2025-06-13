# HPC-Mini-VM-Cluster
Repository documenting an HPC mini VM cluster I am building on a spare laptop - for fun!


# 🖥️ Mini HPC Cluster on a Laptop Using KVM + Slurm + Rocky Linux

This project sets up a miniature High-Performance Computing (HPC) cluster on a laptop using KVM virtualization, Rocky Linux 9.6, SLURM workload manager, and optional GPU passthrough. The goal is to simulate real-world HPC system management and learn HPC administration best practices.

---


## Progress

This project has just been initiated and is in the planning phase. 

TODO:

| Category            | Task                                                                     | Status     |
| ------------------- | ------------------------------------------------------------------------ | ---------- |
| ✅ Base OS Setup     | Install **Rocky Linux 9.6** on host machine                              | ✔ Done     |
| ✅ Disk Setup        | Custom partitioning per best practices (XFS, LVM, EFI, `/var/log`, etc.) | ✔ Done     |
| 🖧 Networking       | Set up Ethernet and WiFi: [Notes here](networking.md)                      | ✔ Done     |
| 🔐 System Hardening | Disable root SSH login. See [Hardening notes](hardening.md)               | ✔ Done     |
| 🔐 System Hardening | Configure SSH key for admin as a test for later                           | ✔ Done     |
| 🔐 System Hardening | Enable firewall (`firewalld` or `nftables`)                              | ✔ Done     |
| 🔐 System Hardening | Set up `fail2ban` or ssh rate-limiting                                   | ☐ Optional |
| 🧠 Virtualization   | Install and configure KVM, libvirt, and `virt-manager`                   | ☐ Todo     |
| 🖧 Networking       | Set up bridge networking (`br0`) for VMs                                 | ☐ Todo     |
| 🖥️ VM Creation     | Create 1 head node + 12 compute node VMs + 1 GPU VM                      | ☐ Todo     |
| 🧪 VM Testing       | Ensure all VMs boot, have network access, and SSH in/out works           | ☐ Todo     |
| ⚙ SLURM Setup       | Install SLURM on head node, configure compute nodes                      | ☐ Todo     |
| 🧪 Job Testing      | Submit basic test jobs via SLURM                                         | ☐ Todo     |
| 🎮 GPU Integration  | Set up passthrough VM w/ NVIDIA drivers + CUDA toolkit                   | ☐ Todo     |
| 📦 File Sharing     | (Optional) Set up NFS for `/home`, `/scratch`                            | ☐ Optional |
| 📊 Monitoring       | (Optional) Install `htop`, `nmon`, or Prometheus stack                   | ☐ Optional |
| 🧹 Automation       | Set up `virt-clone`, `cloud-init`, and Ansible roles                     | ☐ Todo     |
| 🧼 Maintenance      | Script backups, log rotation, and regular cluster health checks          | ☐ Todo     |

---


## 🧰 System Overview

One of many laptops I have sitting around is a semi-beefy System 76 machine which has the below specs.

| Component          | Detail                                                      |
|-------------------|-------------------------------------------------------------|
| **Host OS**        | Rocky Linux 9.6 (minimal install)                           |
| **Laptop CPU**     | Intel Core i9-13900HX — 24 cores / 32 threads               |
| **Memory**         | 32 GB DDR5                                                  |
| **GPU**            | NVIDIA RTX A2000 — Ampere (Passthrough Capable, 6 GB VRAM) |
| **Hypervisor**     | KVM + libvirt + virt-manager                                |
| **Networking**     | Realtek RTL8125 — bridged networking for cluster nodes      |
| **Cluster Design** | 1 head node + 12 compute nodes + 1 GPU node (13 total VMs)  |

---

## 🧱 Disk Partition Plan

```
Target Disk: /dev/nvme0n1
```

| Mount Point        | Size     | Scheme       | Filesystem  | Volume Label | Notes                                                        |
|--------------------|----------|--------------|-------------|---------------|--------------------------------------------------------------|
| `/boot/efi`        | 1 GB     | Standard     | EFI (FAT32) | `EFI`         | UEFI system partition — required for UEFI booting            |
| `/boot`            | 1 GB     | Standard     | XFS         | `BOOT`        | Separate /boot needed, must be **non-LVM**                   |
| `swap`             | 16 GB    | LVM          | swap        | `SWAP`        | Standard swap                                                |
| `/`                | 120 GB   | LVM          | XFS         | `ROOT`        | Main OS and software                                         |
| `/var/lib/libvirt` | 350 GB   | LVM Thin     | XFS         | `VM_IMAGES`   | Thin pool for VM images and snapshots                        |
| `/home`            | 150 GB   | LVM          | XFS         | `HOME`        | User home directories, shared storage                        |
| `/var/log`         | 30 GB    | LVM          | XFS         | `LOGS`        | Prevents log bloat from filling `/`                          |
| **Unallocated**    | ~330 GB  | —            | —           | —             | Reserved for expansion, scratch space, or future volumes     |

---

## 🗃️ Cluster Layout

| Node        | CPUs | RAM   | Notes                                |
|-------------|------|-------|--------------------------------------|
| `head-node` | 1    | 1 GB  | SLURM controller, login node         |
| `node01`    | 1    | 1 GB  | Compute node                         |
| `node02`    | 1    | 1 GB  | Compute node                         |
| `...`       | ...  | ...   | ...                                  |
| `node12`    | 1    | 1 GB  | Compute node                         |
| `gpu-node`  | 4    | 4 GB  | GPU passthrough (NVIDIA RTX A2000)   |

---

## ⚙️ Features & Configuration Highlights

- 🔒 **No root SSH access**: Admin and user accounts (`user1`, ..., `usern`) only
- 🐧 **SLURM Workload Manager**: Installed on head node, configured across compute nodes
- 🚀 **GPU Passthrough**: Using `vfio-pci`, OVMF UEFI, and `virtio` interfaces
- 🔗 **Bridged Networking**: VMs are accessible from LAN with optional DNS names
- 📦 **Ansible-ready**: Cluster is automatable via Ansible for scaling or redeployment
- 📈 **Monitoring ready**: Planned support for `htop`, `nmon`, `ganglia`, or `Prometheus`

---

## 📡 Networking Strategy

- Host has bridged connection (`br0`) attached to physical NIC
- All VMs receive static IPs or DHCP reservations
- SSH enabled on all VMs
- Head node may act as NFS and DNS server for cluster

---

## 🧪 Learning Objectives

- Simulate a real HPC cluster using SLURM and KVM
- Practice GPU passthrough and PCI device management
- Understand disk provisioning, thin LVM pools, and file system design
- Configure automated user, job, and system management
- Implement secure networking and monitoring best practices

---

## 🚧 To-Do / Future Work

- [ ] Automate VM setup via `virt-clone` and cloud-init
- [ ] Set up `/home` and `/scratch` as NFS shares
- [ ] Deploy SLURM accounting and job history
- [ ] Add cluster monitoring (Grafana, Ganglia, etc.)
- [ ] Add backup strategy (rsnapshot, duplicity, etc.)
- [ ] Build sample SLURM job scripts (MPI, CUDA, etc.)

---

