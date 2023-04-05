# Basic Environment Setup For Havana

Instructions for setting up all dependencies

This assumes you have already setup a server with basic user acc privileges, configured a basic firewall setup, and setup Mariadb.

## Install Java 17
---

Description

```bash
apt-get update
apt-get upgrade
sudo apt install openjdk-17-jdk openjdk-17-jre
```



## Havana Setup
---
These are the instructions for downloading all of the relevant server and client files for Havana, along with instructions on setting everything up properly.

Download latest Havana release
```bash
wget https://github.com/Quackster/Havana/releases/download/release-14022023/Havana-1402023.zip
```

Download `unzip` if necessary

```bash
sudo apt install unzip
```

Unzip the release

```bash
unzip Havana-1402023.zip
```

Run both Havana-Server.jar and Havana-Web.jar at least once to generate the necessary configuration files, configure the MySQL attributes to connect to the MariaDB server.

```bash
chmod +x run_server.sh
chmod +x run_web.sh
./run_server.sh
./run_web.sh
```

Download the havana_www.zip file, and then extract it to /tools/www/ this directory is located where you ran Havana-Web.jar

(This is the default directory for static content within the Havana-Web project, but the directory where it looks for static images can be configured in the Housekeeping settings).

```bash
wget https://download1321.mediafire.com/nqv13n97fcogPKXNKSz_HLmCgh8kEEZt3JmtTkg-Xcd5GDfrfqN0j6A8Ir1P_16tBXZAUHryJPO2JTySAUJZeuxotu7z/x94neh4qbu3l2s2/havana_www.zip

unzip havana_www.zip -d tools/www
```

Linux users must install the font manager, to enable the captcha to work on the website.

```bash
apt-get install font-manager
```

Setup database

```bash
mysql -u admin -p
CREATE DATABASE havana;
exit
mysql -u admin -p havana < havana.sql
```

Configure `server.ini` and `webserver-config.ini` with the appropriate host and databse information

To open the game server up on the network the `server.ini` file may look like this 

```
[Global]
server.bind=0.0.0.0
server.port=12321

[Server]
server.port=12321
server.limit.bandwidth=false
server.limit.bandwidth.amount=40960

[Rcon]
rcon.bind=127.0.0.1
rcon.port=12309

[Mus]
mus.bind=0.0.0.0
mus.port=12322

[Database]
mysql.hostname=127.0.0.1
mysql.port=3306
mysql.username=admin
mysql.password=password
mysql.database=havana

[Logging]
log.received.packets=false
log.sent.packets=false

[Console]
debug=false
```

To open the web server up on the network the `webserver-config.ini` file may look like this

```
[Site]
site.directory=tools/www

[Global]
bind.ip=0.0.0.0
bind.port=80

[Rcon]
rcon.ip=127.0.0.1
rcon.port=12309

[Database]
mysql.hostname=127.0.0.1
mysql.port=3306
mysql.username=admin
mysql.password=password
mysql.database=havana

[Template]
template.directory=tools/www-tpl
template.name=default-en

page.encoding=utf-8
```

Both the web server and game server should run now, you can confirm by running both bash scripts `run_server.sh` and `run_web.sh`

### Configure Database 
---

We need to configure the `settings` table in the database with all of our proper site information. Pretty much we need to replace all occurences of `localhost` or `127.0.0.1` with our domain or IP whatever is being used at this stage. Port numbers should also match what was configured in the `.ini` files from the previous step.

### Configure Game Assets
---
We need to properly configure the `external_variables` file in `~/havana/tools/www/dcr/v31/gamedata` this file is used by the client to read in certain configuration options.

Pretty much we need to do a replace all on `http://localhost` with `http://our-domain-or-ip`

### Configure Firewall
---

We need to open up the ports for the game server and web server. Assuming no default values were changed that would mean enabling the following firewall rules.

```bash
sudo ufw enable http
sudo ufw enable https
sudo ufw enable 12321
sudo ufw enable 12322
```

### Start the game and web servers
---

Everything should be configured now to work so we can test this by running the game and web server.

I am running both processes in separate tmux sessions. Create a new named session, run the script then exit the session with `CTRL + B, D`

```bash
tmux new -s Game
./run_server.sh
```

```bash
tmux new -s Web
./run_web.sh
```

At this point both the web server and game server should be accessible over the internet.

### Additional steps
---

There are some missing tables in the database that we can add now but first we should give our admin user account the proper rank. User rank `7` is for admins so change the user rank in the table `users` to `7` and then upload `groups.sql` to the database

At the time of writing this `groups.sql` doesn't appear to be included with tagged releases so you need to either clone the repo or download the zipped contents

```bash
wget https://github.com/Quackster/Havana/archive/refs/heads/master.zip
unzip master.zip
```

Then from the `tools` directory where `groups.sql` is located execute the sql file against the database

```bash
mysql -u admin -p havana < groups.sql
```

This should add missing tables for Habbo Guides, SnowStorm, BattleBall, Wobble Squabble and Lido Diving gaming groups for the website.


There are a TON of leftover references to `classichabbo.com`, you can find them all via `grep` but here is a list of the main places to replace this reference

```
tools/www/web-gallery/styles/myhabbo/assets.css
tools/www-tpl/default-en/index_old.tpl
tools/www-tpl/default-en/games.tpl
tools/www-tpl/default-en/OLD_shockwave_app.tpl
tools/www-tpl/default-en/homes/widget/habblet/trax_song.tpl
tools/www/dcr/v31/crossdomain.xml
tools/www/dcr/v31/gamedata/external_texts.txt
tools/www/c_images/crossdomain.xml
tools/www/email.html
tools/www-tpl/default-en/me.tpl
```

## Minerva

Install .NET SDK and Runtime 6

```bash
sudo apt install snapd
sudo snap install dotnet-sdk --classic --channel=6.0
sudo snap alias dotnet-sdk.dotnet dotnet

sudo snap install dotnet-runtime-60 --classic
sudo snap alias dotnet-runtime-60.dotnet dotnet

export DOTNET_ROOT=/snap/dotnet-sdk/197/sdk/6.0.407/
sudo dotnet --info
```

Run Minerva

```bash
dotnet Minerva.dll
dotnet Minerva.dll --urls=http://listen-addr:5000/ --shockwave-badge-render
```

Allow port in firewall

```bash
sudo ufw allow 5000
```


At this point the website, the game server and the imager should be completely configured and accessible from the internet.

## Nginx + SSL

Install Nginx

```bash
sudo apt update
sudo apt install nginx
```

Permit HTTP and HTTPs ports in firewall

```bash 
sudo ufw allow 'Nginx Full`
```

Ensure the web server is running

```bash 
sudo systemctl status nginx
```

Leave the `default` configuration alone and setup a new server block for our server

```bash
sudo touch /etc/nginx/sites-available/your_domain
sudo nano /etc/nginx/sites-available/your_domain
```

In our case we will be setting up SSL as well so what we can do is quit exposing the Havan Web server and Minerva to the internet.
We will run on localhost and reverse proxy all web traffic through nginx and to our other services

We will redirect all traffic over HTTP to HTTPS

```
# Redirect traffic from port 80 for our domain to the server block listening on port 443
server {
	listen 80;
	listen [::]:80;

	server_name our_domain/com www.our_domain.com;

	# Redirect http traffic to the server block for https
	return 301 https://our_domain.com$request_uri;
}

# Redirect traffic from www.our_domain.com on port 443 to simply our_domain.com on 443
server {
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/our_domain/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/our_domain/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

	server_name www.our_domain.com

	# Redirect www prefixed domain to just https://our_domain.com
	return 301 https://our_domain.com$request_uri;
}

# Out main server block where web traffic will be proxied to our running services
server {
	listen [::]:443 ssl ipv6only=on; # managed by Certbot
	listen 443 ssl; # managed by Certbot
	ssl_certificate /etc/letsencrypt/live/our_domain/fullchain.pem; # managed by Certbot
	ssl_certificate_key /etc/letsencrypt/live/our_domain.com/privkey.pem; # managed by Certbot
	include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
	ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

	server_name our_domain.com;

        location /habbo-imaging {
                proxy_pass http://127.0.0.1:5000;
        }

        location / {
                proxy_pass http://127.0.0.1:8080;
        }
}
```

After we have configured our server block we need to create a syslink from `sites-available` to `sites-enabled`

```bash
sudo ln -s /etc/nginx/sites-available/our_domain /etc/nginx/sites-enabled/
```

To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the /etc/nginx/nginx.conf file. 
Open the file and Find the `server_names_hash_bucket_size` directive and remove the # symbol to uncomment the line

```bash
sudo nano /etc/nginx/nginx.conf
```

Next, test to make sure that there are no syntax errors in any of your Nginx files:

```bash
sudo nginx -t
```

This should return a success message indicating that your config is valid.
If there are no problems we can now restart nginx for our changes to take place

```bash
sudo systemctl restart nginx
```

We will install certbot to get an SSL certificate from Let's Encrypt and run a service to ensure the certificate auto renews

```bash
sudo apt install certbot python3-certbot-nginx
```

We need to ensure there is a server block matching our domain along with a `server_name` directive matching our domain.
Once that is confirmed we can request a certificate

If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.

If that’s successful, certbot will ask how you’d like to configure your HTTPS settings.


```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The certbot package we installed takes care of this for us by adding a systemd timer that will run twice a day and automatically renew any certificate that’s within thirty days of expiration.

You can query the status of the timer with systemctl

```bash
sudo systemctl status certbot.timer
```

To test the renewal process, you can do a dry run with certbot:

```bash
sudo certbot renew --dry-run
```

If you see no errors, you’re all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.
