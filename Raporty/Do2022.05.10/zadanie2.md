
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

```
sudo openssl req -newkey rsa:2048 -nodes -keyform PEM -keyout nginx-ca.key -x509 -days 3650    -outform PEM -out nginx-ca.crt

openssl genrsa -out nginx.key 2048

openssl req -new -key nginx.key -out nginx.csr

sudo openssl x509 -req -in nginx.csr -CA nginx-ca.crt -CAkey nginx-ca.key -set_serial 110 -days 365 -outform PEM -out nginx.crt
```
Signature ok
subject=C = PL, ST = Polska, L = Wroclaw, O = PWr, OU = cyberbezpieczenstwo, CN = bawnginx.pl, emailAddress = ludwik@rydzak.pl
Getting CA Private Key



```bash
#Utworzenie kluczy
openssl genrsa -out selfsigned-userA.key 2048
openssl genrsa -out selfsigned-userB.key 2048
```
```bash
#Utworzenie certyfikatów
openssl req -new -key selfsigned-userA.key -out selfsigned-userA.csr
openssl req -new -key selfsigned-userB.key -out selfsigned-userB.csr
```
![](/Foty/Do2022.05.10/userA_selfsigned.png)
![](/Foty/Do2022.05.10/userB_selfsigned.png)
```bash
#Podpisanie certyfikatu przez CA apache

sudo openssl x509 -req -in selfsigned-userA.csr -CA selfsigned-ca.crt -CAkey selfsigned-ca.key -set_serial 101 -days 365 -outform PEM -out selfsigned-userA.crt
sudo openssl x509 -req -in selfsigned-userB.csr -CA selfsigned-ca.crt -CAkey selfsigned-ca.key -set_serial 102 -days 365 -outform PEM -out selfsigned-userB.crt
```
![](/Foty/Do2022.05.10/podpisanie_certyfikatow.png)

Pakiety do wczytania do przeglądarki:
```bash
#Zrobienie z certyfikatu i klucza zestawu p12 dla apache
sudo openssl pkcs12 -export -inkey selfsigned-userA.key -in selfsigned-userA.crt -out selfsigned-userA.p12
sudo openssl pkcs12 -export -inkey selfsigned-userB.key -in selfsigned-userB.crt -out selfsigned-userB.p12
```

To samo zrobiono na potrzeby nginx:
```
#Utworzenie kluczy
openssl genrsa -out nginx-userA.key 2048
openssl genrsa -out nginx-userB.key 2048

#Utworzenie certyfikatów
openssl req -new -key nginx-userA.key -out nginx-userA.csr
openssl req -new -key nginx-userB.key -out nginx-userB.csr

#Podpisanie certyfikatu przez CA nginx
sudo openssl x509 -req -in nginx-userA.csr -CA nginx-ca.crt -CAkey nginx-ca.key -set_serial 111 -days 365 -outform PEM -out nginx-userA.crt
sudo openssl x509 -req -in nginx-userB.csr -CA nginx-ca.crt -CAkey nginx-ca.key -set_serial 112 -days 365 -outform PEM -out nginx-userB.crt

#Zrobienie z certyfikatu i klucza zestawu p12 dla nginx
sudo openssl pkcs12 -export -inkey nginx-userA.key -in nginx-userA.crt -out nginx-userA.p12
sudo openssl pkcs12 -export -inkey nginx-userB.key -in nginx-userB.crt -out nginx-userB.p12
```


Do serwerów przekazano nowo utworzone certyfikaty
```                                                 
services:
  bawnginx:
    image: nginx:latest
    ports:
      - 5080:80
      - 5443:443
    volumes:
      - ./ssl/nginx.crt:/etc/nginx/ssl/bawnginx.pl.pem
      - ./ssl/nginx.key:/etc/nginx/ssl/bawnginx.pl.key
      - ./ssl/nginx-ca.crt:/etc/nginx/ssl/selfsigned-ca.pem
      - ./ssl/nginx-userA.crt:/etc/nginx/ssl/userA.crt
      - ./ssl/nginx-userB.crt:/etc/nginx/ssl/userB.crt
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
      - ./ssl/selfsigned-ca.crt:/usr/local/apache2/conf/selfsigned-ca.crt

```


## konfiguracja Apache
Konfigurację apache wykonano na podstawie opisu w [dokumentacji](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html).

W pliku konfiguracyjnym httpd-ssl.conf

```
SSLCACertificateFile "/usr/local/apache2/conf/selfsigned-ca.crt"


<Location /only-user-a/>
  SSLVerifyClient require
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} eq "userA"
</Location>

<Location /only-user-b/>  
  SSLVerifyClient require
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} eq "userB"
</Location>

<Location /user-a-or-b/>  
  SSLVerifyClient require 
  SSLVerifyDepth  10
  SSLRequire %{SSL_CLIENT_S_DN_CN} in {"userA", "userB"}
</Location>

```

Można było porównać inne parametry niż CN (common name) lub nawet wszystkie.
UWAGA: Zmieniono wersję TLS'a na 1.2 aby firefox ją obsługiwał.

## Konfiguracja Nginx

Dodano:
```
ssl_verify_client optional;
```
Oraz zmieniono wersję tls'a na 1.2, tak by firefox go obsługiwał. 


```
location /only-user-a {

        if ($ssl_client_verify != SUCCESS) {
             return 401;
        }
           
    }

    location /only-user-b {

        if ($ssl_client_verify != SUCCESS) {
             return 401; 
        }

    }

    location /user-a-or-b {

        if ($ssl_client_verify != SUCCESS) { 
             return 401;
        }

    }
```

Próbowano wprowadzić zabezpieczenia na podstawie:
```
           if ($ssl_client_s_dn_cn !~ "userB"){
                return 403;
            }  
```
Ale ostatecznie żadna metoda nie chciała działać poprawnie. 

## Testy działania Apache

Przed wybraniem certyfikatu:

![](/Foty/Do2022.05.10/apache_request_a.png)

Po wybraniu poprawnego certyfikatu:

![](/Foty/Do2022.05.10/apache_accessa_a.png)

Działa również strona A-or-B

![](/Foty/Do2022.05.10/apache_accessab_a.png)

nie działa strona B

![](/Foty/Do2022.05.10/apache_forbiddenb_a.png)

aby ponownie pokazało sie okno z wyborem certyfikatu, nalezy usunąć z automatycznie używanych certyfikatów:

![](/Foty/Do2022.05.10/delete_cert.png)

Strona B działa po wybraniu certyfikatu B

![](/Foty/Do2022.05.10/apache_accessb_b.png)

Za to strona A ponownie nie działa

![](/Foty/Do2022.05.10/apache_forbidden_a.png)


## Wnioski

Udało się utworzyć certyfikaty klientów i działają na serwerze apache. 
Nie udało się osiągnąć podobnego rezultatu na nginx.
W tym zadaniu apache okazał się być bardziej intuicyjny, a nginx sprawiał same problemy. 
