---
title: "Deploying Life Meets Code Web Server"
date: 2017-09-30T16:22:30-06:00
draft: false
type: "post"
tags:
  - linux
  - centos
  - server
  - cloud
  - apache
aliases:
  - /blog/deploying-life-meets-code-web-server
---

The [first part of this series](/posts/deploying-life-meets-code-the-server) dealt with setting up and securing the operating system for our server. Now we will take a look at getting Apache up-and-running with TLS.

> **Note:** During my initial setup, I used Vultr's Firewall to only allow traffic from my IP address. That way I could work without the fear of information disclosure.

## Apache

Apache itself is simple to install. We'll include `mod_ssl` to allow support for https.

```
$ sudo yum install httpd mod_ssl
$ sudo systemctl start httpd
```

You should now be able to visit `http://YOUR_SERVER_IP` in your browser and see Apache's default welcome page. If you do not, verify you enabled the appropriate ports in `firewalld`. You can also execute `sudo apachectl status` or `sudo journalctl -xe` for more information about what may have happened. Once you're able to see the welcome page in the browser, you can set Apache to start on boot.

```
$ sudo systemctl enable httpd
```

### Configuration

Even though I prefer using CentOS for my servers, I do like the config structure that Ubuntu/Debian-based systems use for Apache. Replicating it is not hard and gives a little more flexibility in turning things on and off.

First, make two directories to hold our personal configuration changes. `conf-available` will hold the config files we create. `conf-enabled` will contain symlinks to the files in `conf-available` that we want Apache to actually know about.

```
$ sudo mkdir /etc/httpd/conf-available
$ sudo mkdir /etc/httpd/conf-enabled
```

Next, make sure that the SELinux contexts match the other conf directories. By default, the new directories should have a type context of `httpd_config_t`.

```
$ ls -lZ /etc/httpd

drwxr-xr-x. root root system_u:object_r:httpd_config_t:s0 conf
drwxr-xr-x. root root unconfined_u:object_r:httpd_config_t:s0 conf-available
drwxr-xr-x. root root system_u:object_r:httpd_config_t:s0 conf.d
drwxr-xr-x. root root unconfined_u:object_r:httpd_config_t:s0 conf-enabled
drwxr-xr-x. root root system_u:object_r:httpd_config_t:s0 conf.modules.d
lrwxrwxrwx. root root system_u:object_r:httpd_log_t:s0 logs -> ../../var/log/httpd
lrwxrwxrwx. root root system_u:object_r:httpd_modules_t:s0 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx. root root system_u:object_r:httpd_config_t:s0 run -> /run/httpd
```

If you need to make a change, the `semanage` tool is quite handy. You'll need to install the `policycoreutils-python` package to obtain it.

```
$ sudo yum install policycoreutils-python
```

If you need to change it, `semanage-fcontext` will help with that.

```
$ sudo semanage fcontext --add --type httpd_config_t "/etc/httpd/conf-(.*)?"
$ sudo restorecon -Rv /etc/httpd
```

I have a single file that contains directives to improve the security of my production servers. It's something I've built up over time at work as new recommendations come in from network scan reports.

```
$ sudo vi /etc/httpd/conf-available/security.conf

---
TraceEnable off
ServerSignature off
ServerTokens Prod
FileETag None
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
---
```

Now create the symlink.

```
$ sudo ln -s /etc/httpd/conf-available/security.conf /etc/httpd/conf-enabled/security.conf
```

For security purposes, the `welcome.conf` file should be removed from production servers.

```
$ sudo rm /etc/httpd/conf.d/welcome.conf
```

I also make two quick initial changes to the main Apache config file.

```
$ sudo vim /etc/httpd/conf/httpd.conf   
```

Look for the ServerName directive and change it's value to localhost. This allows us to control all the sites configured on the server using VirtualHost files instead of having a domain attached to the default config.

```
# /etc/htttpd/conf/httpd.conf
...
ServerName locahost
...
```

Also, look for `<Directory /var/www/html>`. This is the default site config. Update the `Options` line to remove the ability to display directory indexes.

```
...
<Directory /var/www/html>
    Options -Indexes +FollowSymLinks
...
```

At the end of the file, add an `IncludeOptional` directive to import the config files we enable.

```
IncludeOptional conf-enabled/*.conf
```

Verify that the basic config tests pass and restart Apache.

```
$ sudo apachectl configtest
Syntax OK
$ sudo apachectl restart
```

### Domain(s)

At this point, we'll need a domain resolving to the server. If you have already updated the DNS of your domain to resolve to the IP address if your server, you're ready to go. If not, you should get that started since propagation could take some time. How you do this will vary based on your registrar and DNS provider. For now, a simple `hosts` file entry will get us up-and-going.

On your local machine, open your `hosts` file in a text editor. For Linux and Mac it's located at `/etc/hosts`. For Windows it's `C:\Windows\System32\drivers\etc\hosts`.

```
$ sudo vim /etc/hosts
```

Add a line that tells your PC where to resolve the domain to. Replace `SERVER_IP_ADDRESS` with the actual IP address of your server. Replace `mydomain.com` with your actual domain name.

```
SERVER_IP_ADDRESS    mydomain.com
```

We'll follow a similar directory structure for the domains as we did the configs. One for available sites and one for enabled sites.

```
$ sudo mkdir /etc/httpd/sites-available
$ sudo mkdir /etc/httpd/sites-enabled
```

Verify the SELinux contexts for the new directories like we did for the `conf-*` directories.

Open the `httpd.conf` file and add a line to include the `sites-enabled` directory.

```
$ sudo vim /etc/httpd/conf/httpd.conf
...
IncludeOptional sites-enabled/*.conf
```

Create a file in the `sites-available` directory for your domain.

```
sudo vim /etc/httpd/sites-available/mydomain.com.conf

#/etc/httpd/sites-available/mydomain.com.conf
<VirtualHost *:80>
    ServerName mydomain.com
    ServerAlias www.mydomain.com
    DocumentRoot /var/www/mydomain.com/

    ErrorLog /var/log/httpd/mydomain.com/error.log
    CustomLog /var/log/httpd/mydomain.com/requests.log combined
</VirtualHost>
```

In the above `VirtualHost`, we've added the primary domain as the `ServerName`, added the `www.` version as a `ServerAlias` and specified the `DocumentRoot` where the site will live. You can update values to match your domain.

We also specified domain-specific directories for the server logs. This is helpful for tracking down errors on your site if you're hosting multiple domains on a single server. Apache won't create the folders under `/var/log` for us so we'll manually create them.

```
$ sudo mkdir /var/log/httpd/mydomain.com
```

Now we can create the symlink to our site config file for Apache to read.

```
$ sudo ln -s /etc/httpd/sites-available/mydomain.com.conf /etc/httpd/sites-enabled/mydomain.com.conf
```

We also need to create the directory we used as the `DocumentRoot` for our `VirtualHost`.

```
$ sudo mkdir /var/www/mydomain.com
```

Verify that Apace likes everything we've done by running the config test again and restart Apache.

```
$ sudo apachectl configtest
Syntax OK
$ sudo apachectl restart
```

Create a basic `index.html` page in your domain's `DocumentRoot` to verify you can access the site from the browser.

```
$ vim /var/www/mydomain.com/index.html

#/var/log/mydomain.com/index.html
<h1>Welcome to mydomain.com</h1>
```

### TLS

With the existence of [Let's Encrypt](https://letsencrypt.org/), there is no longer an excuse to not have your site secured. Let's Encrypt offers free 90-day TLS/SSL certificates that can be set up for auto-renewal. When getting started, I followed the excellent [How to Secure Apache with Let's Encrypt on CentOS 7 post](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-centos-7) by [Erika Heidi](https://twitter.com/erikaheidi). Her post covers some stuff we've already done, but is worth the read. She also links out to additional resources for learning about TLS security. I'm going to summarize the steps we need to complete.

> **Note:** In order to set up Let's Encrypt, you will need your domain to resolve to the server.

Install the Let's Encrypt `certbot` tool for managing certificates. You can include all the domains and aliases you specified in the sites Apache config file.

```
$ sudo yum install python2-certbot-apache
$ sudo certbot --apache -d mydomain.com -d www.mydomain.com
```

The `certbot` wizard is pretty straight forward. Below is the output from the wizard at the time of this post. I do recommend choosing the redirect option to only allow traffic over `https`.

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): **you@email.com**
Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf. You must agree
in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A

-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: N
Obtaining a new certificate
Performing the following challenges:
tls-sni-01 challenge for mydomain.com
tls-sni-01 challenge for www.mydomain.com
Waiting for verification...
Cleaning up challenges
Created an SSL vhost at /etc/httpd/sites-enabled/mydomain.com-le-ssl.conf
Deploying Certificate for mydomain.com to VirtualHost /etc/httpd/sites-enabled/mydomain.com-le-ssl.conf
Deploying Certificate for www.mydomain.com to VirtualHost /etc/httpd/sites-enabled/mydomain.com-le-ssl.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting vhost in /etc/httpd/sites-enabled/mydomain.com.conf to ssl vhost in /etc/httpd/sites-enabled/mydomain.com-le-ssl.conf

-------------------------------------------------------------------------------
Congratulations! You have successfully enabled https://mydomain.com and
https://www.mydomain.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=mydomain.com
https://www.ssllabs.com/ssltest/analyze.html?d=www.mydomain.com
-------------------------------------------------------------------------------
...
```

`certbot` doesn't know about our directory structure, so it drops the new TLS `VirtualHost` file into the `sites-enabled` folder. We can quickly move the file to keep things consistent.

```
$ sudo mv /etc/httpd/sites-enabled/mydomain.com.conf-le.conf /etc/httpd/sites-available/mydomain.com-le.conf
$ sudo ln -s /etc/httpd/sites-available/mydomain.com-le.conf /etc/httpd/sites-enabled/mydomain.com-le.conf
```

Let's Encrypt automatically updated our original VirtualHost file to redirect to the https site. If you want to operate your domain without the www., you can tweak the file further.

```
$ sudo vim /etc/httpd/sites-available/mydomain.com.conf

#/etc/httpd/sites-available/mydomain.com.comf
#Replace %{SERVER_NAME} with your actual domain name
RewriteRule ^ https://mydomain.com%{REQUEST_URI} [END,QSA,R=permanent]
```

You'll also need to update the https VirtualHost that Let's Encrypt generated.

```
$ sudo vim /etc/httpd/sites-available/mydomain.com-le.conf

#Add these lines inside the VirtualHost
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
```

In my opinion, the default TLS configuration produced by Let's Encrypt is a little on the weak side, but that's understandable since they have to cater to a large audience. The default config only rates a C on [Qualys SSL Labs](https://www.ssllabs.com/). If you're not a TLS settings expert (which I am not), you can use the [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/) to produce a configuration to your liking.

If the server is only going to have one site or you know all your sites will share the same TLS configuration, you can update the main Let's Encrypt files. Otherwise, you can make changes at the VirtualHost level. For this example I'll be updating the main Let's Encrypt settings file to match a Modern configuration from the Mozilla SSL Configuration Generator.

```
$ sudo cp -a /etc/letsencrypt/options-ssl-apache.conf /etc/letsencrypt/options-ssl-apache.bak
$ sudo vim /etc/letsencrypt/options-ssl-apache.conf

#/etc/letsencrypt/options-ssl-apache.conf
...
SSLEngine on

# Intermediate configuration, tweak to your needs
SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite          ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
SSLHonorCipherOrder     on
SSLCompression          off

SSLOptions +StrictRequire

# HSTS (mod_headers is required) (15768000 seconds = 6 months)
Header always set Strict-Transport-Security "max-age=15768000"
...
```

One last piece of maintenance is to remove the default SSL `VirtualHost` that was installed with `mod_ssl`. We won't need the default one since `certbot` creates new SSL `VirtualHosts` for our domains.

```
$ sudo cp -a /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.bak
$ sudo vim /etc/httpd/conf.d/ssl.conf
```

Delete everything from `<VirtualHost _default_:443>` through `</VirtualHost>` and save the file.

Once again, verify that Apache likes the config changes we made and restart the web server.

```
$ sudo apachectl configtest
$ sudo apachectl restart
```

You should now be able to visit your site under `https`. One last step you should take after getting TLS up-and-running is to verify your setting are correct. The easiest way to do that is using SSL Labs. Simply enter your domain name and it will give you a letter grade for your configuration. At the time this was posted, the above config should result in an A+.

Now we need to set up the renewal for our certificates. We can do this easily by adding a line to the `crontab`.

```
$ sudo crontab -e

#Check certificate renewals every Monday at 2:30am
30 2 * * 1 /usr/bin/certbot renew >> /var/log/le-renew.log
```

This line checks on Mondays at 2:30 a.m. for any possible renewals and writes out the results to a log file. Even though the certificates last 90 days, it is good to check frequently to catch any issues with the renewals before they expire.

Now we have our web server set up. The next step will be to deploy our site.
