---
date : '2026-04-21T16:34:56+08:00'
draft : false
title : 'webhook'
categories : ['blog']
---

# webhook status

```
sudo systemctl daemon-reload
```

```
sudo systemctl restart webhook
```

```
sudo systemctl status webhook
```

```
sudo netstat -tlnp | grep webhook
```

# webhook.service

```
vi /etc/systemd/system/webhook.service
```

# deploy.sh

```
vi /usr/local/bin/deploy.sh
```

# log

```
tail -f /var/log/webhook/deploy.log
```

