# Project: Linux Server Configuration 

Configuration of linux server on Amazon lightsail

## Project information

IP Address: 18.196.63.238
URL: http://18.196.63.238.xip.io
SSH port 2200

grader SSH file password: ´graderPassword´

## Software installed with apt:

zsh
neovim
oh-my-zsh
( Just the way I like it :) )

finger
apache2
libapache2-mod-wsgi
flask
pip
postgresql

## Configuration

### update apt and upgrade

´´´sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo reboot´´´

### ssh config 

Changes in /etc/ssh/sshd_config :

´´´Port 2200
PermitRootLogin no
PasswordAuthentication no´´´

Rebooting sshd with:
´sudo service sshd restart´


### grader user setup

´sudo adduser grader´

Add the following line to /etc/sudoers.d/grader
´grader ALL=(ALL) NOPASSWD:ALL´

Then I generated a ssh key localy on my computer with ssh-keygen and putting the .pub part
in ~/.ssh/authorized_keys of the grader user. 


### ufw firewall

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable

## ItemsCatalog application deployment

The Item Catalog project is hosted as a wsgi application with apache and postgresql database

### Google oauth api

The url ´http://18.196.63.238.xip.io´ was added to the google oauth credentials and a new client_secret.json file
was downloaded.

### Postgres setup

Create postgres user for the ubunut user
´sudo -u postgres createuser -s ubuntu´

Create catalog database
´sudo -u postgres createdb catalog´

The line in the project.py and db_setup.py files from ItemCatalog project where the database
url was changed from "sqlite:///catalog.db" to "postgresql:///catalog"

I did not move all entries from the sqlite file to the posgresql database, but created a few entries for testing.

### WSGI application setup

The item catalog project was checked out from github into /var/www/ItemsCatalog

Where the client_secret.json file is loaded in the python code I had to give the full path
´/var/www/ItemsCatalog/client_secret.json´

A ItemsCatalog.wsgi file was created in the project folder:

´´´
import sys
sys.path.insert(0, '/var/www/ItemsCatalog/')

from project import app as application
from project import setup_app
setup_app()
´´´

The apache configuration file ´/etc/apache2/sites-enabled/000-default.conf´ was updated withs this content:

´´´
<VirtualHost *>
        ServerName http://19.196.63.238.xip.io/

        WSGIDaemonProcess project user=ubuntu group=ubuntu threads=5
        WSGIScriptAlias / /var/www/ItemsCatalog/ItemsCatalog.wsgi

    <Directory /var/www/ItemsCatalog>
        WSGIProcessGroup project
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>
</VirtualHost>
´´´

After everything is setup a restart of apache is needed:

´sudo apache2ctl restart´

## External resources used

Other than official documentation I used google and irc to find answeres to the thing I wasn't able to get
working.

URLS to docs used:
´https://modwsgi.readthedocs.io/en/develop/´
´https://www.postgresql.org/docs/10/index.html´
