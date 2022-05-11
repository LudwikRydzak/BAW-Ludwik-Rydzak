Notatka do uruchomienia zadania 2. 

Należy mieć zainstalowany docker-compose a następnie uruchomić komendę:
docker compose -f docker-compose.yml  up

wewnątrz folderu BAW. 

dodatkowo nalezy do /etc/hosts dodać 
127.0.0.1 bawapache.pl
127.0.0.1 bawnginx.pl

dla apache porty to 6080 i 6443
dla nginx porty to 5080 i 5443
