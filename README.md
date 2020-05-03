# Documentation, Store Nginx & PHP-FPM logs in a mongodb Database with Fluentd in a Linux machine

This documentation is written for Debian GNU/Linux 9.11 (stretch)  

Author : Berdoz Noé, Pereira Gabriel, Bergmann Florian, Maître Nicolas

You can check your Debian version with this command :

    $ lsb_release -a

## Set up your system
First of all make sure that your system is up to date
Run : 

    (Root) $ apt update
    (Root) $ apt upgrade

Then, it's highly recommended that you set up NTP daemon.

### Install sudo

    (Root) $ apt-get install sudo


### Install VIM

    (Root) $ apt-get install vim

### Set up NTP
Install NTP:

    (Root) $ apt-get install ntp

Type this command to see servers you are syncing with.

    $ ntpq -p
    
If you get this problem : **No association ID's returned**
    
-> do the following and then try again :

    (Root) $ dpkg-reconfigure ntp
    
You should see a array with this structure :

    $ ntpq -p
    
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
     ntp.pbx.org     xx.xxx.xxx.xxx   2 u    -   64    1   33.763  1799619   1.054
     xray.metadom.co xx.xxx.xxx.xxx   2 u    1   64    1   40.367  1799619   0.001
     hydrogen.cert.u xx.xxx.xxx.xxx   2 u    -   64    1   64.740  1799619   0.001
     mirror          .INIT.          16 u    -   64    0    0.000    0.000   0.001


### Increase Max # of File Descriptors
You can see the maximum number of file descriptors with this command :

    $ ulimit -n

If the number is less than `65535` do the following :
Go to `/etc/security/limits.conf` and add this config at the end of the file :
    
    (Root) $ echo " 
    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536" >> /etc/security/limits.conf
   
Then reboot your shell

### Optimize Network Kernel Parameters
Add this config to `/etc/sysctl.conf`

    (Root) $ echo "net.core.somaxconn = 1024
    net.core.netdev_max_backlog = 5000
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.ipv4.tcp_wmem = 4096 12582912 16777216
    net.ipv4.tcp_rmem = 4096 12582912 16777216
    net.ipv4.tcp_max_syn_backlog = 8096
    net.ipv4.tcp_slow_start_after_idle = 0
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.ip_local_port_range = 10240 65535" >> /etc/sysctl.conf
    
 After that, type this command so the changes can take effect :
 
    (Root) $ sysctl -p
    
### Install curl
    
    (Root) $ apt install curl

## Install Fluentd
    
    (Root/Automatic sudo) $ curl -L https://toolbelt.treasuredata.com/sh/install-debian-stretch-td-agent3.sh | sh

### Launch Daemon

    (Root/Auto root) $ systemctl start td-agent.service

    $ systemctl status td-agent.service
    
## Install MongoDB

    (Root) $ curl https://www.mongodb.org/static/pgp/server-4.0.asc | apt-key add -

Next we will create a source list for the MongoDB repo.  
Add conf in `/etc/apt/sources.list.d/mongodb-org-4.0.list`

    (Root) $ echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/4.0 main" >> /etc/apt/sources.list.d/mongodb-org-4.0.list

Save and close the file, then update your package cache

    (Root) $ apt update

Install the mongodb-org package

    (Root) $ apt-get install mongodb-org

Enable mongod service and then start it

    (Root/Auto root) $ systemctl enable mongod
    (Root/Auto root) $ systemctl start mongod
    
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

    (Root) $ apt-get install nginx

Now enable it

    (Root/Auto root) $ systemctl enable nginx
    
Check if the service NGINX is running

    $ systemctl status nginx
    
You can also check if the server is working by accessing your machine ip in a local web browser

Use this command to check your ip address on the server :
    
    $ ip a
    
### NGINX configuration
First delete the default configuration
    
    (Root) $ rm /etc/nginx/sites-enabled/default
    
Create the config for the "cldremediation" website by creating the `/etc/nginx/sites-available/cldremediation.com` file

    (Root) $  vim /etc/nginx/sites-available/cldremediation.com
    
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

Make symbolic link to the config to enable the server block

    (Root) $ ln -s /etc/nginx/sites-available/cldremediation.com /etc/nginx/sites-enabled/cldremediation.com
    
## Install PHP and PHP7.4-FPM

### Add repository
Download GPG key

    (Root) $ apt -y install lsb-release apt-transport-https ca-certificates 
    (Root) $ wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    
Then add the repository

    (Root) $ echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

### Install PHP7.4

    (Root) $ apt update
    (Root) $ apt -y install php7.4
    
### Remove Apache

    (Root) $ apt remove apache2

### Install fpm extension

    (Root) $ apt-get install php7.4-fpm

Check if FPM is running

    $ systemctl status php7.4-fpm
    
    
## Check if PHP is working well (optional)

Create the directory and index file of the website in `/var/www/`

    (Root) $ mkdir /var/www/cldremediation.com/
    (Root) $ touch /var/www/cldremediation.com/index.php

Insert the following php code inside `/var/www/cldremediation.com/index.php` to test out the web server

    (Root) $ vim /var/www/cldremediation.com/index.php

```php
<html>
    <head>
        <title>Test PHP</title>
    </head>
    <body>
        <?php echo '<p>Hello World</p>'; ?>
    </body>
</html>
```
copy-pasteable command version: 

    (Root) $ echo "<html><head><title>Test PHP</title></head><body><?php echo '<p>Hello World</p>'; ?></body></html>" > /var/www/cldremediation.com/index.php
    
### Restart Nginx and Php-fpm

    $ systemctl restart nginx php7.4-fpm

### Change the host file in your system (optionnal)

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


## Install Fluentd Mongo plugin

First install make and gcc
    
    (Root) $ apt-get install make gcc
    
After that execute this command 
    
    (Root) $ td-agent-gem install fluent-plugin-mongo
    
## Manage permissions
At first **restart the server**  

FLuentd user doesn't have permission in nginx logs files.  
Add the td-agent user to the adm group.

    (Root) $ usermod -a -G adm td-agent

Then restart the td-agent.service

    (Root/Auto root) $ systemctl restart td-agent.service

Now change the permission of the php7.4-fpm.log file in
/var/log/
    
    (Root) $ chown td-agent:td-agent /var/log/php7.4-fpm.log

## Store NGINX and PHP7.4-FPM logs in Mongo

To edit fluentD config open this file 

    (Root) $ vim /etc/td-agent/td-agent.conf

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
        pos_file /var/log/td-agent/php7.4-fpm-logs.log.pos
        tag php7.4-fpm
        format /^\[(?<logtime>[^\]]*)\] (?<level>[A-Z]*): (?<message>.*)$/
    </source>
    
    <match php7.4-fpm.**>
        @type mongo
        database fluentdLogs
        collection phpFpm
    </match>  

Restart your server

    (Root) $ shutdown -r now

Then, restart the services

    (Root) $ service mongod start
    (Root) $ systemctl restart nginx php7.4-fpm td-agent
    
And then, check that their status are active

    $ service mongod status
    $ systemctl status nginx php7.4-fpm td-agent

## Check the data in Mongo

First restart services to create the database

    (Root) $ systemctl restart nginx php7.4-fpm td-agent

After, make sure that there is logs in NGINX, try to open the default webpage.
Go to the ip address of your server with a browser to generate log.

Then open the Mongo shell.

    $ mongo
 
Then check that the database exists with this command:

    mongo > show dbs

If database **fluentdLogs** does not appear in the list, quit mongo and restart services

    (Root) $ systemctl restart nginx php7.4-fpm td-agent

Select database **fluentdLogs** to use it
    
    mongo > use fluentdLogs

And then check that logs are in mongo with this command

    mongo > db.nginx.find()
    
    mongo > db.phpFpm.find()

You should find the logs.
