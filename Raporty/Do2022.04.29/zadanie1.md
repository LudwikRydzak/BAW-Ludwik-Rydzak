######## Notatki ########

https://docs.docker.com/compose/install/
mkdir http-only https-only https-http only-user-a only-user-b admin

#########################

# Zadanie 1.

Cele zadania:

- Utworzenie kontenerów dockera z serwerem apache i serwerem nginx,
- Wygenerowanie self-signed certyfikatów,
- Skonfigurowanie https,
- skonfigurowanie ścieżki /http-only, /https-only, /http-https oraz przekierowanie każdej innej na https

	
## Pobieranie i instalacja Docker Compose 
Instalacja docker compose została wykonana na podstawie dokumentacji ze [strony dockera](https://docs.docker.com/compose/install/). 

``` bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```
### Test instalacji 

``` bash
docker compose version
```

## Aktualizacja pliku /etc/hosts

Dla celów estetycznych skonfigurowano 2 domeny w pliku /etc/hosts.

Do pliku /etc/hosts dodano następujące linie

```
127.0.0.1       bawnginx.pl www.bawnginx.pl
127.0.0.1       bawapache.pl www.bawapache.pl
```

## Generowanie self-signed certyfikatów

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key_nginx.key -out cert_nginx.pem
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key_apache.key -out cert_apache.crt
```

## Tworzenie kontenerów apache i nginx

Tworzenie kontenerów apache i nginx zostało zrobione za pomocą pliku docker-compose.yml

Jako, że jeden port może należeć tylko do jednego serwisu zdecydowano się nie oddawać portu 80 i 443 
żadnemu z serwerów. 

Każdy serwer otrzymał swoją numerację gdzie:
 - dla nginx zarezerwowano porty  5080 i 5443
 - dla apache zarezerwowano porty 6080 i 6443

utworzono dwa serwisy:
```yml
services:
	bawnginx:
	
	bawapache:
```
### Apache
Serwis apache w yaml'u:

```yml
bawapache:
    image: httpd:latest
    ports:
      - 6080:80
      - 6443:443
    volumes:
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf
      - ./bawapache.pl/:/var/www/bawapache.pl/
      - ./httpd_ssl.conf:/usr/local/apache2/conf/extra/httpd-ssl.conf
      - ./cert_apache.crt:/usr/local/apache2/conf/server.crt
      - ./key_apache.key:/usr/local/apache2/conf/server.key
```

Wykorzystano oficjalny obraz apache'a "httpd:latest"

Przemapowano porty maszyny (zewnętrzne) na porty wewnątrz serwisu. 

Dodatkowo przekazano kontenerom ustawienia: 
	- plik konfiguracyjny serwera
	- plik konfiguracyjny ssl
	- pliki certyfikatu i klucza 
	- pliki z zawartością strony do testowania

### Nginx








## Konfiguracja serwera Apache

Na samym początku skonfigurowano stronę tak, aby uruchamiała się w wersji http.
 
W tym celu podmieniono istniejące linie DocumentRoot na nstępujące: 
```
DocumentRoot "/var/www/bawapache.pl"
<Directory "/var/www/bawapache.pl">
```
Oraz umieszczono własny plik index.html w plikach serwera.

Po odwiedzeniu strony, przedstawia się ona następująco:

![Strona apache http](apache_port80.png)

Następnie odblokowano ssl_module w pliku konfiguracyjnym:
```
LoadModule ssl_module modules/mod_ssl.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
```

na samym dole odkomentowano również:
```
Include conf/extra/httpd-ssl.conf
```
Jest to plik konfiguracyjny służący do ustawienia połączenia szyfrowanego. 

Plik pobrano z kontenera za pomocą komendy 

```
#docker cp <containerId>:/file/path/within/container /host/path/target
docker cp b1101c7f7089:/usr/local/apache2/conf/extra/httpd-ssl.conf httpd_ssl.conf
```

W pliku konfogiracyjnym znaleziono domyślne ścieżki dla plików certyfikatu oraz klucza.

```
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
```

Zaktualizowano dane strony:
```
DocumentRoot "/var/www/bawapache.pl"
ServerName bawapache.pl:443
```

Przeglądarka wykrywa certyfikat. 
![Złota ramka]()

Strona działa poprawnie
![strona Apache na procie 443]()

### Przekierowanie Apache 

Do przekierowywania stron w Apach będzie potrzeby rewrite module, który uaktywniamy w pliku konfiguracyjnym.
```
LoadModule rewrite_module modules/mod_rewrite.so
```

Do zarządzania subdomenami użyto następującej logiki:
	1. Wszystkie strony https-only zablokowano na porcie 80
	2. Wszystkie strony http-only zablokowano na porcie 443
	3. Wszystkie strony http-https mają działać na obu portach
	4. Wszystkie strony poza https-only, http-https(bo punkt 3.) przekierowano z 80 -> 443

W tym celu dodano do pliku konfiguracyjnego:
```
RewriteEngine on

RewriteRule "^/https-only/(.+)" "-" [F]
RewriteRule "^(?:(?!\/http-https\/|\/http-only\/).)*$"  "https://bawapache.pl:6443/%{REQUEST_URI}"  [R,L]
```
Blokowanie wszystkich stron /https-only na porcie 80.
Przekierowywanie wszystkich stron które nie mają w sobie /http-only/ oraz /http-https/ na port 443


Oraz do pliku konfiguracyjnego ssl:
```
RewriteEngine on
RewriteRule "^/http-only/(.+)" "-" [F]
```
Blokowanie wszystkich stron /http-only na portcie 443

### Testy poprawności działania

1. Blokowanie https-only na porcie 80
2. Blokowanie http-only na porcie 443
3. Działanie http-https na obu portach
4. Przekierowywanie innych adresów na port 443


## Konfiguracja serwera Nginx 
```
docker cp 5db4f5a83a47:/etc/nginx/conf.d/default.conf nginx_default.conf 
docker cp 5db4f5a83a47:/etc/nginx/nginx.conf nginx.conf   
```

Do pliku konfiguracyjnego dopisano fragment odpowiadający za ssl. 

Został on dodany osobno, a nie połączony, aby można było łatwiej dokonywać zmian.  

```
 server {

    listen      443 ssl;
    server_name  bawnginx.pl;
    ssl_certificate_key /etc/nginx/ssl/bawnginx.pl.key;
    ssl_certificate     /etc/nginx/ssl/bawnginx.pl.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES>
    ssl_prefer_server_ciphers off;

    add_header Strict-Transport-Security max-age=31536000;

    location / {
        root   /usr/share/nginx/bawnginx.pl;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

### Przekierowanie nginx

Do zarządzania subdomenami użyto następującej logiki:
	1. Wszystkie strony https-only zablokowano na porcie 80
	2. Wszystkie strony http-only zablokowano na porcie 443
	3. Wszystkie strony http-https mają działać na obu portach
	4. Wszystkie strony poza https-only, http-https(bo punkt 3.) przekierowano z 80 -> 443
	
W tym celu w serwerze zajmującym się portem 80 dodano:

```
 location /https-only {
        deny all;
        return 403;
    }
 location ^(?:(?!\/http-https\/|\/http-only\/).)*$ {
        return 301 https://bawnginx.pl$request_uri;
    }

```

Na serwerze zajmującym się portem 443 dodano:
```
 location /http-only {
        deny all;
        return 403;
    }
```


