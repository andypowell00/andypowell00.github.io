---
layout: post
title:  "Wordpress on AWS Lightsail Randomly Unavailable"
date:   2023-07-02 16:21:59 -0400
categories: wordpress php aws
tags: wordpress php aws lightsail bitnami
---

We recently launched a Wordpress site for a client that was adament about hosting on AWS.  Their site was medium-low traffic, and they wanted up front pricing instead of pay-as-you-go, so one of the only options for that on AWS is Lightsail.  

This was super easy to setup and all seemed well, during the soft launch period, I noticed at certain points the site would be unreachable.  No alert from AWS, no over used CPU usage alerts or anything like that, Wordpress would just not be able to be reached. After doing some research I found a several other people with similar experiences hosting Wordpress on Lightsail.  

The first issue I found in the apache error and access logs was a common Wordpress exploit with bots hitting XML-RPC.php file.  Everytime a bot would attempt to access this it was using memory that should available for other operations.  This led to me to look at the 'top' command in linux and noticed there was no swapfile setup for this instance.

A common fix for XML-RPC.php is adding a rule in .htaccess to deny access.  Lightsail Wordpress disables .htaccess by default, so I had to find another place to add this in.  Instead I had to place it /opt/bitnami/apache/conf/vhosts/wordpress-https-vhost.conf.

```bash
<Files xmlrpc.php>
order deny,allow
deny from all
</Files>
```

Next I added a swapfile, my instance is 4GB of RAM so I made it 2GB. I followed [this guide](https://www.webhostingforbeginners.net/how-to-add-swap-space-on-aws-lightsail-server-instance/){:target="_blank"} with some minor changes and was super easy to setup.

It's been a few weeks since these updates and the site hasn't gone down again.  