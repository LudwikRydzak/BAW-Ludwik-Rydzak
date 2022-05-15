# Zadanie 3.

W zadaniu 3. należało przygotować rate limiting dla strony /limit-a można było wykonać tylko 100 zapytań na minutę ale tylko 10 przy konkretnym headerze (np z konkretnej przeglądarki).

## Konfiguracja Nginx

Konfigurację Nginx rozpoczęto od załamania się, że nie dodano głównego pliku konfiguracyjnego do docker compose, co znacznie utrudniło wykonywanie poprzednich zadań. 

Błąd naprawiono uzupełniając konfigurację o wskazany plik. 
```
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
      - ./nginx.conf:/etc/nginx/nginx.conf

```


Przy tworzneiu konfiguracji najpierw ustawiono rate-limit dla wszystkich na 1 zapytanie na sekundę dla ścieżki /limit-a.

W tym celu dodano linijkę do pliku nginx.conf
```
limit_req_zone $binary_remote_addr zone=mylimit:1m rate=1r/s;
```

Dodano nową lokalizację do pliku nginx_default.conf

W celu większej personalizacji zmieniono kod odrzucanych requestów z 503 na 444
```
    location /limit-a/ {
        limit_req zone=mylimit nodelay;
        limit_req_status 444;
    }
```

Na poniższych zdjęciach widać wywołanie w curl żądań do serwera oraz odpowiedź serwera dla strony bezpośrednio w ścieżce /limit-a
![Rate-limit limit-a 1/s]()
![Rate-limit limit-a 1/s server response]()

Na poniższych zdjęciach widać wywołanie w curl żądań do serwera oraz odpowiedź serwera dla strony w podkatalogu /limit-a/test
![Rate-limit limit-a/test 1/s]()
![Rate-limit limit-a/test 1/s server response]()


Skoro wiadomo, że użyta dyrektywa działa, kolejnym krokiem jest ustawienie ograniczeń dla user-agenta

## Konfiguracja Appache