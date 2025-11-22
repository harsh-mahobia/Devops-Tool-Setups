# We're setting up 2 server in aws. 1st for nagios dashboard. 2nd is server that is need to be monitored.

## Nagios Server Commands

```
sudo apt update
sudo apt install -y \
build-essential \
gawk \
libgd-dev \
libssl-dev \
apache2 \
php \
libapache2-mod-php \
php-gd \
wget \
unzip \
daemon \
libmcrypt-dev \
libkrb5-dev \
libwrap0-dev
```

```
cd /tmp
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.5.2.tar.gz
tar -zxvf nagios-4.5.2.tar.gz
cd nagios-4.5.2
./configure --with-httpd-conf=/etc/apache2/sites-enabled
```

```

make all

```

```
sudo make install
sudo make install-groups-users
sudo usermod -aG nagios www-data
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
sudo make install-webconf
```

* Download Plugins
```
  sudo apt install -y nagios-plugins nagios-nrpe-plugin
```

* Set admin password:
```
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

* NOW check this directory:
``` ls /usr/local/nagios ```

make sure you've exposed the 80 port from security groups
* Now login with http:<public-ip>/nagios


## Now Commands for Backend Server 
```
sudo apt update
sudo apt install -y nagios-nrpe-server nagios-plugins
```
* Check NRPE installed:
```
sudo systemctl status nagios-nrpe-server
```

* Edit NRPE config:
```
sudo nano /etc/nagios/nrpe.cfg
```

* Find this line:
```
allowed_hosts=127.0.0.1
```

* Change it to include the Nagios EC2 private IP:

```allowed_hosts=127.0.0.1,<NAGIOS_SERVER_PRIVATE_IP>```


Example:
```allowed_hosts=127.0.0.1,172.31.8.124```

* Save → exit.

* Restart NRPE:
```
sudo systemctl restart nagios-nrpe-server
```
## Nagios Server Commands

* Check
```
/usr/lib/nagios/plugins/check_nrpe -H <backend_server_private_ip>
```

* Create Directory 
```
sudo mkdir -p /usr/local/nagios/etc/servers
```

* Create a backend.cfg file
```
sudo nano /usr/local/nagios/etc/servers/backend.cfg
```
* Add this content to file and replace <backend_private_ip>
```
define host {
    use                 linux-server
    host_name           backend-server
    alias               Node.js Backend EC2
    address             <BACKEND_PRIVATE_IP>
}

define service {
    use                 generic-service
    host_name           backend-server
    service_description PING
    check_command       check_ping!100,20%!500,60%
}

define service {
    use                 generic-service
    host_name           backend-server
    service_description NRPE Connection Test
    check_command       check_nrpe!check_load
}

define service {
    use                 generic-service
    host_name           backend-server
    service_description Node.js Port 90
    check_command       check_tcp!90
}
```
* Save → exit.
* Update Permission
```
sudo chown nagios:nagios /usr/local/nagios/etc/servers/backend.cfg
sudo chmod 644 /usr/local/nagios/etc/servers/backend.cfg
```
* Update nagios.conf
```
sudo nano /usr/local/nagios/etc/nagios.cfg
```
```
cfg_dir = /usr/local/nagios/etc/servers  
```
### Test
```
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
you must see 
```
Checked 2 hosts
Checked 11 services
```
* Restart nagios
 ``` 
sudo systemctl restart nagios
```

* Update the Nagios resource file
```
sudo nano /usr/local/nagios/etc/resource.cfg
```

* Find this line:
``` $USER1$=/usr/local/nagios/libexec```

* Change to:
``` $USER1$=/usr/lib/nagios/plugins```
* Restart it another time
```
sudo systemctl restart nagios
```

### Here we go...........
# 🎉🎉🎉🎉🎉
