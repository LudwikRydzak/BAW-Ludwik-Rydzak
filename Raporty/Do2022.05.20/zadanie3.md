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

Wybierany jest ostatni zaktualizowany status. Ustawiono globalnie status 429 oraz lokalnie dla ścieżki status 418 i zwracany był 418.

## Konfiguracja Apache

Jako że nie istnieje sposób na rate-limit domyślnie zaimplementowany do "core'a" apache'a potrzebna jest instalacja odpowiedniego modułu zapoewniającego taką możliwość. 

Aby to osiągnąć należy zmienić jednak sposób inicjowania kontenera. Zamiast w docker compose korzystać z gotowego obrazu apache poprzez dyrektywę
image:httpd, nalezy użyć build: i zbudować z dockerfile'a budującego obraz domyślny httpd i doinstalować do niego odpowiedni moduł.

Na pierwszą (nieudaną) próbę wybrano moduł **mod_cband**

Zadanie rozpoczęto więc od zmiany w docker compose:

```
bawapache:
    build:
      context: .
      dockerfile: ApacheDockerFile
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

Wymieniony plik **ApacheDockerFile**:

```
FROM httpd

RUN apt-get update \
        && apt-get install -y git gcc make
RUN apt-get install -y libapr1 libapr1-dev libaprutil1-dev
RUN git clone https://github.com/maiha/mod_cband /tmp/ \
        && cd /tmp/ \
        && ./configure \
        && make \
        && make install \
        && rm -rf /tmp/* \
        && cp /usr/local/apache2/modules/mod_cband.so /tmp/


```

Moduły libapr1 libapr1-dev libaprutil1-dev są wymagane przy budowaniu modułu (czego nie znaleziono w dokumentacji do modułu).

po instalacji dodano do pliku httpd.conf:
```
LoadModule cband_module       modules/mod_cband.so
```

Następnie zmodyfikowano plik httpd_ssl.conf dodając dyrektywę CBandSpeed:

```
<IfModule mod_cband.c>
  CBandSpeed 500 1 5
</IfModule>
```

Kolejne wrtości oznaczają:

	- 500 - przepustowość w kb/s
	- 1 - liczbę requestów na sekundę
	- 5 - liczbę jednoczesnych połączeń

![test limitowania requestów]()

**Problem**

Dyrektywy tego modułu działają jedynie w kontekście VirtualHosta, nie mogą działać ani wewnątrz <If>, ani wewnątrz <Location>. Znacznie utrudnia to wykonanie zadania. 

Zdecydowano się utworzyć szczególne zasady: Ograniczyć 1 raz na sekundę ruch dla wszystkich (w domyśle ograniczony user agent),
 przekeirować resztę żądań na innego, identycznego co do zawartości virtualhosta, którego limit będzie 2x wiekszy (aby odpowiadał w przybliżeniu założeniom zadania).
 Wewnątrz tego virtualhosta wywołać If'a który będzie wywoływał response 503 jesli jest to ograniczony user, albo będzie pokazywał stronę normalnie, jeśli jest to nieograniczony user. 
 
 Takie rozwiązanie pomimo swojej zawiłości powinno w teorii zapewnić przybliżone do założonych rezultaty.


Próbowano przekierowywać nieudane połączenia na inny URL dyrektywą CBandExceededURL, ale nie zadziałało to poprawnie.

W tym celu utworzono nowy VrtualHost któwy miał być identyczny do zwyklej strony a nastepnie przekierowano dyrektywą:
```
CBandExceededURL https://bawlimit.pl:6443/%{HTTP_HOST}
```
  
Jak wspomniano wcześniej powyższe próby nie zadziałały i strona po osiągnieciu limitu odpowiadała domyślnym kodem 503.

Sam moduł działa bardzo źle, zamisat limitować 1 żądanie na sekundę to opóźnia bardzo dużą część żądań zanim jakiekolwiek zablokuje. Moduł działa wolno i część dyrektyw nie działa prawidłowo. Jest bardzo ograniczony co do kontekstu jego użycia i jego funkcji.


### Nowy moduł - mod_qos

Moduł mod_qos służy głównie do blokowania ataków DoS. 

Instalacja tego modułu jest zadziwiająco prosta w porównaniu do poprzedniego. 
Wystarczy w dockerfile wpisać:
```
RUN apt-get install -y libapache2-mod-qos
```


Jednak po zmaganiach z poprzednim modułem zostały w dockerfile przydatne instalacje, ponieważ ten moduł równiez używa libapr1. 

Również nie zaszkodzi mieć gita. Cały ApacheDockerFile wygląda następująco:
```
FROM httpd

RUN apt-get update \
        && apt-get install -y git gcc make
RUN apt-get install -y libapr1 libapr1-dev libaprutil1-dev
RUN apt-get install -y libapache2-mod-qos
```

W tym miejscu warto wspomnieć o niepowodzeniach przy okazji instalacji.
Przy próbie zbudowania obrazu 

```
docker compose -f docker-compose.yml down 
docker compose -f docker-compose.yml build 
#Tutaj pojawił się błąd

docker compose -f docker-compose.yml up
```
pojawił się błąd:

**Temporary failure resolving 'deb.debian.org'**

Aby go rozwiązać najpierw zrestartowano dockera, a jak to nie przyniosło poprawy, to całą maszynę wirtualną. Reset przyniósł pozytywny skutek.


Do modułu dostać się można ładując go w pliku httpd.conf:
```
LoadModule qos_module  /usr/lib/apache2/modules/mod_qos.so
```
Jest to inna ścieżka niż przy reszcie modułów które instalują się w /usr/locale zamiast /usr/lib.

### Ustawienia rate-limitu

Do ustawienia rate limitu skorzystano z [dokumentacji](http://mod-qos.sourceforge.net/)
Ostatecznie zdecydowano się na dyrektywę QS_EventLimitCount dla limitowania wszystkich do 100 żądań na minutę (60 sekund) 
oraz QS_CondClientEventLimitCount do limitowania konkretnego user-agenta 10 żądań na minutę (60 sekund). 
Kod wklejony przed VrtualHost'em (ustawienia nie mogły działać w kontekście virtualhosta)

```
SetEnvIf Request_URI ^/limit-a Limit_a

QS_EventLimitCount Limit_a 100 60

SetEnvIf User-Agent curl QS_COND=curl
QS_CondClientEventLimitCount 10 60 Limit_a curl

```

### Testy działania

Testy działania przeprowadzono na limitach 10r/m dla limitowanego user_agneta i 15r/m dla nie limitowanego. 

**Najpierw wykonano 10 zapytań curl'em do zablokowania, a następnie 5 kolejnych zapytań Przeglądarką do zablokowania**

![apache_test]()

Na zdjęciu widać, że serwer do zablokowania curla użył dyrektywy QS_CondClientEventLimitCount, a do zablokowania reszty ruchu QS_EventLimitCount.


**Wykonano 17 zapytań mozillą aby pokazać brak aktywacji dyrektywy dla user-agenta przy innym user-agencie**

![apache_mozilla_only]()

Na zdjeciu widać również jak zwiększa się counter przy kolejnych żądaniach.


**Na koniec wykonano serię zapytań curlem na inną stronę niż /limit-a**

![apache_test_home]()

Test wypadł pozytynie nie blokując żadnego żądania. 

## Wnioski

Zadanie rate-limitingu było bardzo proste dla nginx gdzie domyślnie intuicyjnie zablokowano niechciany ruch. 

W celu usprawnienia działania dodano dodatkowo parametr burst aby czas na kolejne żądania nie był podzielony równo po 6 sekund. 

Samo ustawienie rozbudowano na podstawie bloga o konfigurację poszczególnych grup limitujących co znacznie automatyzuje proces konfiguracji.

Dla apache'a zadanie rate limitingu było bardzo dużym wyzwaniem. Wszystkie moduły są nieaktualizowane i nie ma łatwo dostępnej widzy w internecie z którego modułu korzystać. 

Wybrano najpierw moduł mod_cband który okazał się być problematyczny w instalacji oraz niemożliwy w konfiguracji do potrzeb zadania. 

W następnej kolejności wybrano moduł mod_qos i była to duża odmiana zarówno w instalacji jedną linijką w dockerfile,
 a następnie w konfiguracji bezpośrednio pod potrzeby zadania.

Finalnie konfiguracja nginx zajęła wraz z rozpoznaniem się w temacie 2 godziny, 
a konfiguracja Apache'a wraz z poszukiwaniem modułów, rozwiązywaniem problemów instalacji, próbami konfiguracji, 
wybieraniem nowego modułu, instalacją i wybraniem z dokumentacji odpowiednich dyrektyw, które odpowiednio użyto zajęła 4 dni.

Znaczym ułatwieniem dla zadania byłoby wskazanie modułu mod_qos ponieważ praca z nim była rzeczywiście przyjemna, 
pomimo ogromnych rozmiarów modułu i obszernej dokumentacji (Która zdaje sie celowo pomijała przykłady dla użytych dyrektyw).


## Konkluzja

Zadanie udało się wykonać zarówno dla nginx i apache w 100%. 



 