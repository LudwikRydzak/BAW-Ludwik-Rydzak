## Notatki ##

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key_nginx.key -out cert_nginx.pem
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key_apache.key -out cert_apache.crt


#############


#Zadanie 2.

Celem zadania drugiego jest utworzenie certyfikatów dla użytkownika A i użytkownika B oraz
skonfigurowanie ścieżek /only-user-a, /only-user-b oraz /user-a-or-b tak aby tylko odpowiedni użytkownik miał 
do nich dostęp. Dodatkowo, podścieżka /info wyświetli informacje z certyfikatu.

## Tworzenie niezbędnych certyfikatów
utworzono nowy katalog na te wszystkie klucze i certyfikaty
```
mkdir ssl
```

```
sudo openssl req -newkey rsa:2048 -nodes -keyform PEM -keyout selfsigned-ca.key -x509 -days 3650    -outform PEM -out selfsigned-ca.crt

openssl genrsa -out selfsigned.key 2048

openssl req -new -key selfsigned.key -out selfsigned.csr

sudo openssl x509 -req -in selfsigned.csr -CA selfsigned-ca.crt -CAkey selfsigned-ca.key -set_serial 100 -days 365 -outform PEM -out selfsigned.crt
```
Signature ok
subject=C = PL, ST = Polska, L = Wroclaw, O = PWr, OU = cyberbezpieczenstwo, CN = bawapache.pl, emailAddress = ludwik@rydzak.com
Getting CA Private Key

Do serwerów przekazano nowo utworzony certyfikat
```
services:
  bawnginx:
    image: nginx:latest
    ports:
      - 5080:80
      - 5443:443
    volumes:
      - ./ssl/selfsigned.crt:/etc/nginx/ssl/bawnginx.pl.pem
      - ./ssl/selfsigned.key:/etc/nginx/ssl/bawnginx.pl.key
      - ./nginx_default.conf:/etc/nginx/conf.d/default.conf
      - ./bawnginx.pl/:/usr/share/nginx/bawnginx.pl
  bawapache:  
    image: httpd:latest
    ports:
      - 6080:80
      - 6443:443
    volumes:  
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf
      - ./bawapache.pl/:/var/www/bawapache.pl/
      - ./httpd_ssl.conf:/usr/local/apache2/conf/extra/httpd-ssl.conf
      - ./ssl/selfsigned.crt:/usr/local/apache2/conf/server.crt
      - ./ssl/selfsigned.key:/usr/local/apache2/conf/server.key
```

SSL działa z nowym sertyfikatem!

Dodano następujące linie do httpd-ssl.conf
```
SSLVerifyClient require
SSLVerifyDepth  10
SSLCACertificateFile "/usr/local/apache2/conf/selfsigned-ca.crt"
```
Utworzenie kluczy
openssl genrsa -out selfsigned-userA.key 2048
openssl genrsa -out selfsigned-userB.key 2048

Utworzenie certyfikatów
openssl req -new -key selfsigned-userA.key -out selfsigned-userA.csr
openssl req -new -key selfsigned-userB.key -out selfsigned-userB.csr

Podpisanie certyfikatu przez CA 

sudo openssl x509 -req -in selfsigned-userA.csr -CA selfsigned-ca.crt -CAkey selfsigned-ca.key -set_serial 101 -days 365 -outform PEM -out selfsigned-userA.crt
sudo openssl x509 -req -in selfsigned-userB.csr -CA selfsigned-ca.crt -CAkey selfsigned-ca.key -set_serial 102 -days 365 -outform PEM -out selfsigned-userB.crt

Zrobienie z certyfikatu i klucza zestawu p12
sudo openssl pkcs12 -export -inkey selfsigned-userA.key -in selfsigned-userA.crt -out selfsigned-userA.p12
sudo openssl pkcs12 -export -inkey selfsigned-userB.key -in selfsigned-userB.crt -out selfsigned-userB.p12


## konfiguracja Apache
https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html
W pliku konfiguracyjnym httpd-ssl.conf

```
SSLCACertificateFile "/usr/local/apache2/conf/selfsigned-ca.crt"


<Location /only-user-a/>
  SSLVerifyClient require
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} in {"userA"}
</Location>

<Location /only-user-b/>  
  SSLVerifyClient require
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} in {"userB"}
</Location>

<Location /user-a-or-b/>  
  SSLVerifyClient require 
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} in {"userA", "userB"}
</Location>

```



