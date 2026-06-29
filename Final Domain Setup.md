# 🌐 Domain Setup with Nginx (Production Guide)

This document provides a complete, battle-tested guide to configuring a domain with **Nginx reverse proxy + HTTPS (SSL)** on a Linux server.

---

# 🔐 Final HTTPS Config

```
sudo nano /etc/nginx/sites-available/domain.name
```

```nginx
server {
    listen 80;
    server_name domain.com;

    location / {
        proxy_pass http://127.0.0.1:port;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 300;
        proxy_connect_timeout 300;
    }
}
```

```bash
sudo certbot --nginx -d domain.com
```

---

👉 Nginx acts as the gateway — your app stays internal, secure, and flexible.

---
