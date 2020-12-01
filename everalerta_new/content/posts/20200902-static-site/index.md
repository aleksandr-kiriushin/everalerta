---
title: "Static site (blog)"
date: 2020-09-02T22:40:20+03:00
toc: false
tags: ["blog", "scaleway", "hugo", "cloudflare", "vps", "nginx", "uwf"]
---


## Domain

1. Create a custom domain using previous [post]({{< ref "20200902-custom-domain/" >}} "post")

## Static site

1. Install [Hugo](https://gohugo.io/), static site generator
2. Choose Hugo theme using [site](https://themes.gohugo.io/)
3. Use [Quick start](https://gohugo.io/getting-started/quick-start/) to create a simple blog
4. Check how it looks like using command (at localhost):
{{< cmd >}}
hugo server -D
{{< /cmd >}}
5. Compile a public directory with your site using command:
{{< cmd >}}
hugo -D
{{< /cmd >}}

## Server

1. Register an account in [Scaleway](https://www.scaleway.com/)
2. Create ssh key in new directory ~/.ssh/scaleway/:
{{< cmd >}}
ssh-keygen -t rsa -b 4096 -C "aleksandr.kiriushin@everalerta.com"
{{< /cmd >}}
2. Buy a smallest DEV VPS server (about 5$) with Ubuntu
{{< img "scaleway.png" "Scaleway" >}}
3. Note the server's IP
4. Ssh to server, set server's IP into the next command:
{{< cmd >}}
ssh -i ~/.ssh/scaleway/id_rsa root@<IP>
{{< /cmd >}}
5. Update software:
{{< cmd >}}
apt update && apt upgrade
{{< /cmd >}}
6. Setup uwf firewall:
{{< cmd >}}
ufw default allow outgoing
ufw default deny incoming
ufw allow ssh
ufw allow 80/tcp
ufw enable
{{< /cmd >}}
7. Install nginx:
{{< cmd >}}
apt install nginx
systemctl enable nginx
{{< /cmd >}}
8. Create a directory for site:
{{< cmd >}}
mkdir -p /everalerta/public/
chmod +r /everalerta/
chmod +r /everalerta/public/
{{< /cmd >}}
9. Setup a config for Nginx:
{{< cmd >}}
vim /etc/nginx/sites-available/everalerta
ln -s /etc/nginx/sites-available/everalerta /etc/nginx/sites-enabled/everalerta
rm -rf /etc/nginx/sites-enabled/default
systemctl restart nginx
tail -f /var/log/nginx/{access,error}.log
{{< /cmd >}}

{{< code >}}
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name everalerta.com www.everalerta.com;
        root /everalerta/public/;
        index index.html;  
        location / {
                try_files $uri $uri/ =404;
        }
}
{{< /code >}}

10. Copy Hugo static site to server:
{{< cmd >}}
scp -r -i ~/.ssh/scaleway/id_rsa public root@<IP>:/everalerta/
{{< /cmd >}}
11. Set server's IP into Cloudflare's DNS A-record
12. Setup HTTP to HTTPS redirect in Cloudflare settings
13. Try to open your site in browser
