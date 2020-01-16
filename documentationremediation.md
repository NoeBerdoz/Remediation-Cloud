# Documentation
This documentation is written for Debian GNU/Linux 9.11 (stretch)  

Author : Berdoz Noé, Pereira Gabriel

You can check your Debian version with this command :

    $ lsb_release -a

## Set up your system
First of all make sure that your system is up to date
Run :
    
    $ apt update
    
And
    
    $ apt upgrade

Then, it's highly recommended that you set up NTP daemon.

### Set up NTP
Install NTP :
    
    $ apt-get install ntp

Type this command to see servers you are syncing with.

    $ ntpq -p
    
If you get this problem

    No association ID's returned
    
Do the following and then try again :

    $ dpkg-reconfigure ntp
    
You should see something like this :

    $ ntpq -p
    
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
     ntp.pbx.org     xx.xxx.xxx.xxx   2 u    -   64    1   33.763  1799619   1.054
     xray.metadom.co xx.xxx.xxx.xxx   2 u    1   64    1   40.367  1799619   0.001
     hydrogen.cert.u xx.xxx.xxx.xxx   2 u    -   64    1   64.740  1799619   0.001
     mirror          .INIT.          16 u    -   64    0    0.000    0.000   0.001


## Increase Max # of File Descriptors
You can see the maximum number of file descriptors with this command :

    $ ulimit -n

If your console show less then 65535 do the following :
Go to /etc/security/limits.conf and add the following lines at the end of the file :
  
    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536
   
Then reboot your machine

## Optimize Network Kernel Parameters
Add these lines to your file at /etc/sysctl.conf

    net.core.somaxconn = 1024
    net.core.netdev_max_backlog = 5000
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.ipv4.tcp_wmem = 4096 12582912 16777216
    net.ipv4.tcp_rmem = 4096 12582912 16777216
    net.ipv4.tcp_max_syn_backlog = 8096
    net.ipv4.tcp_slow_start_after_idle = 0
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.ip_local_port_range = 10240 65535
    
 After that, type this command so the changes can take effect :
 
    $ sysctl -p
    
## Install curl
    
    $ sudo apt install curl

## Install Fluentd
    
    $ curl -L https://toolbelt.treasuredata.com/sh/install-debian-stretch-td-agent3.sh | sh

### Launch Daemon

    $ systemctl start td-agent.service
    $ systemctl status td-agent.service
    
## Install MongoDB

    $ curl https://www.mongodb.org/static/pgp/server-4.0.asc | sudo apt-key add -

Next we will create a source list for the MongoDB repo.

    $ nano /etc/apt/sources.list.d/mongodb-org-4.0.list

Paste the following in mongodb-org-4.0.list

    deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/4.0 main

Save and close the file, then update your package cache

    $ apt update

Install the mongodb-org package

    $ apt-get install mongodb-org

Enable mongod service and then start it

    $ systemctl enable mongod
    $ systemctl start mongod
    
Check the service status

    $ systemctl status mongod
    
You should see an output like this

    ● mongod.service - MongoDB Database Server
       Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
       Active: active (running) since Wed 2018-09-05 16:59:56 UTC; 3s ago
         Docs: https://docs.mongodb.org/manual
     Main PID: 4321 (mongod)
        Tasks: 26
       CGroup: /system.slice/mongod.service
               └─4321 /usr/bin/mongod --config /etc/mongod.conf 
               
               
## Install NGINX

    $ apt-get install nginx

Now enable it

    $ systemctl enable nginx
    
Check if the service NGINX is running

    $ systemctl status nginx
    
You can also check if the server is working by writing your ip machine on a local browser

Make this command to check your ip address on the server :
    
    $ ip a
    
## NGINX configuration
First desactivate default configuration
    
    $ rm /etc/nginx/sites-enabled/default
    
Edit the following in /etc/nginx/sites-available/cldremediation.com

    $ vim /etc/nginx/sites-available/cldremediation.com

Copy and past this :
    
    server {
            listen 80;
            server_name cldremediation.com www.cldremediation.com;
            root /var/www/cldremediation.com;
            index index.php;
    
            location / {
                    try_files $uri $uri/ =404;
            }
    
            location ~ \.php$ {
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
                include snippets/fastcgi-php.conf;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            }
    
            location ~ /\.ht {
                    deny all;
            }
    }

Save and close the file and then make symbolic link to enable the server block

    sudo ln -s /etc/nginx/sites-available/cldremediation.com /etc/nginx/sites-enabled/cldremediation.com
    
## Install PHP and PHP7.4-FPM

### Add repository
Download GPG key

    $ apt -y install lsb-release apt-transport-https ca-certificates 
    $ wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    
Then add the repository

    $ echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list

### Install PHP7.4

    $ apt update
    $ apt -y install php7.4
    
### Disable Apache

    $ systemctl disable --now apache2

### Install fpm extension

    $ apt-get install php7.4-fpm

Check if FPM is running

    $ systemctl status php7.4-fpm
    
## Check if PHP is working well
This is optional

Edit the index file in /var/www/cldremediation.com/

    $ vim var/www/cldremediation.com/index.php

Copy and past this inside

    <html>
     <head>
      <title>Test PHP</title>
     </head>
     <body>
     <?php echo '<p>Hello World</p>'; ?>
     </body>
    </html>

### Change the host file in your system 

Now, if you change the host file in your system, you can access the index.php by
entering cldremediation.com on your browser

For example, in a Windows machine, go to C:\Windows\System32\drivers\etc
Open the host file as an administrator and add your Debian ip machine with the url of your website at the bottom of the file.
It should look like this:

    # localhost name resolution is handled within DNS itself.
    #	127.0.0.1       localhost
    #	::1             localhost
    192.168.XXX.XXX     cldremediation.com
    
Replace the XXX with your Debian machine's IP address.

Now go to cldremediation.com with a browser.
You should see a page showing your index.php with the 'Hello World'.

##Install Fluentd Mongo plugin

First install make and gcc
    
    $ apt-get install make gcc
    
After that execute this command 
    
    /usr/sbin/td-agent-gem install fluent-plugin-mongo
    
## Manage permissions
FLuentd user doesn't have permission in nginx logs files.  
Add the td-agent user to the adm group.

    $ usermod -a -G adm td-agent

Then restart the td-agent.service

    $ systemctl restart td-agent.service

Now change the permission of the php7.4-fpm.log file in
/var/log/
    
    $chown td-agent:td-agent php7.4-fpm.log

##Store NGINX and PHP7.4-FPM logs in Mongo

To edit fluentD config open this file 

    $  vi /etc/td-agent/td-agent.conf
    
Then remove all the default config and replace it by 

    <source>
      @type tail
      path /var/log/nginx/access.log
      pos_file /var/log/td-agent/nginx-access.log.pos 
      tag nginx.access
      format nginx 
    </source>
    
    <source>
      @type tail
      path /var/log/nginx/error.log
      pos_file /var/log/td-agent/nginx-error.log.pos 
      tag nginx.error
      format nginx 
    </source>
    
    <match nginx.**>
      @type mongo
      database fluentdLogs
      collection nginx
    </match>
    
    <source>
      @type tail
      path /var/log/php7.4-fpm.log
      pos_file /var/log/td.agent/php7.4\[(?<logtime>[^\]]*)\] (?<level>[A-Z]*): (?<message>.*)-fpm-logs.log.pos
      tag php7.4-fpm
      format /^\[(?<logtime>[^\]]*)\] (?<level>[A-Z]*): (?<message>.*)$/
    </source>
    
    <match php7.4-fpm.**>
      @type mongo
      database fluentdLogs
      collection phpFpm
    </match>
    
Now, restart the services

    $ systemctl restart nginx php7.4-fpm td-agent
    
Then check that their status are active

    $ systemctl status nginx php7.4-fpm td-agent    

## Check the data in Mongo
First, make sure that there is logs in Nginx, try to open the default webpage or NGINX.

Then open the Mongo shell.

    $ mongo
 
Then check that the database exist with this command:

    $ show dbs

If you find fluentdLogs in the list then do
    
    $ use fluentdLogs
    
And then check that logs are in mongo with this command

    $ db.nginx.find()
    
    $ db.phpFpm.find()


You should find the logs.
