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


#####email
######33 postfix mta
install postfix (typically already installed)
```
yum install postfix
```
```
alternatives --display mta
```
```
systemctl status postfix.service 
```
smtp:port25. isntall mailx
```
yum install mailx -y
```
write mail
```
mail root
```
then
```
Subject: test
content
.
EOT
```
check mail: /var/spool/mail/root
```
mail
```
press q enter to exit,if check log
```
tail /var/log/maillog
```
###### receiving SMTP
s1
```
postconf -d -n(show k=v)
```
```
postconf -n config_directory
```
```
ls /etc/postfix
```
config
```
vim /etc/postfix/main.cf
```
copy
```
cp /etc/postfix/main.cf /etc/postfix/main.cf.$(date +%F)
```
```
postconf -n inet_interfaces
postconf -n inet_protocols
```
set
```
postconf -e inet_interfaces=all
postconf -e inet_protocols=ipv4
systemctl restart postfix.service
```
s2:
```
mail root@server1.a.vm
```
s1:
```
vim /var/named/db.example
```
edit, add some server
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
server2 A       10.12.2.3
server3 A       10.1.2.3
a.vm.   MX      10   server1.a.vm.
        AAAA    ::1
```
check
```
dig -t MX a.vm
```
ping test
```
ping server2
```
check mydestnation
```
postconf mydestnation
```
set
```
postconf 'mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain'
```
check myorigin
```
postconf myorigin
```
```
postconf -e 'myorigin=$mydomain'
```
check
```
postfix check
systemctl restart postfix.service
```
######38 config smtp relay
s2:
```
cp /etc/postfix/main.cf /etc/postfix/main.cf.$(date +%F)
postconf -e inet_interfaces=all
postconf -e inet_protocols=ipv4
postconf -e relayhost=server1.a.vm
systemctl restart postfix.service
```
s3:
```
cp /etc/postfix/main.cf /etc/postfix/main.cf.$(date +%F)
postconf -e inet_interfaces=all
postconf -e inet_protocols=ipv4
postconf -e relayhost=server1.a.vm
systemctl restart postfix.service
yum install -y mailx
mail root@server2.a.vm
```
######39 dovecot
s1
```
yum install -y dovecot
```
config
```
cd /etc/dovecot
vim dovecot.conf
```
edit
```
protocols = imap pop3 lmtp
listen = *
```
then
```
cd conf.d
vim 10-auth.conf
```
edit
```
disable_plaintext_auth = no
auth_mechanisms = plain login
```
```
vim 10-mail.conf
```
edit
```
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_access_groups =mail
```
then
```
vim 10-master.conf
```
edit
```
unix_listener /var/spool/postfix/private/auth {
	mode=0666
	user=postfix
	group=postfix
}
```
then
```
vim 10-ssl.conf
```
edit
```
ssl=no
```
```
systenctl start dovecot
```
######40 mutt
```
yum install -y mutt
```
create conf file
```
cd 
vim .muttrc
```
edit
```
set spoolfile="imap://tux@server1.a.vm"
```
run
```
mutt
```
######41 config aliases
```
less /etc/aliases
vim /etc/aliases
```
edit
```
webmaster:tux
```
run
```
newaliases
```
test
```
mail webmaster@a.vm
```





#####printing service port 631.
#######44 cups
```
yum install -y cups
systemctl start cups
```
config
```
vim /etc/cups/cupsd.conf
```
edit
```
DefaultEncryption Never
Listen localhost:631
Listen 10.128.25.172:631
<Location />
Order allow,deny
allow localhost
allow 10.128.25.0/24
</Location>
```
then
```
systemctl restart cups
```
visit
```
10.128.25.172:631
```
######45
(to be added)


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
export http_proxy=http://server1.a.vm:3128    (test later)
```
connect to s1:
```
w3m server1.a.vm
```

in s1
```
tail -n1 /var/log/httpd/server1_access
```
#####PHP 
###### 60 php
```
tree /etc/httpd/conf.d
yum install -y php
```
edit conf
```
vim /etc/httpd/conf.d/php.conf
systemctl restart httpd
```
edit
```
upload_max_filesize
post_max_size
max_execution_time
```
show php config
```
php -i
vim /etc/php.ini
```
edit
```
upload_max_filesize = 20M
post_max_size = 24M  >=upload_max_size
max_execution_time = 300
```
create file
```
cd /var/www/html
vim info.php
```
edit
```
<?php phpinfo(); ?>
echo $_SERVER['HTTP_USER_AGENT'];
```
######61 application
```
<h1>Convert Temperature</h1>
<p>You will need to enter a temperature in Centigrade to be converted to Fahrenheit</p>
<?php
  if (isset($_GET['submit'])) {
    $c = $_GET['centigrade'];
    $f = (($c * 9)/5) + 32;
    echo "<p>$c C is <b>$f F</b></p>";
    }
?>
<form method="get" action="<?php echo $_SERVER['PHP_SELF']; ?>">
<input type="text" name="centigrade"></input>
<br>
<input type="submit" value="Convert" name="submit"></input>
</form>
```
#####MariaDB
######63
```
yum install -y php-mysql mariadb mariadb-server
systemctl enable mariadb
systemctl start mariadb
mysql_secure_installation  (remove anoy user,disallow remote login,y,y)
```
######66
sample php
```
<?php
	$user = 'user';
	$password = 'Password1';
	$host = 'localhost';
	$db = 'albums';
	$dbh = mysqli_connect($host,$user,$password,$db);
	echo "Connected to $host <br><br>";
	$query = "SELECT * FROM artists";
	$result = mysqli_query($dbh,$query);
	while ( $row = mysqli_fetch_array( $result,MYSQL_ASSOC ) ) {
		echo $row["artist_id"] . " " . $row["artist_name"]."<br>";
	}
?>
```
#####selinux
######69 install samba
```
yum install -y samba*
```
```
cd ~
mkdir -m 1777 /share
ls -ld /share
cp /etc/hosts /share
```
then edit smb.conf
```
vim /etc/samba/smb.conf
```
edit,add
```
[share]
path = /share
writable = yes
```
verify
```
testparm
```
start server
```
systemctl start nmb smb
systemctl enable nmb smb
```
add user
```
smbpasswd -a root
```
list client
```
smbclient -L //localhost
```
edit
```
smbclient //localhost/share Pass1  //quit toxexit
```

######70
```
getenforce
```
audit daemon
```
ausearch -m avc
```
```
ls -dZ /share/
```
######whatprovides
```
yum whatprovides */sbin/semanage
```
install
```
yum install policycoreutils-python
```
change context
```
semanage fcontext -a -t samba_share_t '/share(/.*)?'
restorecon -Rv /share
```
######boolean and samba

```
mkdir -m 1777 /accounts
```
then edit smb.conf
```
vim /etc/samba/smb.conf
```
add
```
[accts]
path = /accounts
writable = yes
```
then
```
setsebool -P samba_export_all_rw 1
systemctl restart smb
```
copy hosts
```
cp /etc/hosts /accounts/
smbclient //localhost/accts
```
######72 samba from selinux enforcement
```
yum install -y setools-console
```
seinfo
```
seinfo --permissive
```
add module
```
semanage permissive -a smbd_t
```
```
seinfo --permissive
```
