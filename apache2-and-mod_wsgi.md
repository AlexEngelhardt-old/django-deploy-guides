# Deploying Django with Apache2 and mod_wsgi

- I'll assume Ubuntu Server 18.04 here

### Set up the EC2 instance

- Start a t2.small EC2 instance with Ubuntu (18.04) Server, with public access.
- Run:
```
sudo apt-get update
sudo apt install apache2 apache2-dev virtualenv python3-dev python3-pip
sudo service apache2 restart
sudo ln -sf /usr/bin/python3 /usr/bin/python
```
- Apache2 auto-starts on install. You should be able to browse to your instance's public IP now. If not, try `sudo service apache2 status`. Also check your instance's security group if the machine really is accessible from outside.
- Run:
```
sudo chown -R www-data:www-data /var/www
sudo chmod -R g+w /var/www
sudo usermod -a -G www-data ubuntu

# Log out, log in again

mkdir -p /var/www/apps/
cd /var/www/apps
git clone https://github.com/AlexEngelhardt/hired-gun.git
cd hired-gun
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip install -r HiredGun/requirements.txt
# Install wsgi from pip, not via 'apt install libapache2-mod-wsgi':
# https://stackoverflow.com/questions/41005030

# TODO maybe do this from *outside* the virtualenvironment ?
sudo -H pip install mod_wsgi
```
- Add this to the bottom of `/etc/apache2/apache2.conf` (replacing the IP with your actual public IP):
```
WSGIScriptAlias / /var/www/apps/hired-gun/HiredGun/HiredGun/wsgi.py
WSGIPythonHome /var/www/apps/hired-gun/HiredGun/venv
WSGIPythonPath /var/www/apps/hired-gun/HiredGun

WSGIDaemonProcess 18.196.87.1 python-home=/var/www/apps/hired-gun/HiredGun/venv$
WSGIProcessGroup 18.196.87.1

<Directory /var/www/apps/hired-gun/HiredGun/HiredGun>
<Files wsgi.py>
Require all granted
</Files>
</Directory>
```
- Become root, activate your virtualenvironment, then run:
```
# service apache2 stop  # free port 80
mod_wsgi-express start-server HiredGun/wsgi.py --port 8080 --user www-data --group www-data
```

- From local, do `sudo apt install lynx` and then `lynx 127.0.0.1:8080` - this should work!
- Browse to your public IP at port :8000 and verify it's starting to look like your app (the static files don't work yet)
