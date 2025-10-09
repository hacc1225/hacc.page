---
title: Setting Up IPv6 on Arch Linux on Vultr
published: 2018-04-23T02:50:00Z
description: ''
updated: 2025-10-09T05:43:23.522Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'en'
abbrlink: 'vultr-archlinux-ipv6-setup'
---

As a firm believer in the motto "Life lies in tinkering," I swapped the OS on my VPS to Arch Linux shortly after getting it. For convenience during setup, I just used `dhcpcd` to configure the IP address automatically.

The automatic configuration for IPv4 worked perfectly, but I ran into an issue with IPv6. I had never used IPv6 to host a website before, so the problem didn't surface until I recently had a whim to use Cloudflare's IPv6 gateway.

The server could ping external IPv6 hosts, but external hosts couldn't ping the server. After checking the `dhcpcd` logs with `systemctl status dhcpcd`, I discovered the issue: the IPv6 address it automatically acquired was not the same one as the static IPv6 address provided by Vultr.

I looked up the [man page for `dhcpcd.conf`](https://man.archlinux.org/man/dhcpcd.conf.5) and found the method for static address configuration. I just had to add the following to the end of `/etc/dhcpcd.conf`:

```
interface eth0 #Your network interface name
static ip6_address= #Your static IPv6 address
```

Then, a quick restart with `sudo systemctl restart dhcpcd` and everything was working.

---

However, there was one more step. During the initial Nginx setup, I hadn't configured it to listen on the IPv6 address. To use Cloudflare's IPv6 gateway, I also needed to have one of the following lines in my `server` block:

```
listen [::]:80;
```

or

```
listen [::]:443 ssl http2;
```

> A note from Future Hacc:
>
> As a `systemd` disciple these days, I'd probably just go all-in with the full `systemd` suite.
>
> Although it's now impossible to figure out what exactly happened back then to cause the situation where the VPS could ping external hosts but not the other way around, if I had to gaze into my crystal ball, my guess would be this: `dhcpcd` likely used Vultr's RA to automatically configure an address via SLAAC. While this address was valid, it wasn't the primary static address assigned to the VPS in the Vultr control panel, which led to the inbound connection issue.
