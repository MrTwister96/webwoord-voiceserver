```bash
sudo apt install docker.io nginx



sudo mkdir /opt/livekit-server

sudo cp config.yaml /opt/livekit-server



export LIVEKIT_VERSION=v0.13

sudo curl -o /etc/systemd/system/docker.livekit-server@.service -O https://raw.githubusercontent.com/livekit/livekit-server/master/deploy/docker.livekit-server%40.service

sudo systemctl enable docker.livekit-server@${LIVEKIT_VERSION}

sudo systemctl start docker.livekit-server@${LIVEKIT_VERSION}



sudo cp livekit.conf /etc/nginx/sites-available/livekit.conf

sudo ln -s /etc/nginx/sites-available/livekit.conf /etc/nginx/sites-enabled/livekit.conf

sudo systemctl restart nginx



sudo ufw allow 'Nginx Full'

sudo ufw allow 10000:20000/tcp comment 'Livekit RTC Ports'

sudo ufw allow 10000:20000/udp comment 'Livekit RTC Ports'

sudo ufw allow 7881/tcp comment 'Livekit RTC Ports'

sudo ufw allow 7882/udp comment 'Livekit RTC Ports'

sudo ufw reload



sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx -d banshee.ptype.app



crontab -e

0 5 \* \* \* /usr/bin/certbot renew --quiet
```
