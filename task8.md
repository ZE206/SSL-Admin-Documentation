## task 8

### docker installation and user add
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

### create folder

i created 'portfolio' folder. In it make index.html file having portfolio of mine in website subfolder.

Create a Dockerfile and 
```bash
FROM nginx:alpine
COPY ./website /usr/share/nginx/html
EXPOSE 80
```

### build and run docker

``` bash
cd /home/emirz/portfolio

# Build an image
sudo docker build -t portfolio:latest .

# Run the container
sudo docker run -d \
  --name portfolio \
  -p 8081:80 \
  -v /home/emirz/portfolio/website:/usr/share/nginx/html \
  portfolio:latest
```

### create systemd service

create service file - /etc/systemd/system/portfolio.service

```bash
sudo nano /etc/systemd/system/portfolio.service

```

```bash
[Unit]
Description=Portfolio Website Container
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a portfolio
ExecStop=/usr/bin/docker stop portfolio

[Install]
WantedBy=multi-user.target
```

reload daemon and start the service we created.
```bash
sudo systemctl daemon-reload
sudo systemctl enable portfolio.service
sudo systemctl start portfolio.service
```

### Nginx reverse proxy for portofolio

```bash
sudo nano /etc/nginx/sites-available/ssl-apps
```

```bash
location /portfolio/ {
    proxy_pass http://127.0.0.1:8081/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
```

### done

access portfolio at https://blimnock.sslnitc.site/portfolio/



