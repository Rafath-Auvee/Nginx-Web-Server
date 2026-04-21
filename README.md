# Nginx Web Server with HTTPS, SSL & Reverse Proxy

## Overview

A secure production-like web server setup using Nginx on Linux with:
- Static website hosting
- HTTPS via self-signed SSL (OpenSSL)
- HTTP → HTTPS auto redirect
- Reverse proxy to backend on port 3000

---

## Setup Commands

### 1. Install Nginx & OpenSSL

```bash
sudo apt update && sudo apt install nginx openssl -y
```

### 2. Create Web Root

```bash
sudo mkdir -p /var/www/secure-app
sudo chown -R www-data:www-data /var/www/secure-app
```

Copy the HTML file:

```bash
sudo cp html/index.html /var/www/secure-app/index.html
```

### 3. Generate Self-Signed SSL Certificate

```bash
sudo mkdir -p /etc/nginx/ssl

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
```

### 4. Apply Nginx Configuration

```bash
sudo cp conf/secure-app.conf /etc/nginx/sites-available/secure-app
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/secure-app /etc/nginx/sites-enabled/
```

### 5. Test & Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 6. Run Backend on Port 3000

```bash
python3 -m http.server 3000
```

---

## Nginx Configuration

```nginx
server {
    listen 80;
    server_name _;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name _;

    ssl_certificate     /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    root /var/www/secure-app;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## SSL Command

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
```

---

## Verification

### Test HTTP → HTTPS Redirect

```bash
curl -I http://localhost
```

Expected: `301 Moved Permanently` with `Location: https://...`

### Test HTTPS

```bash
curl -k https://localhost
```

Expected: HTML page content returned.

### Test Backend via Reverse Proxy

```bash
curl -k https://localhost/api/
```

Expected: Response from the backend running on port 3000.

---

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/https_working.png` | HTTPS serving the static page |
| `screenshots/redirect_working.png` | HTTP 301 redirect to HTTPS |
| `screenshots/backend_proxy.png` | Backend response via `/api/` proxy |

---

## Project Structure

```
.
├── conf/
│   └── secure-app.conf
├── html/
│   └── index.html
├── screenshots/
│   ├── https_working.png
│   ├── redirect_working.png
│   └── backend_proxy.png
└── README.md
```
