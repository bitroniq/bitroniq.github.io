---
layout: post

title: "How to update reverse dns (PTR) record on DigitalOcean"

excerpt: "If you have, for example Ubuntu 20.04 droplet on DigitalOcean, used
to host a WordPress using localhost MTA to send mail notifications, you need to
set up a Reverse DNS record to make sure your emails are delivered."

header:
  image: "assets/images/blog/2021-12-29-digitalocean-revdns.jpg"
  teaser: "assets/images/blog/2021-12-29-digitalocean-revdns.jpg"
  og_image: "assets/images/blog/2021-12-29-digitalocean-revdns.jpg"

tags:

- devops
- linux
- networking
- hypervisors
- cloud
- digitalocean
- dns

categories:

- DevOps

---

## Why do I need revDNS (PTR) record

If you have, for example Ubuntu 20.04 droplet on DigitalOcean, used to host a
WordPress using localhost MTA to send mail notifications, you need to set up a
Reverse DNS record to make sure your emails are delivered.

Usually the outgoing emails from the hosted sites are being rejected or marked
as spam by many mail servers.

Additionally, if the `A` DNS record for the droplet (VM) is not hosted on
Digital Ocean, but on a third party DNS hosting service, you don't have to move
it to DigitalOcean.

## How to setup a Reverse DNS (PTR) record for my droplet

The Reverse DNS is configured automatically from our end based on the droplet’s
hostname.

### To rename your droplet via the control panel

1. Login to the Digital Ocean Control Panel
2. Go to `Droplets` --> Click the droplet you want to rename
3. Then, on the droplet detail window, click on the name (on top)
   of your droplet (you wouldn’t know you could)
4. Change the name in the entry field and click the check mark

### To confirm the settings are correct

1. From the left menu select `Networking`
2. Select `PTR records` tab

### Finally, make sure you also edit your droplet’s hostname internally

1. Update `/etc/hostname` file
   - on Ubuntu use `sudo hostnamectl set-hostname subdomain.domain.com`
2. Update `/etc/hosts` file

   ```bash
   127.0.0.1 localhost subdomain
   127.0.1.1 subdomain.domain.com
   ```

3. Reboot your droplet (VM) with `sudo reboot`

### Verification

The PTR should be automatically adjusted in few hours due to DNS cache.

Check if it works using `host my-public-ip`
