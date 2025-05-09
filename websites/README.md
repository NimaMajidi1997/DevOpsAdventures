# Build images
Build images with:
```bash
docker build -t nimaianp75/portfolio -f portfolio-dockerfile .
docker build -t nimaianp75/market -f market-dockerfile .
```

You can run the containers inside the server with:
```bash
# nimadevops.de → port 8000
docker run -d -p 8000:80 --name nimadevops-site nimaianp75/nimadevops
# market.nimadevops.de → port 8080
docker run -d -p 8080:80 --name market-site nimaianp75/market
```
# reverse proxy config
Now let's create reverse proxy config for nimadevops.de:
```bash
sudo nano /etc/nginx/sites-available/nimadevops.de
```
with this content:
```bash
server {
    listen 80;
    server_name nimadevops.de www.nimadevops.de;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
and aldo config for market.nimadevops.de:
```bash
sudo nano /etc/nginx/sites-available/market.nimadevops.de
```
with this content:
```bash
server {
    listen 80;
    server_name market.nimadevops.de;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Now you can enable sites inside the server with:
```bash
sudo ln -s /etc/nginx/sites-available/nimadevops.de /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/market.nimadevops.de /etc/nginx/sites-enabled/
```
Also remove any old/conflicting config:
```bash
sudo rm /etc/nginx/sites-enabled/default
```
And finally Test & Reload NGINX:
```bash
sudo nginx -t && sudo systemctl reload nginx
sudo systemctl status nginx

```
## Secure domains with HTTPS
secure both domains with HTTPS using Certbot.
Install Certbot
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```
Run Certbot to issue certificates for both domains
```bash
 sudo certbot --nginx -d nimadevops.de -d www.nimadevops.de -d market.nimadevops.de
```
You can test renewal with:
```bash
 sudo certbot renew --dry-run

```