# Gotify Apache

Install and Configure Gotify without Docker in Apache Server.

## Requirements

- Ubuntu 18.04 64bit LTS
- Apache
- MYSQL

## Installation

- create new user

`Replace <user> with your username | if you are already having an user in your server skip the user creation step`

```bash
adduser <user>
```

- Add superuser group permission to unlock admin privileges

```bash
usermod -aG sudo <user>
```

- Login as user

```bash
su - <user>
```

- Update the packages

```bash
sudo apt-get update
```

```bash
sudo apt-get upgrade
```

- Create a Folder for Gotify Installation

```bash
sudo mkdir -p /var/www/gotifypush
```

- Replace `<user>` with the your username

```bash
sudo chown <user>:<user> /var/www/gotifypush
```

- Set correct Folder Permision

```bash
sudo chmod 775 /var/www/gotifypush
```

- Download Gotify

```bash
cd /var/www/gotifypush
```

- Download Latest Binary File From the Github respo <https://github.com/gotify/server/releases> Release Page

`Example`

```bash
wget https://github.com/gotify/server/releases/download/v2.0.5/gotify-linux-amd64.zip
```

```bash
chmod +x gotify-linux-amd64
```

- Create a New File Named as `config.yml`

```bash
sudo nano config.yml
```

- Just add this Below Configuration

> Leave `ssl` Settings | Add your domain in `stream` settings | Add your Database user and Pass in `database` settings | Update your Account Admin User & pass in `defaultuser` settings also update your gotify installation Directory.

```yml
# Example configuration file for the server.
# Save it to `config.yml` when edited

server:
  listenaddr: "" # the address to bind on, leave empty to bind on all addresses
  port: 9000 # the port the HTTP server will listen on

  ssl:
    enabled: false # if https should be enabled
    redirecttohttps: false # redirect to https if site is accessed by http
    listenaddr: "" # the address to bind on, leave empty to bind on all addresses
    port: 443 # the https port
    certfile: # the cert file (leave empty when using letsencrypt)
    certkey: # the cert key (leave empty when using letsencrypt)
    letsencrypt:
      enabled: false # if the certificate should be requested from letsencrypt
      accepttos: false # if you accept the tos from letsencrypt
      cache:  # the directory of the cache from letsencrypt
      hosts: # the hosts for which letsencrypt should request certificates
      - push.example.com
  
  responseheaders: # response headers are added to every response (default: none)
    Strict-Transport-Security: max-age=31536000
    X-Xss-Protection: 1; mode=block

  cors: # Sets cors headers only when needed and provides support for multiple allowed origins. Overrides Access-Control-* Headers in response headers.
    alloworigins:
      # - "example.com"
    allowmethods:
      # - "GET"
      # - "POST"
    allowheaders:
      # - "Authorization"
      # - "content-type"

  stream:
    allowedorigins: # allowed origins for websocket connections (same origin is always allowed)
  #    - ".+.example.com"
  #    - "push.example.com"

database: # for database see (configure database section)
  dialect: mysql
  connection: root:DBPASS@tcp(127.0.0.1)/gotifydb?charset=utf8&parseTime=True&loc=Local

defaultuser: # on database creation, gotify creates an admin user
  name: admin # the username of the default user
  pass: admin # the password of the default user
passstrength: 10 # the bcrypt password strength (higher = better but also slower)
uploadedimagesdir: /var/www/gotifypush/data/images # the directory for storing uploaded images
pluginsdir: /var/www/gotifypush/data/plugins # the directory where plugin resides
```

### Note Point

If you Enable Firewall on your server allow the port `9000` and Don't Forget to Create a Database - Create a New database in MYSQL Named as `gotifydb`.

- Next Create a New bash file named as `start.sh` in the Gotify Installed directory

```bash
#!/bin/bash

./gotify-linux-amd64
```

```bash
chmod +x start.sh
```

- Over all File listing in Gotify Installation Folder `var/www/gotifypush`

```bash
- config.yml
- start.sh
- gotify-linux-amd64
```

- Create a Vhost for Gotify
- Apache VHost Configuration for HTTP

```bash
sudo nano /etc/apache2/sites-available/gotifypush.conf
```

`gotifypush.conf`

```bash
<VirtualHost *:80>

    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.

    ServerName push.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/gotifypush
    Keepalive On

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf

  RewriteEngine on
  RewriteCond %{SERVER_NAME} =push.example.com
  RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

    ProxyPreserveHost On
    ProxyRequests off
    #ProxyVia Full

    # Proxy web socket requests to /stream
    ProxyPass "/stream" ws://127.0.0.1:9000/ retry=0 timeout=5
    # Proxy all other requests to /
    ProxyPass "/" http://127.0.0.1:9000/ retry=0 timeout=5
    ProxyPassReverse / http://127.0.0.1:9000/

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

- Enable Vhost

```bash
sudo a2ensite gotifypush.conf
```

- For SSL Use `https://certbot.eff.org/` (While a Creating SSL via Cetbot Select `NO` for Force SSL(HTTPS) Redirection Because already I added the Rewrite Rule for HTTPS Force Redirection)
- Vhost Configuration for HTTPS (Actually It will Automatically Created by `certbot` but we need to Add some Extra Configuration to use Gotify/Server via HTTPS)

```bash
sudo nano /etc/apache2/sites-available/gotifypush-le-ssl.conf
```

`gotifypush-le-ssl.conf`

```bash
<IfModule mod_ssl.c>
<VirtualHost *:443>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.

    ServerName push.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/gotifypush
    Keepalive On

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf

    SSLProxyEngine On
    SSLProxyVerify require
    SSLProxyCheckPeerName On
    SSLCertificateFile /etc/letsencrypt/live/push.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/push.example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf

        RewriteEngine on
        RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
        RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
        RewriteRule .* ws://127.0.0.1:9000%{REQUEST_URI} [P]

        ProxyPreserveHost On
        ProxyRequests off
        #ProxyVia Full

        # Proxy web socket requests to /stream
        ProxyPass "/stream" ws://127.0.0.1:9000/ retry=0 timeout=5
        # Proxy all other requests to /
        ProxyPass "/" http://127.0.0.1:9000/ retry=0 timeout=5
        ProxyPassReverse / http://127.0.0.1:9000/

</VirtualHost>
</IfModule>
```

```bash
sudo service apache2 restart
```

- After all setup open Gotify Root Folder `var/www/gotifypush`
- Run the Gotify & test the MYSQL database connection

```bash
./gotify-linux-amd64
```

- If the Test Passed Successfully Press CTRL + C to Stop the gotify/server
- Setup systemd service to Run the Gotify/server Forever

```bash
cd /etc/systemd/system
```

```bash
sudo nano gotifypush.service
```

`gotifypush.service`

`Replace <user> with the name of your user who will own this directory`

```bash
[Unit]
Description=Start Gotify - a simple server for sending and receiving messages
Requires=network.target
After=network.target

[Service]
Type=simple
User=<user>
WorkingDirectory=/var/www/gotifypush
ExecStart=/bin/bash /var/www/gotifypush/start.sh
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

- CTRL + X & Enter to save the Service file Configuration
- Next Enable the systemd service for Gotify server

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable gotifypush
```

```bash
sudo systemctl start gotifypush
```

- Check status

```bash
sudo systemctl status gotifypush
```

- Restart

```bash
sudo systemctl restart gotifypush
```

- That's Successfully we Install and Setup Gotify on Apache Server without Docker

## Update Gotify

Update Gotify to the Latest Version - <https://github.com/mskian/gotify-apache/blob/master/upgrade.md>

## LICENSE

MIT
