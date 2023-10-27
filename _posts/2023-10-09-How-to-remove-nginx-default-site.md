---
title: How to remove Nginx defualt site
date: 2023-09-23 12:00:00 -500
categories: [Cloudflare, ZeroTrust]
tags: [Network]     # TAG names should always be lowercase
---


Method 1: Remove the Default Site Configuration

Open your NGINX configuration directory. Common locations are /etc/nginx/sites-available/ and /etc/nginx/sites-enabled/. Use cd to navigate to the appropriate directory.

List the files in the sites-enabled directory to see which configurations are currently active. You can use the ls command:

```sh
ls /etc/nginx/sites-enabled/
```
You should see a symbolic link (represented by an arrow ->) pointing to the default site configuration, usually named something like default or default.conf. To remove this link, use the unlink command:

```sh
sudo unlink /etc/nginx/sites-enabled/default
```
Replace /etc/nginx/sites-enabled/default with the actual path to the symbolic link if it's named differently in your system.

After unlinking the default site configuration, you'll need to reload NGINX to apply the changes:

```sh
sudo systemctl reload nginx
```