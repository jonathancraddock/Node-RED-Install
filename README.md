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
sudo ufw enable
sudo ufw status verbose
```

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

