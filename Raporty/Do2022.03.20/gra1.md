# GRA 1
<https://uw-team.org/hackme/>

Gra wita gracza krótkim przywitaniem:
> „Masz przed soba prosta gre programistyczna.
> Polega ona na lamaniu kolejnych nastepujacych po sobie haslach. 
> Nie zgaduj hasel, zerkaj do kodu! Badz uczciwy i nie zmieniaj kodu hackme lokalnie!
> Wolno ci uzywac tylko kalkulatora, olowka i twojej glowy!”

Gra polega na wypełnianiu okienka na tekst odpowiednim hasłem.
Hasło należy wyczytać z kodu gry, który można zobaczyć po przyciśnięciu prawym przyciskiem myszy na stronę,
a następnie wybraniu opcji *View page source* lub *Inspect*. 

![wstep_inspect](/Foty/Do2022.03.20/Gra1/wstep_inspect.png "Opcja Inspect w menu kontekstowym strony w przeglądarce.") 
 
## Level #1
<https://uw-team.org/hackme/level1.htm>

Gra rozpoczyna się powoli od wprowadzającego zadania. W kodzie strony widać 2 skrypty z funkcjami *sprawdz()* i *sprawdz�()*
``` javascript
<script>
function sprawdz�(){
if (document.getElementById('haslo').value=='i am too lame') {self.location.href="zaq.htm";} else {alert('Zle haselko :)');}
}
</script>
<script>
function sprawdz(){
if (document.getElementById('haslo').value=='a jednak umiem czytac') {self.location.href='ok_next.htm';} else {alert('Zle haselko :)');}
}
</script>
```
Należy zobaczyć na kod przycisku, który mówi że po jego naciśnięciu (onclick) wykona się funkcja sprawdz().
<input type="button" value="OK" onclick="sprawdz()">
Oznacza to, że poprawnym hasłem jest hasło ***a jednak umiem czytac***


## Level #2
<https://uw-team.org/hackme/ok_next.htm>

W poziomie drugim przycisk znowu wskazuje na użycie funkcji sprawdz(), która ma w sobie nieznane zmienne  w tym zmienną z hasłem „has”, której będziemy szukać. 
```javascript
<script>
function sprawdz(){
if (document.getElementById('haslo').value==has) {self.location.href=adresik;} else {alert('Nie... to nie to haslo...');}
}
</script>
```
Kod posiada jeszcze jeden skrypt, którego źródłem jest plik javascript „hasełko.js” 
``` javascript
<script src="haselko.js"></script>
```
po kliknięciu w plik, a następnie wybraniu opcji *Reveal in Sources panel* przenosi nas do zakładki obok z plikami źródłowymi.
 
![poziom2_sources](/Foty/Do2022.03.20/Gra1/poziom2_sources.png "Brak podgladu tresci obrazka.")

Niestety nie jest możliwe przejrzenie skryptu w ten sposób.

Skrypt jednak został pobrany przez przeglądarkę a jego zawartość można podejrzeć dzięki zakładce *Network --&gt Preview*.
  
![pozioim2_haselko](/Foty/Do2022.03.20/Gra1/poziom2_haselko.png "Zawratosc pliku haselko.js.")

Otrzymujemy dzięki temu informację, że zmienna has przechowuje hasło ***to bylo za proste***, a kolejne zadanie znajduje się na stronie formaster.htm czego nie musimy wykorzystywać. 

## Level #3
<https://uw-team.org/hackme/formaster.htm>

W poziome 3. przycisk znowu odnosi się do funkcji sprawdz() co prawdopodobnie będzie regułą w tej grze. 

Na stronie znajduje się jeden skrypt z 4 funkcjami:

	- right(e)
	- prawy(txx)
	- losuj()
	- sprawdz()
	
Poza funkcjami znajdują się jeszcze zmienne:
```
dod=’unknow’
literki=’abcdefh’
ost=’’
```
Wywoływana jest funkcja *prawy()*, która z kolei wykonuje funkcję *right()*, która robi coś na IE albo Netscape. 

Na google chrome funkcja zwraca true. 

Funkcja *sprawdz()* której szukamy najpierw wywołuje funkcję *losuj()*,  a następnie sprawdza czy wpisana wartość w oknie *haslo* zgadza się ze zmienną *ost*.
Jeśli tak, to przenosi na nowy adres, który jest hasłem \+ „.htm”
hasło to 
```
dod=’unknow’
literki=’abcdefh’
ost=literki.substring(2,4)+'qwe'+dod.substring(3,6);
```
Szczegółowy opis metody substring należącej do klasy String znajduje się pod tym linkiem:

<https://developer.mozilla.org/pl/docs/Web/JavaScript/Reference/Global_Objects/String/substring>

znak o indeksie 2 i 3 (bez indeksu 4) z ‘abcdefg’ to „cd”

znaki 3,4,5 z dod to „now”

Całe hasło powinno składać się do: cd \+ qwe \+ now \= ***cdqwenow*** i jest to prawidłowe hasło.

## Level #4
<https://uw-team.org/hackme/cdqwenow.htm>

W poziomie 4 czeka na nas skrypt w którym hasłem jest wynik działania. 
```javascript
<script>
function sprawdz(){
zaq=document.getElementById('haslo').value;
if (isNaN(zaq)) {alert('Zle haslo!')} else {
wynik=(Math.round(6%2)*(258456/2))+(300/4)*2/3+121;
if (zaq==wynik) {self.location.href='go'+wynik+'.htm';} else {alert('Zle haslo!')}
}}
</script>
```
Rozwiązujemy zatem działanie. Reszta z dzielenia 6\/2 to 0, 0 razy cokolwiek daje 0, więęc lewą stronę dodawania można pominąć. 

(300\/4)\*2\/3\+121 można uprościć do 50\+121 = ***171***

A zatem jest to wynik dodawania i hasło do przejścia na kolejny poziom.

## Level #5
<https://uw-team.org/hackme/go171.htm>

W zadaniu widzimy licznik który odmierza czas od 0 do 59.  Ze skryptu można wyczytać, że naszym zadaniem jest doprowadzić wartość *ile* do 861. 
``` javascript
<script>
var now = new Date();
var seconds = now.getSeconds();

function czas(){
now = new Date();
seconds = now.getSeconds();
txt.innerHTML=seconds;
setTimeout('czas()',1);
}

function sprawdz(){
ile=((seconds*(seconds-1))/2)*(document.getElementById('pomoc').value%2);
if (ile==861) {self.location.href=seconds+'x.htm'} else {alert('Zle haslo!');}
}
</script>
```
wartość ile określana jest poprzez sekundy\*(sekundy\-1)\/2\ * (cyfra pomocnicza%2) co oznacza, że cyfra pomocnicza może być albo 0 albo 1 (każda inna jest odpowiednikiem jednej z nich). 

(s(s\-1))\/2 \= 861
s(s\-1) \= 1722 
To równanie jest spełnione dla liczby s\=***42***
i aby zamek zadziałał ***należy ustawić cyfrę pomocniczą na nieparzystą*** (na przykład 1). 


## Level #6
<https://uw-team.org/hackme/42x.htm>

``` javascript
<script>
var lit='abcdqepolsrc';
function sprawdz(){
var licznik=0;
var hsx='';
var znak='';
zaq=document.getElementById('haslo').value;
for (i=1; i<=5; i+=2){
licznik++;
if ((licznik%2)==0) {znak='_';} else {znak='x';}
hsx+=lit.substring(i,i+1)+znak;
}
hsx+=hsx.substring(hsx.length-3,hsx.length);
if (zaq==hsx) {self.location.href=hsx+'.htm';} else {alert('Zle haslo!');}
}
</script>
```

Ze skryptu wynika że mamy zmienne:

*zaq*, czyli hasło, które wpisujemy w okno

*lit* -> ‘abcdqepolsrc’

*licznik* -> 0

*hsx* -> „”

*znak* -> „”

Dalej wykonywana jest pętla z iteratorem i = 1,3,5

Co każdą iterację zwiększany jest licznik o 1. 

Jeśli licznik jest parzysty to do zmiennej znak wpisywany jest ‘\_’, jeśli jest nieparzysty to znak ‘x’.
Następnie do zmiennej *hsx* dodawany jest z każdą iteracją fragment zmiennej *lit + znak*

i tak dla 3 iteracji

i \= 1

znak \= ’x’

hsx \= ‘’ \+ ’b’ \+ ’x’ \= „bx”

i \= 3

znak \= ’\_’


hsx \= „bx” \+ ’d’ \+ ’\_’ \= „bxd\_”

i \= 5

znak \= ’x’
hsx \= „bxd\_” + ’e’ \+ ’x’ \= ”bxd\_ex”
Po zakończedniu dodane są jeszcze 3 ostatnie znaki ze zmiennej *hsx*.
hsx \= „bxd\_ex” \+ ”\_ex” \= ***bxd_ex_ex***
co okazuje się być poprawnym hasłem.

## Level #7
<https://uw-team.org/hackme/bxd_ex_ex.htm>

W poziomie 7. mamy rozwiązać szyfr podstawieniowy.

Tak naprawdę można od razu przejść do kolejnej strony dzieki wskazówce z if'a, ale wtedy pozbawiamy się zabawy.
```
if (wyn=='plxszn_xrv') {self.location.href=wyn+'.htm';}
```

Dostajemy funkcję która zawiera w sobie szyfrogram oraz klucz i na tej podstawie musimy wydobyć wiadomość, którą należy wpisać do okna.
``` javascript
<script>
function sprawdz(){
zaq=document.getElementById('haslo').value;
wyn='';
for (i=0; i<=zaq.length-1; i++){
lx=zaq.charAt(i);
ly='';
if (lx=='a') {ly='z'}
if (lx=='b') {ly='y'}
if (lx=='c') {ly='x'}
if (lx=='d') {ly='w'}
if (lx=='e') {ly='v'}
if (lx=='f') {ly='u'}
if (lx=='g') {ly='t'}
if (lx=='h') {ly='s'}
if (lx=='i') {ly='r'}
if (lx=='j') {ly='q'}
if (lx=='k') {ly='p'}
if (lx=='l') {ly='o'}
if (lx=='m') {ly='n'}
if (lx=='n') {ly='m'}
if (lx=='o') {ly='l'}
if (lx=='p') {ly='k'}
if (lx=='q') {ly='j'}
if (lx=='r') {ly='i'}
if (lx=='s') {ly='h'}
if (lx=='t') {ly='g'}
if (lx=='u') {ly='f'}
if (lx=='v') {ly='e'}
if (lx=='w') {ly='d'}
if (lx=='x') {ly='c'}
if (lx=='y') {ly='b'}
if (lx=='z') {ly='a'}
if (lx==' ') {ly='_'}
wyn+=ly;
}
if (wyn=='plxszn_xrv') {self.location.href=wyn+'.htm';} else {alert('Zle haslo!');}
}
</script>
```
szyfrogram \-\> 'plxszn_xrv'
wiadomość \-\> ‘kocham cie’
Wynikowa wiadomość po wpisaniu do okna przenosi nas do kolejnego zadania. 

## Level #8
<https://uw-team.org/hackme/plxszn_xrv.htm>

``` javascript
<script>
var roz='dsabdkgsawqqqlsahdas'; var tmp=roz.substring(2,5)+roz.charAt(12);
document.write('<\s'+'c'+'r'+'i'+'p'+'t src="%7A%73%65%64%63%78%2E%6A%73"><\/s'+'c'+'r'+'i'+'p'+'t'+'>');
function sprawdz(){
zaq=document.getElementById('haslo').value; wyn=''; alf='qwertyuioplkjhgfdsazxcvbnm';
qet=0; for (i=0; i<=10; i+=2){
get+=10; wyn+=alf.charAt(qet+i); qet++;}
wyn+=eval(ax*bx*cx);
if (wyn==zaq) {self.location.href=wyn+'.htm';} else {alert('Zle haslo!');}
}
</script>
```
Na początku skryptu widzimy obfuskowany zapis dodający potajemnie dodatkowy skrypt na stronie. 

Po wpisaniu zawartości zmiennej *src* w [kalkulator hexadecymalny](https://www.asciitohex.com/) otrzymujemy nazwę skryptu.

![poziom8_hextoascii](/Foty/Do2022.03.20/Gra1/poziom8_hextoascii.png "Widok tłumaczonego fragmentu z hexadecymalnego na ascii.")

![poziom8_zsedcx](/Foty/Do2022.03.20/Gra1/poziom8_zsedcx.png "Zawartość znalezionego pliku zsedcx.js.")

Przy okazji trafiamy na niespodzianke:

![pozim8_niespodzianka](/Foty/Do2022.03.20/Gra1/poziom8_niespodzianka.png "Znaleziona niespodzianka w plikach strony.")

W skrypcie mamy zmienną wyn, która będzie wynikiem obliczeń i sumowania znaków i jej obliczenie pozwoli poznać hasło. 

Jest pętla, która iteruje od 0 do 10 włącznie co 2 oraz zmienne *qet* i *get* przy czym zmiennej *get* nigdy nie używamy i pewnie jest to pułapka, na którą trzeba uważać przy czytaniu kodu.

Iterujemy 6 razy po pętli zwiększając i o 2, za każdym razem zwiększając *qet* na końcu. Do *wyn* dopisujemy znak o indeksie *qet\+i*, czyli 6 znaków \-\> qrupjf
Na końcu dopisujemy wynik działania obliczonego w znalezionym skrypcie.
ax \= 6
bx \= 3
cx \= 9
6 \* 3 \* 9 \= 162
wyn \= ***qrupjf162***

## Wygrana!
<https://uw-team.org/hackme/qrupjf162.htm>

![wygrana](/Foty/Do2022.03.20/Gra1/wygrana.png "Wygrana!")
 
