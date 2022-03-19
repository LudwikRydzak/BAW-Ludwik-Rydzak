# GRA 2
<https://uw-team.org/hm2/>

![Zasady](/Foty/Do2022.03.20/Gra2/wstep.png "Jedna zasada, wszystkie chwyty dozwolone")

Do rozwiązania zagadek posłużą:

- Prawy przycisk myszy \-\> inscpect
- narzędzie *curl*

## Level #1
<https://uw-team.org/hm2/level1.htm>

W poziomie pierwszym znajdujemy przycisk uruchamiający fukcję spr(), która znajduje się w poniższym skrypcie.

```javascript
<script>
function spr(){
if (document.getElementById('formularz').value==document.getElementById('haslo').value)
{ self.location.href=document.getElementById('haslo').value+'.htm'; } else {alert('Nie, to nie to haselko :(');}
}
</script>
```

Szukana wartość hasła jest taka sama jak wartość zmiennej value elementu *formularz*.

```javascript
<input value="text" name="formularz" id="formularz" type="hidden">
```
value formularza to ***text*** i jest to szukane hasło.

## Level #2
<https://uw-team.org/hm2/text.htm>

```javascript
<script>
function spr(){
if (document.getElementById('haslo').value==unescape('%62%61%6E%61%6C%6E%65')) 
{ self.location=document.getElementById('haslo').value+'.htm'; } else { alert('Zle haslo!'); }
}
</script>
```

Ze skryptu można wyczytać, ze hasło powinno odpowiadać wartością pewnemu ciągowi zapisanemu hexadecymalnie. 

Po przekonwertowaniu tego fragmentu w [kalkulatorze ascii](https://www.asciitohex.com/) otrzymujemy hasło ***banalne***.

Alternatywnie można wyszukać kompilator javascript online i tam wykonywac fragmenty kodu sprawdzając co z nich wynika. 

## Level #3
<https://uw-team.org/hm2/banalne.htm>

W poziomie trzecim znajdujemy skrypt który mówi, że wartość podana jest najpierw konwertowana na liczbę a potem na liczbę binarną i ma być równa 10011010010.

```javascript
<script>
function binary(liczba) {
return liczba.toString(2);
}
function spr(){
if (binary(parseInt(document.getElementById('haslo').value))==10011010010) 
{ self.location=document.getElementById('haslo').value+'.htm'; } else { alert('Zle! \nPodstawy matematyki sie klaniaja :)');}
}
</script>
```
Po przekonwertowaniu 10011010010 na system dziesiętny w systemowym kalkulatorze otrzymano hasło ***1234***

## Level #4
<https://uw-team.org/hm2/1234.htm>

Do kolejnego poziomu będzie potrzebne użycie narzędzia curl. 

```bash
curl https://uw-team.org/hm2/1234.htm 
```

Otrzymujemy zawartość strony:

```javascript
<!-- FIGE Z MAKIEM DOSTANIESZ, A NIE HASLO! //-->
<html><head><title>Hackme 2.0 - by Unknow</title></head><body text="white" bgcolor="black" link="yellow" vlink="yellow" alink="yellow">
<script>
haslo='';
cos=parseInt(unescape('%32%35%38'));
while ((haslo!=cos.toString(16)) && (haslo!='X') ){
haslo=prompt('podaj haslo:\nwpisz X aby zatrzymac skrypt','');
}
if (haslo==cos.toString(16)) { self.location=haslo+'.php';} else {self.location='http://www.uw-team.org/';}
</script>
<h3>Hackme 2.0 - level #4</h3>
<a href="hehehe.htm">Kliknij mnie :)</a>
</body></html>
```

Odkodowanie ciągu unescape('%32%35%38') w [kalkulatorze ascii](https://www.asciitohex.com/) (lub w kompilatorze javascript)
prowadzi do otrzymania liczby 
***102***, która jest hasłem do tego poziomu. 

## Level #5
<https://uw-team.org/hm2/102.php>

Kod kolejnego etapu jest niewidoczny, natomiast dostajemy kod, który jest podpowiedzią. 

```
if (!isset($haslo)) {$haslo='';}
if (!isset($login)) {$login='';}
if ($haslo=="tu jest haslo") {$has=1;}
if ($login=="tu jest login") {$log=1;}
if (($has==1) && ($log==1)) { laduj nastepny level } else { powroc do tej strony } 
```

Wiemy, że zmienna *has* i zmienna *log* mają być równe 1.

Zmienne zmieniają wartość przy odpowiednio wpisanym tekście ale można je podać również w url’u.
Po wpisaniu tekstu w pola *haslo* i *login* można zauważyć zmianę url'a gdzie po znaku zpaytania *?* podane są wartości zmiennych.
Zamiast polegać na instrukcjach warunkowych opartych na zmiennych *haslo* i *login*, można zmienić bezpośrednio zmienne  *has* i *log*.

<https://uw-team.org/hm2/102.php?log=1&has=1>

Po wejściu w odpowiednio spreparowany link przechodzimy do kolejnego poziomu.

![poziom5_brawo](/Foty/Do2022.03.20/Gra2/poziom5_brawo.png "Brawo!")

## Level #6

<https://uw-team.org/hm2/url.php>

Kolejny poziom informuje nas, że dostaliśmy ciasteczko. 
Jest to oczywiście nawiązanie do mechanizmu „plików cookies”, czyli ciasteczek, które zazwyczaj przechowują różne informacje na temat sesji lub gromadzą różne dane statystyczne. 

Można je zobaczyć jako fragment nagłówka

![poziom6_header](/Foty/Do2022.03.20/Gra2/poziom6_header.png "Headers: cookie")
 
Lub w zakładce Cookies

![poziom6_cookies](/Foty/Do2022.03.20/Gra2/poziom6_cookies.png "Ciasteczka")
 
Jeśli ktoś wykradnie nasze ciasteczko, może ukraść też sesję i mieć dostęp do jakiejś platformy bez faktycznych danych logowania. 
W tym przypadku za pomocą ciasteczka przekazano adres strony kolejnego zadania.

## Level #7
<https://uw-team.org/hm2/ciastka.htm>

Kolejny poziom wymaga hasła.

![poziom7_podaj haslo](/Foty/Do2022.03.20/Gra2/poziom7_podajhaslo.png "Podaj haslo prompt")

```bash
curl https://uw-team.org/hm2/ciastka.htm
```

```javascript
<html><head><title>Hackme 2.0 - by Unknow</title></head><body text="white" bgcolor="black" link="yellow" vlink="yellow" alink="yellow">
<script>
strona='zle.htm';
haslo=prompt("Hackme 2.0 - level #7 \nPodaj has�o","") 
document.write('<script type="text/javascript" src="include/'+haslo+'.js"><\/script>'); 
onload=function(){ 
if(haslo==null) { self.location='http://www.uw-team.org/' } else location.href=strona; }
</script>
</body></html>  
```

Z pobranej zawratości strony wiemy, że kolejna strona znajduje się w katalogu include serwera strony.
Niektóre strony pozwalają na przeglądanie katalogów, a niektóre są podatne na *path traversal*, czyli podatność pozwalająca na przeglądanie zawartości nie tylko udostępnianej przez aplikację serwera ale też i całego dostępnego systemu plików. 

Zwykle pozwala to tylko na ograniczony dostęp do zasobów, kiedy serwer jest uruchomiony na dedykowanym koncie użytkownika, ale nawet wtedy istnieje możliwość dalszych eskalacji uprawnień.

Po wpisaniu w pasku adresu katalogu \- <https://uw-team.org/hm2/include/> \- widać kolejny plik cosik.js.

![poziom7_include](/Foty/Do2022.03.20/Gra2/poziom7_include.png "Wnetrze katalogu include na serwerze")
 
Wewnątrz pliku *cosik.js* znajduje się jedna linijka z adresem kolejnej strony:

```
strona='listing.php';
```

## Level #8
<https://uw-team.org/hm2/listing.php>

Poziom ósmy zabezpieczony jest poprzez referrera. Jeśli wchodzilibyśmy na tę stronę przez link pochodzący z onet.pl, wtedy w nagłówku jako referrer byłby wpisany onet.pl i zostalibyśmy wpuszczeni na stronę.

![poziom8_referent](/Foty/Do2022.03.20/Gra2/poziom8_referent.png "Zabezpieczenie strony. Trzeba wejsc przez onet.pl.")

```bash
curl https://uw-team.org/hm2/listing.php
```
Już w tym momencie można przeczytać zwartość strony wraz z hasłem ale znacznie łatwiej jest uruchomić znaczące fragmenty tej strony lokalnie w przeglądarce.
Wybrano więc jedną z poprzednich stron i edytowano jej kod. 
 
![poziom8_edit](/Foty/Do2022.03.20/Gra2/poziom8_edit.png "Edytowanie statycznej zawartości strony w przeglądarce.")
 
Podmieniono zawartość strony na zawartość obecnej strony i zakomentowano „zabezpieczenie”.

![poziom8_komentarz](/Foty/Do2022.03.20/Gra2/poziom8_komentarz.png "Zakomentowane zabezpieczenie")
 
Ostatecznie z linii
```
<div id="ukryte" style="display:none">
```
usunięto atrybut *style* i zmieniono background strony na zielony
 
![poziom8_zielony](/Foty/Do2022.03.20/Gra2/poziom8_zielony.png "Zielone tlo. Bardzo ładne.")
 
Można było też po prostu zaznaczyć fragment strony z napisem.

***kxnxgxnxa*** to hasło.

Otrzymujemy wiadomość że kolejny etap ukryty jest w pliku *pokaz.php*.

## Level #9
<https://uw-team.org/hm2/pokaz.php>

Kolejny poziom wita nas alertem z informacją, że dopiero po godzinie 1 w nocy można odwiedzić stronę. Pobieramy jej zawartość curlem. 

```bash
curl https://uw-team.org/hm2/pokaz.php   
```
 
```javascript
<html><head><title>Hackme 2.0 - by Unknow</title></head><body text="white" bgcolor="black" link="yellow" vlink="yellow" alink="yellow">
<script>
var now = new Date();
var godzina = now.getHours();
var minuta = now.getMinutes();
if ((godzina>23) && (minuta>55)) {
} else { alert('Dostep do tej strony mozliwy jest jedynie po godzinie 1 w nocy! \nObejdz to jakos :)'); self.location='http://www.uw-team.org/';}
</script>
<h3>Hackme 2.0 - level #9</h3>
<!-- Widze ze zrodlo juz masz... albo je sciagnales, albo przestawiles sobie godzine na komputerze ;) //-->
<font color="lime"><pre>
01100111 01110010 01100001 01110100 01110101 01101100 01100001 01100011 01101010 01100101 00100001 
00100000 01110101 01100100 01100001 11000101 10000010 01101111 00100000 01000011 01101001 00100000 
01110011 01101001 11000100 10011001 00100000 01110101 01101011 01101111 11000101 10000100 01100011 
01111010 01111001 11000100 10000111 00100000 01110100 01100101 00100000 01110111 01100101 01110010 
01110011 01101010 01100101 00100000 01001000 01100001 01100011 01101011 01101101 01100101 00101110
</pre></font>
<br>Milego dekodowania :)
</body></html>
```   

Po wklejeniu wiadomości w [translator binary to ascii](https://www.asciitohex.com/) otrzymano informację o wygranej.

![poziom9_wygrana](/Foty/Do2022.03.20/Gra2/poziom9_wygrana.png "Wygrana!")
