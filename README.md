# Jarkom-Modul-2-F06-2022

(1)
OSTANIA
CONFIG:
```
auto eth3
iface eth3 inet static
	address 192.202.3.1
	netmask 255.255.255.0
```

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.202.0.0/16
```

WISE
CONFIG:
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

```service bind9 restart```

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

unzip -j ~/wise.zip -d /var/www/wise.f06.com/
service apache2 restart
```

Client
```bash
lynx wise.f06.com
```
