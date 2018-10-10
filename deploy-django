#!/bin/bash

if ["$1" -eq "-h"]
    then
        echo "2 aguments need:
        1: app name
        2: username
        3: server ip address"
fi

if [ -z "$1" ]
    then
        echo "Name of app not supplied"
        exit 1
fi

if [ -z "$2" ]
    then
        echo "User not provided"
        exit 1
fi

if [ -z "$3" ]
    then
        echo "Server Ip is missing"
        exit 1
fi

APPNAME=$1
USERNAME=$2
IPADDRESS=$3

tput setaf 2; echo "installing packages"
tput sgr 0;
sudo apt update
sudo apt install -y python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx

tput setaf 2; echo "installing pip and virtualenv"
tput sgr 0;
sudo pip3 -H install --upgrade pip
sudo pip3 -H install virtualenv

tput setaf 2; echo "setting up new venv"
tput sgr 0;
cd ${PWD}/${APPNAME}
virtualenv -p python3 venv
source venv/bin/activate

tput setaf 2; echo "installing requirements"
tput sgr 0;
pip install -r requirements.txt

tput setaf 2; echo "setting up Gunicorn"
tput sgr 0;
cat <<EOL | sudo tee /etc/systemd/system/${APPNAME}.service
[Unit]
Description=gunicorn ${APPNAME} daemon
After=network.target

[Service]
User=${USERNAME}
Group=${USERNAME}
WorkingDirectory=${PWD}
ExecStart=${PWD}/${APPNAME}/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:${PWD}/${APPNAME}/${APPNAME}.sock ${APPNAME}.wsgi:application

[Install]
WantedBy=multi-user.target
EOL

tput setaf 2; echo "setting up Nginx"
tput sgr 0;
cat <<EOL | sudo tee /etc/nginx/sites-available/${APPNAME}
server {
    listen 80;
    server_name ${IPADDRESS};

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root ${PWD}/${APPNAME};
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:${PWD}/${APPNAME}/${APPNAME}.sock;
    }
}
EOL

sudo ln -s /etc/nginx/sites-available/${APPNAME} /etc/nginx/sites-enabled
sudo rm /etc/nginx/sites-enabled/default

tput setaf 2; echo "setting up UFW permissions"
tput sgr 0;
sudo ufw allow 'Nginx Full'
sudo ufw allow 'ssh'

tput setaf 1; 
echo "
1. dont forget to turn on ufw
2. setup postgresql
"
tput sgr 0;
