######## Notatki ########

https://docs.docker.com/compose/install/

#########################


## Instalacja Docker Compose



### Pobieranie i instalacja 
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

Do pliku /etc/hosts dodano następujące linie

```
127.0.0.1       bawnginx.pl www.bawnginx.pl
127.0.0.1       bawapache.pl www.bawapache.pl
```

## Tworzenie kontenerów apache i nginx

Tworzenie kontenerów apache i nginx zostało zrobione za pomocą pliku docker-compose.yml
