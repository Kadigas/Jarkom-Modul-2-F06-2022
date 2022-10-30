# Jarkom-Modul-2-F06-2022

(1)
OSTANIA CONFIG:
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.202.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.202.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.202.3.1
	netmask 255.255.255.0
```

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.202.0.0/16
cat /etc/resolv.conf
```

SSS CONFIG:
```
auto eth0
iface eth0 inet static
	address 192.202.1.2
	netmask 255.255.255.0
	gateway 192.202.1.1
```

Garden CONFIG:
```
auto eth0
iface eth0 inet static
	address 192.202.1.3
	netmask 255.255.255.0
	gateway 192.202.1.1
```

Berlint CONFIG:
```
auto eth0
iface eth0 inet static
	address 192.202.2.2
	netmask 255.255.255.0
	gateway 192.202.2.1
```

Eden CONFIG:
```
auto eth0
iface eth0 inet static
	address 192.202.2.3
	netmask 255.255.255.0
	gateway 192.202.2.1
```

WISE CONFIG:
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 192.202.3.2
	netmask 255.255.255.0
	gateway 192.202.3.1
```

ALL
```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf
ping google.com
```

(2)
WISE & BERLINT
```bash
apt-get update
apt-get install bind9 -y
```

WISE
```bash
nano /etc/bind/named.conf.local :
zone "wise.f06.com" {
    type master;
    file "/etc/bind/wise/wise.f06.com";
};
```
```bash
mkdir /etc/bind/wise
cp /etc/bind/db.local /etc/bind/wise/wise.f06.com
```
```bash
nano /etc/bind/wise/wise.f06.com
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.f06.com. root.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.f06.com.
@       IN      A       192.202.3.2     ; IP WISE
www     IN      CNAME   wise.f06.com.
@       IN      AAAA    ::1
```

```bash
service bind9 restart
```

SSS & Garden
```bash
nano /etc/resolv.conf
```
```
nameserver 192.202.3.2
```

(3)
WISE
```bash
nano /etc/bind/wise/wise.f06.com
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.f06.com. root.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.f06.com.
@               IN      A       192.202.2.3     ; IP Eden
www             IN      CNAME   wise.f06.com.
eden            IN      A       192.202.2.3     ; IP Eden
www.eden        IN      CNAME   eden.wise.f06.com.
@               IN      AAAA    ::1
```
```bash
service restart bind9
```

(4)
WISE
```bash
nano /etc/bind/named.conf.local
```
```
zone "wise.f06.com" {
    type master;
    file "/etc/bind/wise/wise.f06.com";
};

zone "2.202.192.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/2.202.192.in-addr.arpa";
};
```
```bash
cp /etc/bind/db.local /etc/bind/wise/3.202.192.in-addr.arpa
nano /etc/bind/wise/3.202.192.in-addr.arpa
```
```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.f06.com. root.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.202.192.in-addr.arpa. IN      NS      wise.f06.com.
3                       IN      PTR     wise.f06.com.
```
```bash
service bind9 restart
```
CLIENT
```bash
apt-get update
apt-get install dnsutils
```
```bash
host -t PTR 192.202.3.2
```

(5)
WISE
```bash
nano /etc/bind/named.conf.local
```
```
zone "wise.f06.com" {
    type master;
    notify yes;
    also-notify { 192.202.2.2; }; // IP Berlint
    allow-transfer { 192.202.2.2; }; // IP Berlint
    file "/etc/bind/wise/wise.f06.com";
};
```
```bash
service bind9 restart
```

Berlint
```bash
nano /etc/bind/named.conf.local
```
```
zone "wise.f06.com" {
    type slave;
    masters { 192.202.3.2; };
    file "/var/lib/bind/wise.f06.com";
};
```
```bash
service bind9 restart
```
WISE
```bash
service bind9 stop
```
CLIENT
```bash
nano /etc/resolv.conf
```
```
nameserver 192.202.3.2
nameserver 192.202.2.2
```
```bash
ping wise.f06.com
```

(6)
WISE
```bash
nano /etc/bind/wise/wise.f06.com
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.f06.com. root.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.f06.com.
@               IN      A       192.202.2.2     ; IP WISE
www             IN      CNAME   wise.f06.com.
eden            IN      A       192.202.2.3     ; IP Eden
www.eden        IN      CNAME   eden.wise.f06.com.
ns1             IN      A       192.202.2.2     ; IP Berlint
operation       IN      NS      ns1
@               IN      AAAA    ::1
```
```bash
nano /etc/bind/named.conf.options
```
```bash
allow-query{any;};
```

TAMBAHAN: FORWARDERS -> WISE
```bash
nano /etc/bind/named.conf.options
```
```
forwarders {
    192.168.122.1;
};
```
```bash
service bind9 restart
```

Berlint
```bash
nano /etc/bind/named.conf.options
```
```
allow-query{any;};
```
```bash
nano /etc/bind/named.conf.local
```
```
zone "operation.wise.f06.com" {
    type master;
    file "/etc/bind/operation/operation.wise.f06.com";
};
```
```bash
mkdir /etc/bind/operation 
nano /etc/bind/operation/operation.wise.f06.com
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.f06.com. root.operation.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      operation.wise.f06.com.
@       IN      A       192.202.2.3     ; IP Eden
www     IN      CNAME   operation.wise.f06.com.
```
```bash
service bind9 restart
```
(7)
Berlint
```bash
nano /etc/bind/operation/operation.wise.f06.com
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.f06.com. root.operation.wise.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.f06.com.
@               IN      A       192.202.2.3     ; IP Eden
www             IN      CNAME   operation.wise.f06.com.
strix           IN      A       192.202.2.3     ; IP Eden
www.strix       IN      CNAME   strix.operation.wise.f06.com.
```

(8)
Client
```bash
apt-get update
apt-get install lynx
```

Eden
```bash
apt-get update
apt-get install apache2
service apache2 start
apt-get install php
apt-get install libapache2-mod-php7.0
```
```bash
nano /etc/apache2/sites-available/000-default.conf
```
```bash
/var/www/wise.f06.com
```
```bash
service apache2 restart
cd /var/www/
mkdir /var/www/wise.f06.com
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/wise.f06.com.conf
nano /etc/apache2/sites-available/wise.f06.com.conf
```
```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/wise.f06.com
ServerName wise.f06.com
ServerAlias www.wise.f06.com
```
```bash
a2ensite wise.f06.com.conf
```
```bash
wget https://raw.github.com/theresianwg/pratikum-jarkom-modul-2/main/eden.wise.zip
wget https://raw.github.com/theresianwg/pratikum-jarkom-modul-2/main/strix.operation.wise.zip
wget https://raw.github.com/theresianwg/pratikum-jarkom-modul-2/main/wise.zip

unzip ~/wise.zip
cp -r ~/wise/. /var/www/wise.f06.com/
service apache2 restart
```

Client
```bash
lynx wise.f06.com
```
# Soal 10
### Setelah itu, pada subdomain `www.eden.wise.yyy.com` , Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada `/var/www/eden.wise.yyy.com`

Pertama pada Server Eden, mengkonfigurasi file menggunakan 
```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/eden.wise.f06.com.conf
nano /etc/apache2/sites-available/eden.wise.f06.com.conf
```
dan juga
```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/eden.wise.f06.com
ServerName eden.wise.f06.com
ServerAlias www.eden.wise.f06.com
```
lalu membuat documentroot pada `/var/www/eden.wise.f06.com` , dilanjutkan mengaktifkan virtualhost menggunakan a2ensite
```
a2ensite eden.wise.f06.com.conf
service apache2 reload
```
melakukan copy content ke documentroot menggunakan 
```
unzip ~/eden.wise.zip
cp ~/eden.wise/. /var/www/eden.wise.f06.com/
service apache2 restart
```
### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan ```lynx eden.wise.f06.com```


# Soal 11
### Akan tetapi, pada folder `/public`, Loid ingin hanya dapat melakukan directory listing saja

Pertama pada Server Eden, mengkonfigurasi file dapat menggunakan `nano /etc/apache2/sites-available/eden.wise.f06.com.conf`
Selanjutnya, menambahkan Options + Indeks pada direktori yang ingin di list
```
<Directory /var/www/eden.wise.f06.com/public>
    Options +Indexes
</Directory>
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx eden.wise.f06.com/public`

# Soal 12
### Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder `/error` untuk mengganti error kode pada apache 

Pertama pada Server Eden, mengkonfigurasi file dapat menggunakan `nano /etc/apache2/sites-available/eden.wise.f06.com.conf`
Dilanjutkan, dengan menambahkan konfigurasi `ErrorDocument` pada setiap error yang mengarah pada file `/error/404.html`
```
<Directory /var/www/eden.wise.f06.com>
    ErrorDocument 404 /error/404.html
</Directory>
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx eden.wise.f06.com/test`

# Soal 13
### Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.eden.wise.yyy.com/public/js` menjadi `www.eden.wise.yyy.com/js`

Pertama pada Server Eden, mengkonfigurasi file dapat menggunakan `nano /etc/apache2/sites-available/eden.wise.f06.com.conf`
Lalu untuk menambahkan konfigurasi Alias, menggunakan
```
Alias "/js" "/var/www/eden.wise.f06.com/public/js"
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx eden.wise.f06.com/js`

# Soal 14
### Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port `15000` dan port `15500` 

Pertama pada Server Eden, mengkonfigurasi file menggunakan
```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/strix.operation.wise.f06.com.conf
nano /etc/apache2/sites-available/strix.operation.wise.f06.com.conf
```
lalu menambahkan VirtualHost baru pada port `15000` dan port `15500` 
```
<VirtualHost *:15000 *:15500>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.f06.com
        ServerName strix.operation.wise.f06.com
        ServerAlias www.strix.operation.eden.wise.f06.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Dilanjutkan, mengkonfigurasi file menggunakan `nano /etc/apache2/ports.conf `untuk menambahkan listen port `15000` dan port `15500` 
```
Listen 1500
Listen 15500
```
selanjutnya melakukan
```
a2ensite strix.operation.wise.f06.com.conf
service apache2 reload
```
lalu melakukan uznip, menggunakan
```
unzip ~/strix.operation.wise.zip
cp -r ~/strix.operation.wise/. /var/www/strix.operation.wise.f06.com/
```
dan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan
```
lynx strix.operation.wise.f06.com:15000
lynx strix.operation.wise.f06.com:15500
```

# Soal 15
###  Dengan autentikasi username Twilight dan password opStrix dan file di `/var/www/strix.operation.wise.yyy`

Pertama pada Server Eden, menjalankan command `htpasswd -nb Twilight opStrix > /etc/apache2/.htpasswd`
lalu, mengkonfiogurasi file dengan `nano /etc/apache2/sites-available/strix.operation.wise.f06.com.conf`. 
Dan juga pada `/var/www/strix.operation.wise.f06.com`
```
<Directory /var/www/strix.operation.wise.f06.com>
    AuthType Basic
    AuthName "Restricted Files"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx strix.operation.wise.f06.com:15000`

# Soal 16
### Dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke `www.wise.yyy.com`

Pertama pada Server Eden, mengkonfigurasi file menggunakan `nano /var/www/html/.htaccess` lalu, dapat menambahkan
```
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^192\.202\.2\.3$
RewriteRule ^(.*)$ http://wise.f06.com/$1 [L,R=301]
```
Dilanjutkan dengan `nano /etc/apache2/sites-available/000-default.conf` dan menambahkan
```
<Directory /var/www/html>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx 192.202.2.3`


# Soal 17
### Karena website `www.eden.wise.yyy.com` semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian! 

Pertama pada Server Eden, mengkonfigurasi dengan `nano /var/www/eden.wise.f06.com/public/images/.htaccess`, lalu menambahkan
```
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)eden(.*)
RewriteRule eden http://eden.wise.f06.com/public/images/eden.png$1 [L,R=301]
```
Selanjutnya, mengkonfigurasi file menggunakan `nano /etc/apache2/sites-available/eden.wise.f06.com.conf` dan
```<Directory /var/www/eden.wise.f06.com/public/images>
    Options +FollowSymLinks -Multiviews +Indexes
    AllowOverride All
</Directory>
```
Lalu melakukan `service apache2 restart`

### Testing 
Pada client yaitu SSS atau Garden kita dapat melakukan testing menggunakan `lynx www.eden.wise.f06.com/public/images/eden-student.png`

