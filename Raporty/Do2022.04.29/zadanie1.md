######## Notatki ########

https://docs.docker.com/compose/install/

#########################

# Zadanie 1.

Cele zadania:

- Utworzenie kontenerów dockera z serwerem apache i serwerem nginx,
- Wygenerowanie self-signed certyfikatów,
- Skonfigurowanie https,
- skonfigurowanie ścieżki /http-only, /https-only, oraz przekierowanie każdej innej na https

	
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

## Tworzenie kontenerów apache i nginx

Tworzenie kontenerów apache i nginx zostało zrobione za pomocą pliku docker-compose.yml

### Apache



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

Następnie odblokowano:
```
LoadModule ssl_module modules/mod_ssl.so

LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

```
 ssl_module w pliku konfiguracyjnym oraz na samym dole odblokowano 
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


