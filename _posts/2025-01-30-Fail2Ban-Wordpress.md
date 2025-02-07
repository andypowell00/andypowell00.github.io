---
layout: post
title:  "Utilizing fail2ban on Lightsail Wordpress Site"
date:   2025-01-30 16:21:59 -0400
categories: wordpress php aws
tags: wordpress php aws lightsail bitnami
---

# Securing Your WordPress Site on AWS Lightsail with Fail2Ban

## Introduction

Fail2Ban is a powerful intrusion prevention tool that helps protect your server from brute-force attacks by monitoring log files and temporarily banning IP addresses that show malicious behavior.

We ran into an increase of bot activity on a particular site, which lead me to start researching some extra ways to mitigate any downtime on the site or possible hacks.  I found the use of regex against the error and access logs with excessive malicious bots was a very nice way to increase the safety of the site.

The big one for me was the xmlrpc.php file, which we had already denied access to, but even if enough bots hit that and get 403, it can be detrimental to site stability so wanted to take it further.  I noticed any IP that hit that attempted to hit that file either a GET or POST, was a malicious bot.  I checked the logs and looped through the IPs and checked them against various IP sites like https://www.abuseipdb.com/ to ensure no legimate traffic would be in this grouping.  Another trend was the bots hitting that file would also attempt to hit hundreds of other 404s and other attempts at hacks, so this helps mitigate other attacks as well.

## Step 1: Create Custom Fail2Ban Filters

### WordPress Login Protection
Create a filter for WordPress login attempts at `/etc/fail2ban/filter.d/wordpress-login.conf`:

```
[Definition]
failregex = <HOST>.* "POST .*wp-login.php
ignoreregex =
```

### XML-RPC Protection
Create a filter for XML-RPC attacks at `/etc/fail2ban/filter.d/wordpress-xmlrpc.conf`:

```
[Definition]
failregex = <HOST>.* "(GET|POST) .*xmlrpc.php
ignoreregex =
```



## Step 2: Configure Jail Settings

Edit `/etc/fail2ban/jail.local` with the following configurations:

```
[DEFAULT]
banaction = iptables-allports[blocktype=DROP]
persistent = true

[sshd]
enabled = true
filter = sshd
action = iptables[name=SSH, blocktype=DROP]
logpath = /var/log/auth.log
maxretry = 2
bantime = 86400
findtime = 600

[wordpress-login]
enabled = true
filter = wordpress-login
action = iptables-allports[name=WordPress-Login, blocktype=DROP]
logpath = /opt/bitnami/apache/logs/access_log
maxretry = 5
bantime = 600
findtime = 300

[wordpress-xmlrpc]
enabled = true
filter = wordpress-xmlrpc
action = iptables-allports[name=WordPress-XMLRPC, blocktype=DROP]
logpath = /opt/bitnami/apache/logs/access_log
maxretry = 1
bantime = 600
findtime = 300
```

## Helpful Diagnostic Commands

### Check Logs for Suspicious Activity
```bash
grep -E "xmlrpc\.php|wp-login\.php" error_log
```

### List Current IP Tables DROP Rules
```bash
iptables -L -n | grep DROP
```

### Verbose IP Tables Listing to see current IP bans
```bash
iptables -L -n -v
```

## Key Configuration Explanations

- `maxretry`: Number of attempts before ban
- `bantime`: Duration of IP ban (in seconds)
- `findtime`: Time window for tracking attempts
- `blocktype=DROP`: Completely blocks the IP address

## Best Practices

1. Regularly update Fail2Ban rules
2. Monitor server logs
3. Keep WordPress and plugins updated (learned this one the hardway)

**Note**: Always test configurations in a staging environment first to ensure no legitimate traffic is blocked.
