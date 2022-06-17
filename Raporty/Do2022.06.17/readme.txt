Projekt ataku na wybraną wersję wordpressa. 
Ludwik Rydzak

Ciekawym przykładem tego zadania jest maszyna backdoor ze strony hackthebox. 
(Tam wykorzystano path traversal do wstępnego dostępu do plików na serwerze,
a następnie iterując po folderach proc znaleziono proces odpowiedzialny za pozostawiony tytułowy backdoor) 

W tym przykładzie pokazano jednak inną podatność wordpressa, która pozwoliła na uzyskanie dostępu do prywatnych stron administratora. 


plik docker-compose.yml zawiera podstawowy obraz wordpressa w starej wersji (4.7) oraz jakąś bazę danych. 

Wystarczy go zbudować i uruchomić komendami:

docker compose -f docker-compose.yml  build

docker compose -f docker-compose.yml  up