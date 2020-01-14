# Documentation
This documentation is written for Debian GNU/Linux 9.11 (stretch)  

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

## Launch Daemon

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
    
{{ WORK IN PROGRESS NOE : https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-debian-9}
https://docs.fluentd.org/output/mongo}