# Node-RED-Install
Notes on Node-RED install on an Ubuntu 22 droplet hosted by DigitalOcean.

## Actions Log

Create droplet, receive credentials via email
Use recovery console to reset root password

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

