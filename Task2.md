## Task 2

### Root login disable
SSH daemon config is found at /etc/ssh/sshd_config. This files defines how server SSH server authenticates and accepts connections.

```bash
sudo systemctl restart ssh
```

allowing direct root login is bad because hackers only need to guess

### disable password based authentication
In ssh daemon config, set 'PasswordAuthentication no'. Prevents password based attack. SSH authentication is far better than password authentication.

### changing port

In ssh daemon config, set the port to 'PORT 2222'. Port 22 is heavily targeted by the malicious things in internet. THis reduces log noises and unnecessary login attempts

### configuring SSH

Created a key pair using "ssh-keygen". This created public key and private key. And then copied public  key to server using

```bash
ssh-copy-id -p 2222 user@server-ip
```

### fail2ban

architecture: log file→ fail2ban filter (regex detection) → jail (policy) → firewall rule (ban IP)

```bash
sudo apt update
sudo apt install fail2ban

sudo systemctl enable fail2ban
sudo systemctl start fail2ban

sudo systemctl status fail2ban

sudo cp jail.conf jail.local
sudo nano jail.local
```

parameters set in sshd header in jail.local:
    maxretry (attempts before ban)
    findtime (time window)
    bantime (how long ban lasts)`


to see realtime fail2ban activity
```bash
sudo tail -f /var/log/fail2ban.log
```