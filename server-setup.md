# Home Server Setup
Instructions are for setting up my home server on an AWOW AL34 & AK41 Mini PC with Debian 11 Bullseye. The following steps assume the Debian install is already complete.

## Administrative
### Install and setup sudo:
1. `apt install sudo`
2. `/sbin/usermod -aG sudo <username>`

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
2. `sudo nano /etc/fstab` and add a line with the UUID, mount point, file system type, options, dump and pass options: `UUID=<uuid> /home/user/backup ext4    defaults        0       0`

### Setup email SMTP for notifications:
1. `sudo apt install exim4-daemon-light`
2. `dpkg-reconfigure exim4-config` and setup using the following:

````
    General type of mail configuration:	mail sent by smarthost; received via SMTP or fetchmail
    System mail name:	localhost
    IP addresses to listen on for incoming SMTP connections:	127.0.0.1 (leave default)
    Other destinations for which mail is accepted:	Leave empty.
    Machines to relay mail for:	Leave empty.
    IP address or host name of the outgoing smarthost:	smtp.gmail.com::587
    Hide local mail name in outgoing mail?:	No
    Keep number of DNS-queries minimal (Dial-on-Demand)?:	No
    Delivery method for local mail:	mbox format in /var/mail/
    Split configuration into small files?:	No
````

3. `sudo nano /etc/exim4/passwd.client` and add `smtp.gmail.com:username@gmail.com:mypassword`
4. Send test email using `echo test only | mail -s 'Test Subject' user@gmail.com`
5. Logs are located in `/var/log/exim4/mainlog`

### Enable unattended upgrades:
1. `sudo dpkg-reconfigure -plow unattended-upgrades` to enable unattended upgrades
2. `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and uncomment `Unattended-Upgrade::Automatic-Reboot "true";`, `Unattended-Upgrade::Automatic-Reboot-Time "03:00";` and `Unattended-Upgrade::Mail "user@email.com";`

### Install Docker & Docker Compose:
1. https://docs.docker.com/engine/install/debian/
2. https://docs.docker.com/compose/install/

### Setup EdgeTPU:
1. https://www.coral.ai/docs/m2/get-started/#2a-on-linux

On step 3, only `sudo groupadd apex` is needed to supress an log warning

### Setup DDNS updates:
1. Check instructions at duckdns.org (login for specific instructions)

### Setup Cockpit Project:
1. https://cockpit-project.org/

## Restore Docker containers:
### Restore volumes
1. `docker create volume <volume name>` for every volume needed
2. From the backup drive, `cp -r volumes /var/lib/docker/`

### Start the containers
1. `docker-compose up -d`

## Setup Network UPS
### Install & setup NUT server:
1. `sudo apt install nut-server`
2. `sudo nano /lib/systemd/system/nut-server.service` and add (fixes issue after reboot where nut-server will fail to start):

````
[Service]
ExecStartPre=/bin/sleep 30
````
3. From the NUT folder on the back up drive, `cp -r * /etc/nut`
4. `sudo systemctl reboot nut-server`

### Install & setup NUT clients (on second server):
1. `sudo apt install nut-client`
2. Restore NUT client config files

## Backups
### Install rsync:
1. `sudo apt install rsync`

### Setup cronjobs:
1. `sudo crontab -e -u root`
2. Add cronjobs, ex: `30 1 * * * rsync -a /home/user/docker/docker-compose.yml /home/user/backup/home-server-1/docker` (occurs at 1:30 am every day)

## Maintenence
### Update docker containers
### Update containers:
1. `docker-compose pull`
2. `docker-compose up -d`
