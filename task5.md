## Task-5

## 1.Nginx installation

```bash

```

## 2. non priviledged user setup

```bash
sudo useradd -r -s /bin/false webapps
sudo mkdir /home/webapps
sudo chown webapps:webapps /home/webapps
sudo usermod -d /home/webapps webapps
```

## 3.app1 setup
downladed app1 
```bash
cd /opt
sudo wget https://do.edvinbasil.com/ssl/app -O app1
sudo wget https://do.edvinbasil.com/ssl/app.sha256.sig -O app1.sha256.sig
```

verifying sha256
```bash
sha256sum app1
cat app1.sha256.sig
```

add execute permission for webapp
```bash
sudo chmod +x /opt/app1
sudo chown webapps:webapps /opt/app1
#run as webapp user
sudo -u webapps /opt/app1 &
```

## 4.app2 setup

```bash
cd /opt
sudo git clone https://gitlab.com/tellmeY/issslopen
sudo chown -R webapps:webapps /opt/issslopen
```

install and execute bun runtime
```bash
sudo -i bash -c 'curl -fsSL https://bun.sh/install | bash'
# movinf to local/bin as it is a global path
sudo cp /root/.bun/bin/bun /usr/local/bin/bun
sudo chmod 755 /usr/local/bin/bun
```

install dependencies
```bash
cd /opt/issslopen
sudo -u webapps npm install
```
create token
```bash
sudo nano /opt/issslopen/.env
```
and add  ADMINS=sslAdmin:supersecrettoken123

run it 
```bash
sudo -u webapps /usr/local/bin/bun index.ts &
```

install and generate ssl certificate
```bash 
sudo apt install certbot python3-certbot-nginx -y
sudo certbot certonly --nginx -d blimnock.sslnitc.site
```

modify nginx config

```bash
server {
    listen 80;
    server_name blimnock.sslnitc.site;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name blimnock.sslnitc.site;

    ssl_certificate /etc/letsencrypt/live/blimnock.sslnitc.site/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/blimnock.sslnitc.site/privkey.pem;

    # Security headers
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self';" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location /server1/ {
        proxy_pass http://127.0.0.1:8008/server1/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /server2/ {
        proxy_pass http://127.0.0.1:3000/sslopen;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /sslopen {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

incase of any edit -> https://blimnock.sslnitc.site/sslopen/edit/supersecrettoken123

used securityheaders.com for csp