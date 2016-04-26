---
layout: post
title: 4 Steps to Get and Use a Free SSL Certificate with Let's Encrypt
description: A simple guide to get a free SSL/TLS certificate using Let's Encrypt and to configure nginx to use.
keywords: SSL, TLS, SSL/TLS, SSL certificate, letsencrypt, Let's Encrypt, http, https, nginx, ssh, git, github, IT
---

This post will show you how to get, install and use a free SSL certificate using Let's Encrypt.
[Let’s Encrypt](https://letsencrypt.org) is a free, automated, and open certificate authority (CA).
Let's Encrypt not only provides free SSL certificates but also makes much easier the
previously tedious process of getting and renewing them.

## 1. Installing Let's Encrypt

In this first step, we will install Let's Encrypt tools on the machine where the SSL certificate
is going to be used.
First, clone the Let's Encrypt repository:

```bash
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
```

Then run `./letsencrypt-auto --help`, this will install some dependencies and configure the letsencrypt
environment. Towards the end of the command's output you will see a line like the following:

```
Requesting root privileges to run letsencrypt...
   sudo /home/<user>/.local/share/letsencrypt/bin/letsencrypt --help
```

From now on, you can run `letsencrypt` using:

```bash
sudo /home/<user>/.local/share/letsencrypt/bin/letsencrypt
```

## 2. Set Up Your Domain and Web Server

To verify that you own the domain for which you are trying to generate an SSL certificate, the `letsencrypt`
tool will generate a challenge that the tool will then try to access through your domain name.

The `letsencrypt` command to generate a certificate looks something like:

```bash
letsencrypt certonly --webroot -w /var/www/example -d example.com
```

Where `example.com` is the domain for which to generate the SSL certificate.
The tool will place the challenge file in the `/var/www/example/.well-known/acme-challenge/` directory.
It will then look for the challenge at `http://example.com/.well-known/acme-challenge/<challenge>`.
For the certificate generation to succeed the challenge must be found, therefore you must:

- Have your DNS configured so that `example.com` points to your machine
- Have your web server set up to serve files in `/var/www/example/.well-known/acme-challenge/`.

It might be the case that you have a web server already configured. Consider, for instance, the case
where Nginx is running on your machine as a reverse proxy, not serving any static content.
Even in this case you can make `letsencrypt` work by ensuring that `http://example.com/.well-known/acme-challenge/`
properly serves the files in `/var/www/example/.well-known/acme-challenge/`. To do this add
the following to the proper `server` configuration in Nginx's `sites-enabled` directory:

```
server {
  ...
  location /.well-known/acme-challenge/ {
    autoindex on;
    root /var/www/example/.well-known/acme-challenge/;
  }
  ...
}
```

## 3. Generate your SSL Certificate

Now that your DNS and web server are configured everything is set to generate the SSL certificate:

```bash
sudo letsencrypt certonly --webroot -w /var/www/example -d example.com
```

If run correctly this command will output the location of the generated certificates and key:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your cert will
   expire on 2016-08-01. To obtain a new version of the certificate in
   the future, simply run Let's Encrypt again.
 - If you like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

In the `/etc/letsencrypt/live/example.com/` directory you will find the following files:

```
cert.pem  chain.pem  fullchain.pem  privkey.pem
```

Notice that Let's Encrypt does not support wildcard certificates
([for the moment](https://community.letsencrypt.org/t/please-support-wildcard-certificates/258)). However,
you can generate a certificate for more than one subdomain, as follows:

```bash
sudo letsencrypt certonly --webroot -w /var/www/example -d example.com -d www.example.com -d blog.example.com
```

For this command to succeed the challenge must be reachable through all the specified subdomains (e.g.
`http://example.com/.well-known/acme-challenge/`, `http://www.example.com/.well-known/acme-challenge/` and
`http://blog.example.com/.well-known/acme-challenge/` must all point to `/var/www/example/.well-known/acme-challenge/`).

## 4. Use Your SSL Certificate with Nginx

The generated certificates and key can be found in `/etc/letsencrypt/live/example.com/`:

```
cert.pem  chain.pem  fullchain.pem  privkey.pem
```

To use the generated SSL certificate open your site's configuration (in Nginx's `site_enabled` directory)
and add the following:

```
server {
    ...
    listen              443 ssl;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}
```

Now you can go visit `https://example.com` and enjoy your free SSL certificate.