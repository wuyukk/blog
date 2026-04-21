+++
date = '2026-04-22T18:22:57+08:00'
draft = false
categories = ['blog']
title = 'Hugo Nginx'

+++
## nginx cfg
```
 location /webhook/ {
        allow 140.82.112.0/20;
        allow 192.30.252.0/22;
        allow 185.199.108.0/22;
        allow 127.0.0.1;
        deny all;

        proxy_pass http://127.0.0.1:9000/hooks/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
        proxy_buffering off;
    }
```

```
 location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot|webp)$    {
     expires 30d;
     add_header Cache-Control "public, immutable";
     log_not_found off;
     access_log off;
 }

 location ~* \.(html|htm|xml|txt|json)$ {
     expires 1h;
     add_header Cache-Control "public";
 }

 location ~ /\. {
     deny all;
     access_log off;
     log_not_found off;
 }

 location ~* \.(sql|bak|backup|sh|log|ini|conf|yml|yaml|md)$ {
     deny all;
     return 404;
 }
```

## /etc/systemd/system/webhook.service

```
[Unit]
Description=Webhook Service
After=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/bin/webhook -ip 127.0.0.1 -port 9000 -hooks /etc/webhook/hooks.json -verbose
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## deploy

```
#!/bin/bash

SOURCE_DIR="/home/anthony/blog-source"
WEB_DIR="/var/www/blog"
LOG_FILE="/var/log/webhook/deploy.log"

echo "$(date): ========== start deploy ==========" >> $LOG_FILE

cd $SOURCE_DIR || {
    echo "$(date): ERROR: cannot enter $SOURCE_DIR" >> $LOG_FILE
    exit 1
}

echo "$(date): fetching remote updates" >> $LOG_FILE
git fetch origin >> $LOG_FILE 2>&1

echo "$(date): force sync with remote main branch" >> $LOG_FILE
git reset --hard origin/main >> $LOG_FILE 2>&1

echo "$(date): updating submodules" >> $LOG_FILE
git submodule update --init --recursive >> $LOG_FILE 2>&1

echo "$(date): generating static files" >> $LOG_FILE
hugo --destination $WEB_DIR --minify -D >> $LOG_FILE 2>&1

echo "$(date): setting permissions" >> $LOG_FILE
chmod -R 755 $WEB_DIR >> $LOG_FILE 2>&1
```

