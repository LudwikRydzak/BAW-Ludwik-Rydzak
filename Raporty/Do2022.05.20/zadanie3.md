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

![Rate-limit limit-a 1/s](/Foty/Do2022.05.20/rate-limit_1hz_limit-a.png)

![Rate-limit limit-a 1/s server response](/Foty/Do2022.05.20/rate-limit_1hz_limit-a_server.png)


Na poniższych zdjęciach widać wywołanie w curl żądań do serwera oraz odpowiedź serwera dla strony w podkatalogu /limit-a/test

![Rate-limit limit-a/test 1/s](/Foty/Do2022.05.20/rate-limit_1hz_limit-a_test.png)

![Rate-limit limit-a/test 1/s server response](/Foty/Do2022.05.20/rate-limit_1hz_limit-a_test_server.png)


Skoro wiadomo, że użyta dyrektywa działa, kolejnym krokiem jest ustawienie ograniczeń dla user-agenta.
Konfigurację wykonano na podstawie [dokumentacji](https://www.nginx.com/blog/rate-limiting-nginx/) oraz [bloga](https://urlund.com/blog/rate-limit-nginx-by-user-agent/) z którego wzięto znaczną część kodu.
**W tym celu dodano następujący kod do pliku nginx.conf**

```

limit_req_zone $binary_remote_addr zone=default:1m rate=100r/m;


# 1 = soft, 2 = medium, 3 = hard
map $http_user_agent $rate_user {
    default "";
    "~curl/*" 2;
}

# http status to apply when rules are used
limit_req_status 429;

# soft rate limit
map $rate_user $rate_user_soft {
    default "";
    1    $http_user_agent;
}

limit_req_zone $rate_user_soft zone=rateuser_soft:16m rate=20r/m;

# medium rate limit
map $rate_user $rate_user_medium {
    default "";
    2    $http_user_agent;
}

limit_req_zone $rate_user_medium zone=rateuser_medium:16m rate=10r/m;

# hard rate limit
map $rate_user $rate_user_hard {
    default "";
    3    $http_user_agent;
}
 
limit_req_zone $rate_user_hard zone=rateuser_hard:16m rate=5r/m;

    include /etc/nginx/conf.d/*.conf;

}

```

**oraz do pliku nginx_default.conf**

```
    location /limit-a/ {
    # apply  rules
        limit_req zone=default burst=20 nodelay;
        limit_req zone=rateuser_soft burst=10 nodelay;
        limit_req zone=rateuser_medium burst=10 nodelay;
        limit_req zone=rateuser_hard burst=5 nodelay;
        limit_req_status 418;
    }

```
Dzięki dodaniu burst=10 żądania nie są dzielone równo po 6 sekund na żądanie, a możliwe jest wykonanie 10 żądań na raz i dopiero po tym czasie trzeba poczekać 6 sekund na kolejne żądanie. Działa to jak bufor 10 miejsc. 
Dzięki takiemu rozwiązaniu bardzo łatwo dodawać różne wersje ograniczeń nadając różnym user agentom różne kategorie limitingu. 

Test rozwiązania przeprowadzono na ograniczeniu curl'a do 10 żądań na minutę i 20 żądań dla wszystkich.

![test ograniczenia curla dla 10 żądań](/Foty/Do2022.05.20/rate_user_medium.png) 
![test ograniczenia defaultowego dla 20 żądań mieszane](/Foty/Do2022.05.20/rate_default.png)
![test ograniczenia defaultowego dla 20 żądań sama mozilla](/Foty/Do2022.05.20/rate_default_sama_mozilla.png)

Powyższe zdjęcia pokazują uruchomienie rate limitu kolejno :
 - dla 10 żądań przy ograniczeniu user-agenta nałożonego na curla (wywołane ograniczenie rateuser_medium)
 - dla 20 żądań dla wszystkich mieszane (wywołane ograniczenie default)
 - dla 20 żądań ale tak by na pewno pokazać, że sam nieograniczany user-agent mozilli też ulega zablokowaniu (wywołane ograniczenie default)

### Wnioski częściowe

Brak przekazania pliku konfiguracyjnego do docker-compose wywołuje duzo niechcianej frustracji dlatego należy zawsze sprawdzać przekazywane pliki.

Ustawianie prostego rate-limitingu dla jednej ścieżki na serwerze nginx jest dosyć przyjemne i działa "od ręki".

Można kontrolować dostęp dla całych ścieżek zarówno dla wszystkich jak i dla pojedynczych user-agentów (np konkretnych wersji przeglądarek)

## Konfiguracja Apache
