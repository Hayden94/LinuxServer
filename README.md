i. IP: 34.212.24.58<br />
PORT: 2200<br />
USERNAME: grader<br />
ii. 34.212.24.58/itemcatalog<br />
iii. Configurations include: User grader has super user permissions and can use sudo command, SSH key created and enforced, SSH is hosted on non default port, only allows connections for SSH(port 2200), HTTP(port 80), and NTP(port 123), and Item Catalog application hosted as a WSGI app.<br />
iv. SSH key is the 'grader_key.pem' file located within this repo.<br />
v. http://fosshelp.blogspot.in/2014/03/how-to-deploy-flask-application-with.html<br />
This was used to help direct me towards deploying my flask Item Catalog app.

# Step by step walkthrough

## 1. SSH into Ubuntu user
a. Open terminal and run `ssh ubuntu@34.212.24.58`

## 2. Update currently installed packages
a. `sudo apt-get update`
b. `sudo apt-get upgrade`
c. `sudo apt-get dist-upgrade`

## 3. Create grader user and add sudo permissions
a. `sudo adduser grader`
b. This will create user grader. Set the user password to: `grader`
c. Run `sudo usermod -aG sudo grader`
d. In order to test sudo permissions, switch to the grader user by running `su - grader`
e. Run a command that is only accessable to sudo users, such as listing the contents of `/root` by running the command `sudo ls -la /root`
f. Enter the respective password

## 4. Configure and install SSH key for grader user.
a. Return to your local 'Ubuntu' machine by pressing `Ctrl-D` in the terminal
b. To create your SSH key pair, run `ssh-keygen` in your home directory
c. Save the key and name it like so: `/home/Ubuntu/.ssh/ssh-grader`
d. Enter passphrase `grader`
e. Copy the contents of SSH public key by running `cat .ssh/ssh-grader.pub`
f. To install the key, log back into the grader user with `su - grader`
g. In your home directory, make a directory `mkdir .ssh` and create the file `touch .ssh/authorized_keys`
h. Edit the contents of the authorized keys, `sudo nano .ssh/authorized_keys`, paste in the contents of the SSH public key, and save the file.
i. Grant proper permissions on SSH by running `sudo chmod 700 .ssh` and `sudo chmod 644 .ssh/authorized_keys`
j. Test the SSH login back on your local machine by running `ssh grader@34.212.24.58 -i ~/.ssh/ssh-grader` and enter the correct passphrase.
k. The last step is to force key based authentication. Do this by editing the ssh config `sudo nano /etc/ssh/sshd_config`, change `port 22` to port `2200`, find the line `PasswordAuthentication yes` and change it to `PasswordAuthentication no`. Save the file and restart ssh with `sudo service ssh restart`

## 5. Disable root login
a. Edit SSH config with `sudo nano /etc/ssh/sshd_config`and change `PermitRootLogin yes` to `PermitRootLogin no`.
b. Save the file and restart ssh with `sudo service ssh restart`

## 6. Configure firewall
a. Check firewall status with command `sudo ufw status`, it should be disabled by default.
b. Set default policies with commands `sudo ufw default deny incoming` and `sudo ufw default allow outgoing`
c. Allow SSH on our non-default port with `sudo ufw allow 2200`
d. Allow http with `sudo ufw allow http`
e. Allow ntp with `sudo ufw allow ntp`
f. Finally, enable the firewall running `sudo ufw enable`

## 7. Configure web server and deploy flask application
a. Run and install the following packages `sudo apt-get install apache2 libapache2-mod-wsgi python-dev python-pip git`
b. Make the directory where the flask app will be located with `sudo mkdir /usr/share/ItemCatalog`
c. CD into the directory `cd /usr/share/ItemCatalog`
d. Clone the repository `git clone https://github.com/Hayden94/ItemCatalog.git`
e. Give proper file permissions with `sudo chmod 777 client_secrets.json catalog.py db_setup.py populator.py`
f. Install necessary pip packages
```
pip install virtualenv
pip install Flask
pip install render_template
pip install requests
pip install oauth2client
pip install sqlalchemy
```
g. Create a virtual environment with command `sudo virtualenv venv`
h. Activate the environment with `source venv/bin/activate`
i. Time to create the wsgi file by making the directory `sudo mkdir wsgi`
j. Create and edit the wsgi file with `sudo nano wsgi/flask.wsgi`
k. Within that file enter and save:
```
import os
import sys

activate_this = '/usr/share/ItemCatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

sys.stdout = sys.stderr

sys.path.insert(0, os.path.join(os.path.dirname(os.path.realpath(__file__)), '../..'))

sys.path.append('/usr/share/ItemCatalog/')

from catalog import app as application
```
l. Next step is to configure the apache settings for the flask app. Start by `cd /etc/apache2/conf-enabled`
m. Create the config file with `sudo nano itemcatalog.conf`
n. With that file enter:
```
WSGIScriptAlias /itemcatalog /usr/share/ItemCatalog/wsgi/flask.wsgi
WSGIScriptReloading On

<Directory /usr/share/ItemCatalog/wsgi>
  Order allow,deny
  Allow from all

  <IfModule mod_headers.c>
   Header set Access-Control-Allow-Origin "*"
  </IfModule>
</Directory>
```

## 8. Configure and install postgresql
a. Run and install `sudo apt-get install postgresql postgresql-contrib`
b. Postgres automatically creates a new user during installation access it with `sudo su - postgres` and then connect to the database with `psql`
c. Create a new user with password: `CREATE USER catalog WITH PASSWORD password;`
d. Give the user CREATEDB permissions: `ALTER USER catalog CREATEDB;`
e. Create the catalog database owned by catalog user: `CREATE DATABASE catalog WITH OWNER catalog;`
f. Connect to the database: `\c catalog`
g. Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;`
h. Lock down the permissions to only let catalog role create tables: `GRANT ALL ON SCHEMA public TO catalog;`
i. Log out of postgres: `\q` and return to grader user with `exit`

## 9. Create and populate database
a. Create the database with `python /usr/share/ItemCatalog/db_setup.py` and then populate the database with `python /usr/share/ItemCatalog/populator.py`
b. Restart your apache server by running the command `sudo service apache2 restart`
c. Project complete! You may now view the catalog app at http://34.212.24.58/itemcatalog

