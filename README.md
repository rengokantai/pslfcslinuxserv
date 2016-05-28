#### pslfcslinuxserv
#####1
######3 systemd
s1:
```
yum install -y net-tools bash-completion vim-enhanced
```
```
ps -fp 1   //show pid =1
```
```
systemctl status/mask/unmask sshd
```
#####2
######6
```
yum install -y bind bind-utils
systemctl enable named
systemctl start named
ipatables -L (package:bind service:named config:caching port:53))
netstat -ltn
dig www.github.com @127.0.0.1
```
######7 DNS forwarding
```
vim /etc/named.conf
```
edit
```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { none; };
```
then
```
named-checkconf -v
systemctl restart named
```

add forwarding
```
        session-keyfile "/run/named/session.key";
        forwarders{8.8.8.8; 8.8.4.4;};
        forward only;
};
```
add print-severity
```
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
                print-severity yes;
```
rerun (clear data)
```
> /var/named/data/named.run
```
check
```
cat !$
```
```
vim /etc/named.rfc1912.zones
```
list all zone names
```
ls /var/named
```
######9 conf a BIND DNS
```
zone "a.vm."{
        type master;
        file "db.example";
        allow-update { none; };
};
zone "." IN {
        type hint;
        file "named.ca";
};
```
######10 creating a dns zone
```
cd /var/named
cp named.empty db.example
chgrp named db.example
```
then
```
vim db.example
```
edit
```
$TTL 3H
$ORIGIN a.vm.
a.vm.   IN SOA server1.a.vm.    root.a.vm. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
a.vm.   NS      server1.a.vm.
server1 A       10.128.25.172
        AAAA    ::1

```
check
```
named-checkzone a.vm db.example
cat /var/named/data/named.run
```
######11 using dns tools
```
dig server1.a.vm @127.0.0.1
```
```
yum install python-dns -y
```
```
python /usr/share/doc/python-dns-1.12.0/examples/mx.py
cp /usr/share/doc/python-dns-1.12.0/examples/mx.py /usr/share/doc/python-dns-1.12.0/examples/our.py
vim !$
```
edit
```
#!/usr/bin/env python

import dns.resolver
r = dns.resolver.Resolver()
r.nameservers = ['127.0.0.1']

answers = r.query('a.vm', 'NS')
for rdata in answers:
    print rdata
```
#####FTP
######13
config DNS client on server 2  
install vsftpd on server 1
config vsftpd on server 1
create ftp yum repo on server 1
use ftp repo from server 2  
s2:
```
cat /etc/resolv.conf
```
connect s2 to s1.  
s2:
```
```


#####install apache
######50
```
yum install httpd httpd-manual
systemctl start httpd.service
pgrep httpd
ps -fp $(pgrep httpd)
yum install -y epel-release w3m
w3m http://127.0.0.1
aapchectl configtest
cd /etc/httpd
tree
```
config
```
vim /etc/httpd/conf/httpd.conf
```
edit
```
ServerName "server1.a.vm"
```
create a virtual host
```
vim /etc/httpd/conf.d/main.conf
```
edit
```
<VirtualHost *:80>
ServerName "server1.a.vm"
DocumentRoot "/var/www/html"
</VirtualHost>
```
test
```
!apa
```
create a new
```
cp main.conf moodle.conf
```
edit
```
<VirtualHost *:80>
ServerName "moodle.a.vm"
DocumentRoot "/var/www/moodle"
</VirtualHost>
```
then create dir
```
mkdir /var/www/moodle
```
test,add content
```
echo "testke" >!$/index.html
!sys
```
test,visit
```
w3m moodle.a.vm
```
if failed,then
```
echo "10.128.25.172 moodle.a.vm" >> /etc/hosts
```
######install apache config log files
```
pwd -P
```
```
vim /etc/httpd/conf.d/main.conf
```
edit
```
<VirtualHost *:80>
ServerName "server1.a.vm"

DocumentRoot "/var/www/html"
ErrorLog "logs/server1_error"
CustomLog "logs/server1_access" common
<Directory "/var/www/html">
AllowOverride none

</Directory>
<Location /status>
SetHandler server-status
</Location>
</VirtualHost>
```
then
```
cd /etc/httpd
grep LogFormat conf/httpd.conf    //get log format
```
```
w3m 127.0.0.1/manual
```
restrict access
```
vim /etc/httpd/conf.d/main.conf
```
edit
```
<VirtualHost *:80>
ServerName "server1.a.vm"

DocumentRoot "/var/www/html"
ErrorLog "logs/server1_error"
CustomLog "logs/server1_access" common
<Directory "/var/www/html">
AllowOverride none

</Directory>
<Location /status>
SetHandler server-status
Require ip 127.0.0.1 10.128.25.172  //other ip will be 403 error
</Location>
</VirtualHost>
```
restart
```
!sys
```
show error log
```
vim /etc/httpd/logs/server1_error
```
######restrict access by username
```
vim /etc/httpd/conf.d/moodle.conf
```
edit
```
<VirtualHost *:80>
ServerName "moodle.a.vm"
DocumentRoot "/var/www/moodle"
<Directory> "/var/www/moodle">
Require valid-user
AuthType Basic
AuthName "Type your pass"
AuthBasicProvider file
AuthUserFile "/etc/httpd/conf.d/moodle"
</Directory>
</VirtualHost>
```
create user
```
!sys
cd /etc/httpd/conf.d/
htpasswd -c moodle root
```
add another user
```
htpasswd moodle user2
```
access:
```
w3m moodle.a.vm
```
######55 enabling ssl
```
yum install -y mod_ssl
```
```
cd /etc/httpd/conf.d/ssl.conf
```
backup
```
cp /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.template
```
gen ssh
```
openssl req -x509 -new -nodes -keyout moodle.key -out moodle.crt
```
set common name= moodle.a.vm,others blank
```
cd /etc/httpd/conf.d/
chmod 400 moodle.key moodle.crt
```
then
```
vim moodle.conf
```
edit
```
<VirtualHost *:80>
ServerName "moodle.a.vm"
DocumentRoot "/var/www/moodle"
<Directory "/var/www/moodle">
Require valid-user
AuthType Basic
AuthName "Private Access"
AuthBasicProvider file
AuthUserFile "/etc/httpd/conf.d/moodle"
</Directory>
</VirtualHost>
Listen 443  //delete this line to avoid error
<VirtualHost *:443>
ServerName "moodle.a.vm"
DocumentRoot "/var/www/moodle"
SSLEngine on
SSLCertificateKeyFile "conf.d/moodle.key"   //added
SSLCertificateFile "conf.d/moodle.crt"    //added
<Directory "/var/www/moodle">
Require valid-user
AuthType Basic
AuthName "Private Access"
AuthBasicProvider file
AuthUserFile "/etc/httpd/conf.d/moodle"
</Directory>
</VirtualHost>
```
then
```
apachectl configtest
```
######56 squid server
```
yum install -y squid
```
```
vim /etc/squid/squid.conf
```
edit
```
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network  //for this part only keep this line
```
start service
```
systemctl start squid.service
```

in s2:
```
export http_proxy=http://server1.a.vm:3128
```
connect to s1:
```
w3m server1.a.vm
```

in s1
```
tail -n1 /var/log/httpd/server1_access
```
