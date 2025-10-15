# 07 - NGINX Edge Proxy

## Objective
Set up an external NGINX edge proxy in front of Traefik, with wildcard SSL certificates managed by Certbot via Cloudflare DNS challenge.

## Steps

### 1. Install dependencies
```bash
apt update
apt install -y nginx certbot python3-certbot-dns-cloudflare
```

### 2. Create Cloudflare credentials file
File: `/root/.secrets/cloudflare.ini`
```
dns_cloudflare_api_token = <CLOUDFLARE_API_TOKEN>
```
Ensure restricted permissions:
```
chmod 600 /root/.secrets/cloudflare.ini
```


### 3. Obtain wildcard certificate
```
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d digitalepalika.com \
  -d "*.digitalepalika.com" \
  --agree-tos \
  -m admin@digitalepalika.com \
  --non-interactive
```

Certificate path will be:
```
/etc/letsencrypt/live/digitalepalika.com/
```

### 4. Create NGINX configuration
File: `/etc/nginx/conf.d/k8s.conf`

```
server {
    listen 80;
    server_name *.digitalepalika.com;
    location / {
        proxy_pass http://<CONTROL_PLANE_IP>:30880;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 443 ssl;
    server_name *.digitalepalika.com;

    ssl_certificate /etc/letsencrypt/live/digitalepalika.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/digitalepalika.com/privkey.pem;

    location / {
        proxy_pass https://<CONTROL_PLANE_IP>:30443;
        proxy_ssl_verify off;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Replace `<CONTROL_PLANE_IP>` with your cluster’s Traefik node public IP.

### 5. Test and reload NGINX
```
nginx -t
systemctl reload nginx
```

### 6. Verify proxy routing
```
curl -vk https://whoami.digitalepalika.com
```

Expected:
```
Hostname: whoami-xxxxxx
```

### 7. Set up auto-renewal
Test renewal:
```
certbot renew --dry-run
```

Enable cron job:
```
crontab -e
```

Add:
```
0 3 * * * certbot renew --quiet && systemctl reload nginx
```

### 8. Harden the edge (optional)
```
apt install -y ufw fail2ban
ufw allow 22/tcp
ufw allow 80,443/tcp
ufw --force enable
```

