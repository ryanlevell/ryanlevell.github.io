---
layout: post
title:  "Using CaddyServer For Automatic HTTPS"
date:   2016-04-2 10:28:03 -0500
categories: caddyserver aws namecheap domain dns https mac
---

### Intro
I often read articles at [Hacker News](https://news.ycombinator.com/) and found the article [Let's Encrypt and Nginx – State of the art secure web deployment](https://letsecure.me/secure-web-deployment-with-lets-encrypt-and-nginx/).  
As I often do, I diverted all my attention to making my own server use HTTPS by default. I was excited to follow these steps and write a post about my experience, but I found something more interesting in the [comments](https://news.ycombinator.com/item?id=11399476) of the article - [Caddy](https://github.com/mholt/caddy).

### Caddy
As stated on the GitHub page, "Caddy is a lightweight, general-purpose web server for Windows, Mac, Linux, BSD and Android. It is a capable alternative to other popular and easy to use web servers."   

I have a domain that I am not using, `levell.xyz`. I wanted to server HTTPS by default with this domain.

#### Creating a server
I used AWS to create an Ubuntu server box.

1. Launched a new nano instance from the AWS console.
 - Nano was a new size to me, 1/2GB RAM and $.0065/hour.
2. I added an elastic IP in the AWS console - basically a static IP.
3. I successfully logged in via SSH.
 - AWS uses SSH keys by default.
 - `ssh -i <path to .pem key file> <username>@<ip address>`
 - AWS Ubuntu uses user `ubuntu` by default.
 - The IP address can be found in the EC2 dashboard.
 - The pem is downloaded from AWS when you launch your first instance.
4. Unrelated/just for fun, I changed SSH to use port 7899 for additional security
 - This file, on Ubuntu, can be found at `/etc/ssh/sshd_config`
 - Change `Port 22` to `Port <some port>`
 - `ssh -i <path to .pem key file> -p <some port> <username>@<ip address>`

#### Installing Caddy
I used the [quick start](https://github.com/mholt/caddy#quick-start).

1. Find the latest release [here](https://github.com/mholt/caddy/releases/latest).
2. Download: `wget https://github.com/mholt/caddy/releases/download/<latest release here>/caddy_linux_amd64.tar.gz`
3. Extract: `tar -xvf <caddy .tar.gz file>`
 - Move this to the you website's root folder (this can be anywhere you choose)

#### Running Caddy from an IP address

Add `Caddyfile` to your website's root folder:

```
<AWS IP address here>

gzip
browse
ext .html
websocket /echo cat
log ../access.log
header /api Access-Control-Allow-Origin *
```

Run caddy:

```
~ $: ./caddy
Activating privacy features... done.
<your IP address>:2015
Warning: File descriptor limit 1024 is too low for production sites.
At least 4096 is recommended. Set with "ulimit -n 4096".
```

Port 2015 is used by default by Caddy.  
- Port 2015 must be opened in your AWS inbound security group.

Add an index.html to your website's root folder:

```
<html>
<body>
testing
</body>
</html>
```

Verify the server is working by hitting `<your IP address>:2015` from a browser.  
- Notice there is no HTTPS - localhost and IP addresses will not use HTTPS.

#### Running Caddy from a domain
I bought a domain from [namecheap](https://namecheap.com). These instructions will be namecheap specific.

1. Log in to namecheap
2. Navigate to the dashboard
3. Click on `MANAGE` of the domain you want to use
4. Click the `Advanced DNS` tab
5. Create 2 Host Records
 - `A Record` with `Host=@` and `Value=<your AWS IP>`
 - `CNAME Record` with `Host=www` and `Value=<your domain>`
 - The CNAME is optional. It redirects `www.domain.com` to `domain.com`
6. Save all changes
7. Wait for DNS to update with your new records
 - This could take a very long time

Now we can update the `Caddyfile` with our domain. We need 2 entries if we included the CNAME

```
<your domain> {
 gzip
 browse
 ext .html
 websocket /echo cat
 log ../access.log
 header /api Access-Control-Allow-Origin *
}

www.<your domain> {
 gzip
 browse
 ext .html
 websocket /echo cat
 log ../access.log
 header /api Access-Control-Allow-Origin *
}
```

Run caddy again:

```
~ $: ./caddy
Activating privacy features...2016/04/06 03:33:11 [levell.xyz] failed 
to get certificate: [levell.xyz] error presenting token: Could not
start HTTP server for challenge -> listen tcp :80: bind: permission denied
```

I sadly got an error. I thought maybe the DNS server did not update yet or I set up the host records wrong. I eventually realized my problem was simpler. When using a domain, `sudo` must be used.

```
~ $: sudo ./caddy
```

In a browser navigate to `<your domain>` and you should see `https://<your domain>` automatically. It's that simple.

Since I am using GitHub Pages I have since shutdown this AWS instance. I didn't have anything at the moment to put on the server, but I left two points:

1. Can I use it with a web server such as Tomcat?
2. To start at bootup, I think I could just add a service to `/etc/init.d` to run caddy.
