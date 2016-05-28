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
```
vim /etc/httpd/conf.d/main.conf
```
