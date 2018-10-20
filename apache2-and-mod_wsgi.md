# Deploying Django with Apache2 and mod_wsgi


## TODOs


- Set DEBUG=False
- Why do we need to change the template DIRS?


## Set up the EC2 instance


- Start a t2.small EC2 instance with Ubuntu (18.04) Server, with public access.

- Run:
```
sudo apt update
sudo apt upgrade

sudo apt install -y apache2 libapache2-mod-wsgi-py3 vim \
  build-essential python3 python3-dev python3-pip python3-venv
sudo pip3 install virtualenv
```

- Apache2 auto-starts on install. You should be able to browse to your instance's public IP now. If not, try `sudo service apache2 status`. Also check your instance's security group if the machine really is accessible from outside.


## Configure Django


- Inside the **/home/ubuntu/** directory run:
```
git clone https://github.com/AlexEngelhardt/hired-gun.git project
```

- Create virtual environment
```
cd project
virtualenv -p python3 .venv
source .venv/bin/activate
```

- Configure Django project
```
cd HiredGun/
pip install -r requirements.txt 
python manage.py migrate
python manage.py loaddata projects/fixtures/auth.json
python manage.py loaddata projects/fixtures/Client.json
python manage.py loaddata projects/fixtures/Project.json
python manage.py loaddata projects/fixtures/Session.json
python manage.py collectstatic
deactivate
```

- Add instance IP address to `ALLOWED_HOSTS` list in django settings. This is the list of hosts this Django sites **can serve**, i.e. not the visitors' IP addresses!
```
export LOCAL_IPV4=`curl http://169.254.169.254/latest/meta-data/public-ipv4`
sed -i "s/ALLOWED_HOSTS = \[\]/ALLOWED_HOSTS = \['$LOCAL_IPV4'\]/" HiredGun/settings/common.py
```

- Fix base template dir in django settings
```
# 'DIRS': [os.path.join(BASE_DIR, '..', 'templates'),],
sed -i "s/'DIRS': \['.\/templates',\],/'DIRS': \[os\.path\.join(BASE_DIR, '\.\.', 'templates'),\],/" HiredGun/settings/common.py
```

### Set production values

```
head -c 16 /dev/urandom | md5sum | cut -f 1 -d\ > HiredGun/secret.txt
sed -i 's/HiredGun\.settings\.development/HiredGun\.settings\.production/' manage.py
sed -i 's/HiredGun\.settings\.development/HiredGun\.settings\.production/' HiredGun/wsgi.py
```

## Configure Apache


```
sudo bash -c 'cat << EOF > /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /home/ubuntu/project/HiredGun
    ErrorLog /tmp/error.log
    CustomLog /tmp/access.log combined

    WSGIDaemonProcess hired_gun python-home=/home/ubuntu/project/.venv python-path=/home/ubuntu/project/HiredGun
    WSGIProcessGroup hired_gun
    WSGIScriptAlias / /home/ubuntu/project/HiredGun/HiredGun/wsgi.py

    Alias /static /home/ubuntu/project/HiredGun/HiredGun/static
    <Directory /home/ubuntu/project/HiredGun/HiredGun/static>
        Require all granted
    </Directory>

    <Directory /home/ubuntu/project/HiredGun/HiredGun>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
</VirtualHost>
EOF'
```


## Start Apache


```
sudo chown -R :www-data ~/project
sudo chmod -R g+w ~/project
sudo apache2ctl configtest
sudo systemctl restart apache2
```
