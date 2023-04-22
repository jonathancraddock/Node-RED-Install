# Node-RED-Install
Notes on Node-RED install on an Ubuntu 22 droplet hosted by DigitalOcean.

## Actions Log

Create droplet, receive credentials via email
Use recovery console to reset root password

### Via Recovery Console

Create a new user:
```bash
sudo adduser <username>
sudo usermod -aG sudo <username>
```

Confirm SSH is enabled:
```bash
systemctl status sshd.service
```

Update SSH config:
```bash
cd /etc/ssh
sudo nano sshd_config
```

Added/Modified the lines:
```bash
AllowGroups sudo
PasswordAuthentication yes
ChallengeResponseAuthentication yes
```
An alternative would have been, `AllowUsers <username>`, with additional names separated by a <kbd>Space</kbd> if required.

> **Note**
> Check the subfolder `sshd_config.d` because the default `sshd_config` may be inheriting permissions from here. In this example, there was a file named `50-cloud-init.conf` with the config `PasswordAuthentication no`. Comment that line if required.

Restart the SSH service:
```bash
sudo systemctl restart sshd.service
```

### Via SSH

Test the new user via SSH, and then disable root logins:
```bash
cd /etc/ssh
sudo nano sshd_config
```

Modify the line:
```bash
PermitRootLogin no
```

Restart the SSH service:
```bash
sudo systemctl restart sshd.service
```

-----

Because password logins are allowed, update the system and then install **Fail2Ban**.

Update:
```bash
sudo apt update
sudo apt upgrade
sudo reboot now
```

Install Fail2Ban:
```bash
sudo apt install fail2ban
```

Create a local jail for blocking SSH abuse:
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Add/Modify the lines:
```bash
[sshd]
mode = normal
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
banaction = ufw
```

Check the status of UFW. (Probably disabled at this stage.)
```
sudo ufw status verbose
```

Ensure that OpenSSH is allowed, and re-check the status of UFW:
```bash
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw allow from <CIDR Notation> to any port 22
sudo ufw enable
sudo ufw status verbose
```
^- *Take care if allowing only from a specifed IP range, to avoid accidental lockout!*

> **Note**
> The syntax `sudo ufw allow from 82.69.0.0/17 to any port 22` limits SSH connections to only the range provided by Zen Internet, for example.

With UFW enabled, enable Fail2Ban:
```bash
sudo systemctl restart fail2ban.service
```

You can check the status of Fail2Ban, and specifically the SSH jail, using the following:
```bash
sudo systemctl status fail2ban.service
sudo fail2ban-client status sshd
```

-----

> **Note**
> Taking a snapshot here.

-----

### Installing Apache

Update and install:
```bash
sudo apt update
sudo apt install apache2
```

Check and update UFW rules:
```bash
sudo ufw app list
sudo ufw allow in "Apache Full"
sudo ufw status verbose
```

-----

### Installing PHP

Install and confirm version:
```bash
sudo apt install php libapache2-mod-php php-mysql
php -v
```

Create a PHP file to test install, eg `/var/www/html/index.php`:
```php
<?php
phpinfo();
```

-----

### Generic Landing

Empty the `/var/www/html/` folder and create a generic landing page:
```bash
cd /var/www/html/
sudo rm *
sudo nano index.html
```

Generic landing page:
```html
<!DOCTYPE html>
<html>
<head>
	<title>Hello, World!</title>
</head>
<body>
	<p>Hello, World!</p>
</body>
</html>
```

-----

> **Note**
> Taking a snapshot here.

-----

### Installing NodeJS and NodeRED

Node-RED recommends v16:
```bash
cd ~
curl -sL https://deb.nodesource.com/setup_15.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt install nodejs
node -v
npm -v
```

Use NPM to install NodeRED:
```bash
sudo npm install -g --unsafe-perm node-red node-red-admin
```

You can run / check version with:
```bash
node-red -v
```
<kbd>Ctrl</kbd> + <kbd>C</kbd> to quit.

To ensure NodeRED runs on startup:
```bash
sudo nano /etc/systemd/system/node-red.service
```

Create the following config:
```bash
[Unit]
Description=Node-RED
After=syslog.target network.target

[Service]
ExecStart=/usr/local/bin/node-red-pi --max-old-space-size=128 -v
Restart=on-failure
KillSignal=SIGINT

# log output to syslog as 'node-red'
SyslogIdentifier=node-red
StandardOutput=syslog

# non-root user to run as
WorkingDirectory=/home/<username>/
User=<username>
Group=<username>

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl enable node-red
```

Starting and stopping, if required:
```bash
sudo systemctl start node-red
sudo systemctl stop node-red
```

Generate a password hash:
```bash
node-red-admin hash-pw
```

Copy the hash, then edit the settings:
```bash
nano ~/.node-red/settings.js
```

Find the `adminAuth` section and uncomment.
```javascript
    /** To password protect the Node-RED editor and admin API, the following
     * property can be used. See http://nodered.org/docs/security.html for details.
     */
    adminAuth: {
        type: "credentials",
        users: [{
            username: "<username>",
            password: "<hash>",
            permissions: "*"
        }]
    },
```
