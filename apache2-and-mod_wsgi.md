# Deploying Django with Apache2 and mod_wsgi

## Set up the EC2 instance

- Start a t2.small EC2 instance with Ubuntu (18.04) Server, with public access.

- Run:
```
sudo apt update
sudo apt install -y apache2
sudo apt install -y libapache2-mod-wsgi-py3
sudo apt install -y git vim
sudo apt install -y build-essential python3 python3-dev python3-pip python3-venv
sudo pip3 install virtualenv
```

- Apache2 auto-starts on install. You should be able to browse to your instance's public IP now. If not, try `sudo service apache2 status`. Also check your instance's security group if the machine really is accessible from outside.

- Inside of **/home/ubuntu/** directory Run:
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
python manage.py collectstatic
deactivate
```

- Add instance IP address to `ALLOWED_HOSTS` list in django settings
```
ALLOWED_HOSTS = ['IP_ADDRESS_HERE']
```

- Fix base template dir in django settings
```
'DIRS': [os.path.join(BASE_DIR, '..', 'templates'),],
```

- Configure Apache
```
sudo vim /etc/apache2/sites-available/000-default.conf

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

sudo chown :www-data ~/project
sudo apache2ctl configtest
sudo systemctl restart apache2
```