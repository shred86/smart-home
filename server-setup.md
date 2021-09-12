# Home Server Setup
Instructions are for setting up my home server on an AWOW AL34 Mini PC with Debian 11 Bullseye. The following steps assume the Debian install is already complete.

## Administrative
### Install and setup sudo:
1. `apt-get install sudo`
2. `/bin/usermod -aG sudo <username>`

### Install missing drivers:
1. `sudo nano /etc/apt/sources.list` and add `non-free` after `main` to `deb http://deb.debian.org/debian/ bullseye main` and `deb-src http://deb.debian.org/debian/ bullseye main`
2. `sudo apt update`
3. `sudo apt install firmware-realtek`
4. `apt install firmware-misc-nonfree`

### Add SSH keys for root and user account:
1. `sudo nano /etc/ssh/sshd_config`
2. Uncomment and set `PermitRootLogin yes`
3. `sudo systemctl restart ssh`

#### From the client computer:
1. `ssh-copy-id root@<server ip>`
2. `ssh-copy-id <username>@<server ip>`

Go back and comment out `PermitRootLogin` in `/etc/ssh/sshd_config`

### Mount additional hard drives:
1. `lsblk -f` to view current mounted drives and partitions
2. `sudo blkid /dev/sdb1` to get UUID of partition
3. `sudo nano /etc/fstab` and add a line with the UUID, mount point, file system type, options, dump and pass options: `UUID=<uuid> /home/user/backup ext4    defaults        0       0`

### Setup email SMTP for notifications:
1. `sudo apt install exim4-daemon-light`
2. `dpkg-reconfigure exim4-config` and follow instructions - use `smtp.gmail.com::465`
3. `sudo nano /etc/exim4/passwd.client` and add `smtp.gmail.com:username@gmail.com:mypassword`

### Enable unattended upgrades:
1. `sudo dpkg-reconfigure -plow unattended-upgrades` to enable unattended upgrades
2. `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and uncomment `Unattended-Upgrade::Automatic-Reboot "true";`, `Unattended-Upgrade::Automatic-Reboot-Time "03:00";` and `Unattended-Upgrade::Mail "user@email.com";`

### Install Docker:
https://docs.docker.com/engine/install/debian/

### Setup EdgeTPU:
https://www.coral.ai/docs/m2/get-started/#2a-on-linux

On step 3, only `sudo groupadd apex` is needed to supress an log warning



Still need to add my docker setup...
