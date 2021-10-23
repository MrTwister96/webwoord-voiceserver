# WebWoord Voice Server

Voice server deployment with [Livekit](https://livekit.io/)

# Production Deployment

This deployment guide was tested on a Ubuntu 20.04 Kernel-Based VM.

## Preperation

#### Apt update and upgrade

```bash
sudo apt update
sudo apt upgrade
```

#### Install required packages

```bash
sudo apt install -y docker.io nginx certbot python3-certbot-nginx
```

#### Clone repo

```bash
sudo git clone https://github.com/MrTwister96/webwoord-voiceserver.git /opt/livekit-server
```

## Livekit Application

#### Edit `config.yaml` to update the API key and secret

```bash
sudo nano /opt/livekit-server/config.yaml
```

```bash
log_level:  info
port:  7880
rtc:
    port_range_start:  10000
    port_range_end:  20000
    use_external_ip:  true
    tcp_port:  7881
    udp_port:  7882
keys:
    YOUR-LIVEKITAPIKEY-HERE:  YOUR-LIVEKITAPISECRET-HERE
```

#### Setup and run docker container

```bash
export LIVEKIT_VERSION=v0.13
sudo curl -o /etc/systemd/system/docker.livekit-server@.service -O https://raw.githubusercontent.com/livekit/livekit-server/master/deploy/docker.livekit-server%40.service
sudo systemctl enable docker.livekit-server@${LIVEKIT_VERSION}
sudo systemctl start docker.livekit-server@${LIVEKIT_VERSION}
```

## NGINX SSL Reverse Proxy Setup

#### Edit `livekit.conf` to update the server_name option to match your DNS entry

```bash
sudo nano /opt/livekit-server/livekit.conf
```

```bash
server {
    server_name <your-livekit-server.domain.com>;
    listen 80 http2;
    listen [::]:80 http2;
    location / {
        proxy_pass http://127.0.0.1:7880;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
}
```

#### NGINX Configuration

```bash
sudo cp /opt/livekit-server/livekit.conf /etc/nginx/sites-available/livekit.conf
sudo ln -s /etc/nginx/sites-available/livekit.conf /etc/nginx/sites-enabled/livekit.conf
sudo systemctl restart nginx
```

#### Configure UFW (If Applicable)

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow 10000:20000/tcp comment 'Livekit RTC Ports'
sudo ufw allow 10000:20000/udp comment 'Livekit RTC Ports'
sudo ufw allow 7881/tcp comment 'Livekit RTC Ports'
sudo ufw allow 7882/udp comment 'Livekit RTC Ports'
sudo ufw reload
```

#### Generate Let's Encrypt SSL Certificate

This will ask you some questions after which the certificate will be generated and verified. Make sure that your DNS A record is pointing to your server.

```bash
sudo certbot --nginx -d <your-livekit-server.domain.com>
```

#### Setup automatic certificate renewal cron job

```bash
crontab -e

0 5 \* \* \* /usr/bin/certbot renew --quiet
```
