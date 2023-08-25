---
layout: post
title:  "Add new SSH Key to Github in Ubuntu"
date:   2023-08-23 16:21:59 -0400
categories: github ssh
tags: github 
---

# Adding new SSH key to your Github Account for a new dev machine

I recently got a new laptop and had to get all my credentials re setup which is always a fun time.  Luckily Github makes this process very easy to get started cloning repos via SSH super easy. [Github documents this process for other operating systems as well.](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=linux). 

Copy the contents of your public SSH key:

```bash
$ cat ~/.ssh/id_rsa.pub
```

If you don't have one generated already go ahead and use the ssh-keygen command to do so:

```bash
$:~/.ssh$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ap/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ap/.ssh/id_rsa
Your public key has been saved in /home/ap/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:randomkeynameH0Ix5coF9wSBA/gNUY6lxADpKv7LJGwXJWxA $devicename
The key's randomart image is:
+---[RSA 3072]----+
|.E*+*=+o..       |
|oo+.+O.oo        |
|o=o=* *.         |
|o.*+ *           |
|.. o= o S        |
|  . .= o         |
| o .o . .        |
|. =  . .         |
| =.              |
+----[SHA256]-----+
```

Now in your Github account go to Settings->SSH and GPG keys->Add new->paste in your key and confirm. 

You're all set to start pulling from your local machine now.


