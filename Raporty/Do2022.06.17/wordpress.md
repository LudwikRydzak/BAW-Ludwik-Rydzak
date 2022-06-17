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

Jak widać na zrzucie ekranu poniżej, prywatne ustawienia działają i dla innych użytkowników strona nie jest widoczna.

![widocznosc](/Foty/Do2022.06.17/widocznosc.png)

Na koniec ustaiono domenę wordpress.baw w pliku /etc/hosts pod localhosta.
## Rekonesans
Rekonesans rozpoczęto od skanu podatności. 

W tym celu wykorzystano 2 narzędzia: wpscan oraz searchsploit

```
wpscan --url http://wordpress.baw:8000  
```
Wynikiem było:
```
Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.10 (Debian)
 |  - X-Powered-By: PHP/5.6.30
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://wordpress.baw:8000/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://wordpress.baw:8000/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://wordpress.baw:8000/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.5 identified (Insecure, released on 2017-05-16).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://wordpress.baw:8000/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.7.5'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://wordpress.baw:8000/, Match: 'WordPress 4.7.5'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:01 <======================================================> (137 / 137) 100.00% Time: 00:00:01

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Thu Jun 16 15:57:39 2022
[+] Requests Done: 175
[+] Cached Requests: 4
[+] Data Sent: 43.982 KB
[+] Data Received: 18.41 MB
[+] Memory used: 206.902 MB
[+] Elapsed time: 00:00:18

```

Głównym znaleziskiem była wersja wordpressa 

```
 WordPress version 4.7.5 identified (Insecure, released on 2017-05-16)
```

Przy pomocy narzędzia searchsploit wyszukano podatności na tę wersję wordpressa.

![available_exploits](/Foty/Do2022.06.17/available_exploits.png)


Okazało się że jedna z podatności zakłada widoczność prywatnych postów i stron przy wpisaniu w url atrybutu static=1.

[Link](https://github.com/offensive-security/exploitdb/blob/master/exploits/multiple/webapps/47690.md) do opisu podatności.

## Atak
Atak polegał na użyciu prostej podatności dzięki czemu wyciekły prywatne strony administratora w tym strona z jego hasłem.

Hasło widoczne na dole zrzutu.

![wyciek_hasla](/Foty/Do2022.06.17/wyciek_hasla.png)

Hasło pozwoliło na udane zalogowanie na panel administratora.
![udane_włamanie](/Foty/Do2022.06.17/udane_wlamanie.png)

