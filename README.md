# letsencrypt-nginx-automagic
A script and nginx config files that makes it easy to set up and maintain letsencrypt certificates
on Ubuntu 15.10 and quite possibly a lot of other debian-esque distributions.

The approach was inspired by this page:
https://johnmaguire.me/2015/12/configuring-nginx-lets-encrypt-automatic-renewal/


## Configuring nginx to allow certificate creation

If you want to have actual content on http, then simply include the config/lets-encrypt.conf file in
the server block for your http server, before the location section that serves your content:

```
server {
  listen   80;

  server_name foo.example.com;

  # The following 4 lines are exactly the same as using the include line, the content is included
  # here for clarity, but if there are many virtual hosts, then including a single config file
  # in each server section is much nicer:
  # include /opt/letsencrypt-nginx-automagic/config/lets-encrypt.conf;

  location /.well-known/acme-challenge {
    default_type  "text/plain";
    root          /opt/letsencrypt-nginx-automagic/acme;
  }

  # Do whatever you need to serve the rest of the location, like proxying for a backend:
  location / {
    proxy_pass http://10.0.3.100/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-forwarded-for $remote_addr;
  }
}
```


If you simply  want to redirect everybody away from http towards https,
then the configuration is much easier and requires just one config file,
ike my /etc/nginx/sites-enabled/default.conf:

```
server {
  listen      80 default_server;
  
  location /.well-known/acme-challenge {
    default_type  "text/plain";
    root          /opt/letsencrypt-nginx-automagic/acme;
  }
    
  location / {
    return 301 https://$host$request_uri;
  }
}
```

Whenever a configuration change is made to nginx run: sudo service nginx reload


## Creating a certificate

For each new certificate simply run the script as root like this:
```
sudo /opt/letsencrypt-nginx-automagic/letsencrypt-automagic foo.example.com
```

The script will clone https://github.com/letsencrypt/letsencrypt if it hasn't yet done
so and use letsencrypt-auto to create the new certificate.


## Automatic renewal

When the script is run without parameters it will renew all the existing certificates,
so to automatically renew all certificates, just put a line like this one in roots
crontab:

0 5 1 * * /opt/letsencrypt-nginx-automagic/letsencrypt-automagic > /opt/letsencrypt-nginx-automagic/last-renewal.log 2>&1


## Configuring nginx to use a certificate

