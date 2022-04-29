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

## aktualizacja pliku /etc/hosts


# Nginx
## tworzenie kontenera 
