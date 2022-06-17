# Wordpress włamanie


## Wordpress instalacja

Wordpress w wersji 4.7 zainstalowano przy użyciu docker compose. Plik docker-compose.yml wygląda następująco:
``` yml
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:4.7
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```


Utworzenie obrazu i uruchomienie wykonano przy użyciu standardowych komend konsoli build i up

```
docker compose -f docker-compose.yml  build
docker compose -f docker-compose.yml  up
```
## wordpress - Panel administratora
                                                                                     
Przy pierwszym uruchomieniu uruchamia się panel instalacyjny i wybieranny jest język.

![pierwsze_uruchomienie](/Foty/Do2022.06.17/pierwsze_uruchomienie.png)

Zostało utworzone przykładowe konto administratora.

Login: Ludwik

Hasło: root

![konto](/Foty/Do2022.06.17/konto.png)

Na koniec wordpress powiadamia o udanej instalacji.

![installed](/Foty/Do2022.06.17/installed.png)

## Scenariusz podatności - Prywatna strona z zapisanym hasłem do konta. 

Na potrzeby zadania utworzono w wordpressie **page** z danymi logowania z niewiadomych przyczyn zapisanych w ten właśnie sposób. Mogły to być dane do konta bankowego, czy inne prywatne informacje. 

W tym celu utworzono nową stronę i ustawiono jej widoczność jako prywatną

![private_page](/Foty/Do2022.06.17/private_page.png)

wpscan --url http://wordpress.baw:8000  

![set_rhost](/Foty/Do2022.06.17/set_rhost.png)
![available_exploits](/Foty/Do2022.06.17/available_exploits.png)
![wyciek_hasla](/Foty/Do2022.06.17/wyciek_hasla.png)
![udane_włamanie](/Foty/Do2022.06.17/udane_włamanie.png)

