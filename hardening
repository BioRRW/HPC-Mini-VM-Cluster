## First, I want to ensure future users will have a clean and usable shell setup:

Customize `/etc/skel/`

I will start by adding a `.bashrc`,  a `.bash_profile` and an empty `.ssh/authorized_keys`

I kept all of the default contents and just added a bit for the prompt in the `.bashrc`: 
```
# Prompt
PS1='\u@\h:\w\$ '
```

`.bash_profile` was left alone.

Created an empty `.ssh/authorized_keys` and set permissions so that only the user can read and write:
```
mkdir -p /etc/skel/.ssh
chmod 700 /etc/skel/.ssh
touch /etc/skel/.ssh/authorized_keys
chmod 600 /etc/skel/.ssh/authorized_keys
```

Lastly, I made sure the permissions were set so the user can write to their skel files and everyone can read them (not including the .ssh/authorized_keys)
`chmod 644 /etc/skel/.bashrc /etc/skel/.bashrc_profile /etc/skel/.bashrc_logout`

## Disable root ssh login

Modify `/etc/ssh/sshd_config` and set `PermitRootLogin no`:

Restart SSH:
`sudo systemctl restart sshd`

Restrict SSH to specific users:

## Generate an SSH key as a test for future login for admin account

Generate an SSH key for `admin`:
`ssh-keygen -t ed25519 -C "admin@hpc-head"`

Add the key to my public key to authorized keys
```
mkdir -p ~/.ssh
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## Install and Configure Firewall
```
sudo dnf install firewalld -y
sudo systemctl enable --now firewalld
```

Then, do a basic setup:
```
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --permanent --add-service=libvirt
sudo firewall-cmd --permanent --add-port=6817-6818/tcp   # SLURM default ports
sudo firewall-cmd --permanent --add-port=6817-6818/udp
sudo firewall-cmd --reload
```

## Enable Automatic Security Updates

Install `dnf-automatic`
```
sudo dnf install dnf-automatic -y
sudo systemctl enable --now dnf-automatic.timer
```

Lastly, at least for now, checked that SELinux is in enforcing mode:
`getenforce`
