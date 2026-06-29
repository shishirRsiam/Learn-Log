# 🌐 Domain Setup with Nginx (Production Guide)

This document provides a complete, battle-tested guide to configuring a domain with **Nginx reverse proxy + HTTPS (SSL)** on a Linux server.

---

# 📌 Overview

We will achieve:

* Map a domain → server IP (DNS)
* Route traffic via **Nginx**
* Reverse proxy to internal app (e.g., `127.0.0.1:8072`)
* Enable **HTTPS using Let's Encrypt (Certbot)**
* Configure HTTP → HTTPS redirect

---

# 🧱 Architecture

```
User → https://yourdomain.com → Nginx (443)
                                   ↓
                          Reverse Proxy
                                   ↓
                        http://127.0.0.1:PORT
```

---

# ⚙️ Prerequisites

* Ubuntu/Debian server
* Domain name (e.g., `yourdomain.com`)
* Root or sudo access
* Running backend service (e.g., Django, Node, etc.)

---

# 🌍 Step 1 — Configure DNS

Go to your domain provider and add:

```
Type: A
Host: @
Value: YOUR_SERVER_IP

Type: A
Host: www
Value: YOUR_SERVER_IP
```

For subdomain:

```
Type: A
Host: api
Value: YOUR_SERVER_IP
```

---

# 🧰 Step 2 — Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

Start & enable:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

# 📁 Step 3 — Create Nginx Config

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

### Basic Reverse Proxy (HTTP only)

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8072/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

# 🔗 Step 4 — Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

Test config:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

---

# 🧪 Step 5 — Test HTTP

```bash
curl -I http://yourdomain.com
```

Expected:

```
HTTP/1.1 200 OK
```

👉 If this fails, fix BEFORE adding SSL.

---

# 🔒 Step 6 — Install Certbot (SSL)

## Recommended (Snap version)

```bash
sudo apt remove certbot -y
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

---

# 🔐 Step 7 — Enable HTTPS

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

For subdomain:

```bash
sudo certbot --nginx -d api.yourdomain.com
```

---

# 🔁 Step 8 — Auto Redirect HTTP → HTTPS

Certbot usually adds this automatically:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}
```

---

# 🔐 Step 9 — Final HTTPS Config

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

# 🔄 Step 10 — Auto Renewal

Test renewal:

```bash
sudo certbot renew --dry-run
```

👉 Certbot auto-renews via cron/systemd.

---

# 🛡️ Optional Security Headers

Add inside `server {}`:

```nginx
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

---

# 🚨 Common Issues & Fixes

## 1. Nginx fails to start

```bash
sudo nginx -t
```

👉 Fix syntax errors.

---

## 2. Domain not working

* Check DNS propagation
* Verify IP mapping

---

## 3. SSL errors

* Ensure domain resolves correctly
* Remove broken manual certificates
* Use Certbot instead

---

## 4. Backend not responding

Check:

```bash
curl http://127.0.0.1:8072
```

---

## 5. Mixed Content Error

👉 Happens when:

* frontend uses HTTPS
* API uses HTTP

Fix:

* serve API over HTTPS (this setup)

---

# ⚡ Best Practices

* Never manually manage SSL unless required
* Always test HTTP before HTTPS
* Keep backend isolated (localhost)
* Use subdomains for APIs:

  * `app.domain.com`
  * `api.domain.com`

---

# 🧠 Production Architecture Example

```
Frontend (Vercel)
        ↓
https://yourdomain.com

Backend (Server)
https://api.yourdomain.com
        ↓
Nginx → Django/Node (localhost)
```

---

# ✅ Final Checklist

* [ ] DNS configured
* [ ] Nginx installed
* [ ] Reverse proxy working (HTTP)
* [ ] Certbot SSL installed
* [ ] HTTPS working
* [ ] Auto-renew enabled

---

# 🎯 Conclusion

A correct setup ensures:

* Secure HTTPS communication
* Clean domain routing
* Scalable backend architecture

👉 Nginx acts as the gateway — your app stays internal, secure, and flexible.

---
