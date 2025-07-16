# scaleway-debian-12-cheatsheet

## SSH / PuTTY

https://www.scaleway.com/en/docs/create-and-connect-to-your-server/

## Update

```bash
sudo apt update
sudo apt upgrade
```

### An error occured during upgrade : 

```
grub-install: warning: File system `ext2' doesn't support embedding.
grub-install: warning: Embedding is not possible.  GRUB can only be installed in this setup by using blocklists.  However, blocklists are UNRELIABLE and their use is discouraged..
grub-install: error: will not proceed with blocklists.
dpkg: error processing package grub-cloud-amd64 (--configure):
 installed grub-cloud-amd64 package post-installation script subprocess returned error exit status 1
Processing triggers for libc-bin (2.36-9+deb12u10) ...
Processing triggers for initramfs-tools (0.142+deb12u3) ...
update-initramfs: Generating /boot/initrd.img-6.1.0-37-amd64
Processing triggers for ca-certificates (20230311+deb12u1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Errors were encountered while processing:
 grub-cloud-amd64
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

Restarted apt upgrade => it worked

## byobu

```bash
sudo apt install byobu
byobu

# Launch auto at login
byobu-enable
```

## New user

```bash
adduser spout
usermod -g www-data spout
```

## sudo

```bash
adduser spout sudo
```

## SSH

```bash
sudo nano /etc/ssh/sshd_config
#Port 22
Port 7022

sudo service ssh restart
```

## Firewall

https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server

```bash
sudo apt install ufw

sudo nano /etc/default/ufw
# Already set in Debian 12
IPV6=yes

sudo ufw disable
sudo ufw enable

sudo ufw default deny incoming
sudo ufw default allow outgoing

# ufw allow ssh
sudo ufw allow 7022/tcp
sudo ufw allow http
sudo ufw allow https

sudo ufw show added
sudo ufw enable
sudo ufw status
```

## fail2ban

```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.conf

destemail = votremail@domain.com
action = %(action_)s

# action_ => simple ban
# action_mw => ban et envoi de mail
# action_mwl => ban, envoi de mail accompagné des logs

sudo service fail2ban restart
```

## Mail

```bash
sudo apt install exim4-config
# Was already installed
sudo dpkg-reconfigure exim4-config
```

1. internet site; mail is sent and received directly using SMTP
2. System mail name: ENTER
3. IP-addresses: ENTER
4. Other destinations: ENTER
5. Domains to relay mail for: ENTER
6. Machines to relay mail for: ENTER
7. Keep number of DNS-queries minimal: NO
8. Delivery method: mbox format
9. Split configuration into small files: NO
10. Root and postmaster mail recipient: ENTER

## DEB.SURY.ORG

https://deb.sury.org/

```bash
curl -sSL https://packages.sury.org/php/README.txt | sudo bash -x
sudo apt update
```

## PHP

```bash
sudo apt install php7.4-fpm php7.4-gd php7.4-mysql php7.4-pgsql php7.4-sqlite3 php7.4-mbstring php7.4-xml php7.4-intl php7.4-curl php7.4-zip php7.4-soap php7.4-redis
```

## MariaDB

https://www.geek17.com/fr/content/debian-9-stretch-installer-et-configurer-mariadb-65
https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-debian-9

```bash
sudo apt install mariadb-server
sudo mysql_secure_installation
```

### Test root password defined

```bash
sudo mysql -u root -p
```

## nginx

```bash
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default

root /var/www;
index index.php index.html index.htm index.nginx-debian.html;

# Uncomment location ~\.php$ {
# Uncomment include snippets/fastcgi-php.conf;
# Uncomment fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;

sudo service nginx reload

# www-data
sudo chown www-data:www-data /var/www
sudo chmod g+w /var/www

# Gzip
sudo nano /etc/nginx/nginx.conf
# Uncomment:
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

sudo nano /etc/nginx/nginx.conf
# Uncomment:
server_tokens off;

sudo service nginx reload
```

## Locales

```bash
sudo dpkg-reconfigure locales

[*] fr_FR.UTF-8

locale -a
```

## Gettext

```bash
sudo apt install gettext
```

## Redis

```bash
sudo apt install redis-server
```

## ClamAV

```bash
sudo apt install clamav clamav-freshclam
```

## Adminer

```bash
sudo nano /usr/bin/adminer-update
```

```
#!/bin/bash
wget -O /var/www/adminer.php https://www.adminer.org/latest.php
```

```bash
sudo chmod +x /usr/bin/adminer-update
sudo adminer-update
```

## CURL

```bash
sudo apt install curl
```

## Git

```bash
sudo apt install git
```

## ZIP

```bash
sudo apt install zip unzip
```

## htop

```bash
sudo apt install htop
```

## ncdu

```bash
sudo apt install ncdu
```

## Backup

### Rclone

https://rclone.org/
https://www.scaleway.com/en/docs/how-to-migrate-object-storage-buckets-with-rclone/

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

```bash
rclone config
rclone config show
```

```bash
sudo wget https://raw.githubusercontent.com/spout/debian-10-install-cheatsheet/master/backup.php -O /opt/backup.php
sudo nano /opt/backup.php
sudo chmod +x /opt/backup.php
sudo ln -s /opt/backup.php /etc/cron.daily/backup
sudo run-parts --test /etc/cron.daily

# Test
/opt/backups.php
```

## Supervisor

```bash
sudo apt install supervisor
```

## HTTPS / Let's encrypt

https://certbot.eff.org/instructions?ws=other&os=snap

```bash
sudo apt install snapd

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

```bash
nano /etc/nginx/sites-available/example.com

location ~ /.well-known {
    allow all;
    root /var/www;
}

sudo nginx -t
sudo service nginx reload

sudo certbot certonly --webroot -w /var/www/ -d example.com -d www.example.com --rsa-key-size 4096
# Wildcard
certbot certonly --manual --preferred-challenges dns --register -d example.com -d *.example.com

sudo certbot renew --dry-run

sudo crontab -e
0 */12 * * * certbot renew --quiet --post-hook "service nginx reload"

sudo openssl dhparam -out /etc/ssl/private/dhparams.pem 4096

sudo nano /etc/nginx/nginx.conf

##
# SSL Settings
##

#ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
#ssl_prefer_server_ciphers on;

ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_session_cache shared:SSL:10m;
ssl_dhparam /etc/ssl/private/dhparams.pem;
add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
```
