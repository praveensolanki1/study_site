Step 1: Creating a VPS(Droplet) on Digital Ocean
Go to https://cloud.digitalocean.com/droplets and click on Create Droplet the button. Then, select Ubuntu version.
Give any name of it.

Step 2: Installing dependencies
#!/bin/bash

# Update 
sudo apt update

# Install Nginx
sudo apt install -y nginx

# Install Python
sudo apt install -y python3 python3-pip

# Install Virtualenv
sudo pip3 install virtualenv

# start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# It reloads the systemd manager configuration.
sudo systemctl daemon-reload

Step 3: Setting up our project and its environment:
ssh to your server using the terminal.
Clone Your Django Project and create a virtual environment inside that directory.

Then install requirements.txt file 
pip install -r requirements.txt
pip install django gunicorn

This installs Django and gunicorn in our virtual environment
Add your IP address or domain to the ALLOWED_HOSTS variable in settings.py.

If you have any migrations to run, perform the action:
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic 

add these lines in settings.py file 
add this line “whitenoise.runserver_nostatic”, into your Installed_apps of setting file.
add ‘whitenoise.middleware.WhiteNoiseMiddleware’, into MiddleWare of your setting File.
Also, add these lines at the bottom of the project/urls. py file.

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root = settings.STATIC_URL)


Run this command

> pip install whitenoise

Configuring gunicorn:
Deactivate the virtual environment by executing the command below:

deactivate 

Create a system socket file for gunicorn:
sudo vim /etc/systemd/system/gunicorn.socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

Next, we will create a service file for gunicorn

sudo vim /etc/systemd/system/gunicorn.service
add these content and save it.

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
[Service]
User=root
Group=www-data
WorkingDirectory=/root/YourProjectDirectoryName
ExecStart=/root/YourProjectDirectoryName/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          yourSettingsFileFoldername.wsgi:application
[Install]
WantedBy=multi-user.target  

Lets now start and enable the gunicorn socket  
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

Configuring Nginx as a reverse proxy
Before You create an Nginx File.
With this command, you can check if a file already exists.
cd /etc/nginx/sites-enabled
You can delete the existing default file using the command.

sudo rm -f FileName

Create a configuration file for Nginx using the following command:
sudo vim /etc/nginx/sites-available/blog

Paste the below contents inside the file created

server {
    listen 80 default_server;
    server_name _;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /YourStaticFilesDirectoryName/ {
        root /root/YourProjectDirectoryName;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}

Create a configuration file for Nginx using the following command:
sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/
Run this command to load a static file
$ sudo gpasswd -a www-data username
Restart Nginx and allow the changes to take place.

sudo systemctl restart nginx
sudo service gunicorn restart
sudo service nginx restart 
  
  
  


  












  
