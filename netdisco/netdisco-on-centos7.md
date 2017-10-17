# Netdisco installation on Centos 7

## Base setup 
```
useradd -m -p x -s /bin/bash netdisco
yum -y install perl-core perl-DBD-Pg net-snmp-perl net-snmp-devel openssl-devel make automake gcc perl-Filesys-Notify-Simple
yum -y install postgresql-server

systemctl enable postgresql
postgresql-setup initdb
systemctl start postgresql
```

## Database setup
```
su - postgres
createuser -DRSP netdisco
somepassword

createdb -O netdisco netdisco```
```

## Installation
```
su - netdisco
curl -L https://cpanmin.us/ | perl - --notest --local-lib ~/perl5 App::Netdisco
[cut]

mkdir ~/bin
ln -s ~/perl5/bin/{localenv,netdisco-*} ~/bin/
```

## Configuration files
```
mkdir ~/environments
cp ~/perl5/lib/perl5/auto/share/dist/App-Netdisco/environments/deployment.yml ~/environments
chmod 600 ~/environments/deployment.yml
vim ~/environments/deployment.yml
```

## Deploy and set admin password
```
~/bin/netdisco-deploy
```

## systemd support

Backend unit file: `/etc/systemd/system/netdisco-backend.service`
```
[Unit]
Description=Netdisco Backend Service
AssertFileIsExecutable=/home/netdisco/bin/netdisco-backend
After=syslog.target network-online.target

[Service]
Type=forking
User=netdisco
Group=netdisco
ExecStart=/home/netdisco/bin/netdisco-backend start
ExecStop=/home/netdisco/bin/netdisco-backend stop
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

Frontend unit file: `/etc/systemd/system/netdisco-web.service`
```
[Unit]
Description=Netdisco Web Service
AssertFileIsExecutable=/home/netdisco/bin/netdisco-web
After=syslog.target network-online.target netdisco-backend.service

[Service]
Type=forking
User=netdisco
Group=netdisco
ExecStart=/home/netdisco/bin/netdisco-web start
ExecStop=/home/netdisco/bin/netdisco-web stop
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

Enable at startup:
```
systemctl daemon-reload
systemctl enable netdisco-backend.service
systemctl enable netdisco-web.service

systemctl start netdisco-backend.service
systemctl start netdisco-web.service
```

## Reference

- Installation: https://metacpan.org/pod/App::Netdisco
- Systemd: https://metacpan.org/pod/distribution/App-Netdisco/lib/App/Netdisco/Manual/Systemd.pod



