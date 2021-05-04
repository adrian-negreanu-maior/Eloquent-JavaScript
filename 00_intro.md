{{meta {load_files: ["code/intro.js"]}}}

# Introduction

{{quote {author: "Ellen Ullman", title: "Close to the Machine: Technophilia and its Discontents", chapter: true}

Suntem convinși că de fapt construim un sistem pentru propriile noastre scopuri. Credem că îl construim ca pe o 
imagine a noastră... Dar computerul nu este ca și noi. Este o proiecție a unei părți foarte mici din eul nostru: 
acea parte dedicată logicii, ordinii, regulilor și clarității.

quote}}

{{figure {url: "img/chapter_picture_00.jpg", alt: "Imaginea unei șurubelnițe și a unei plăci integrate", chapter: "framed"}}}

Aceasta este o carte despre instruirea computerelor. Computerele sunt, în zilele noastre, cam la fel de comune 
ca și șurubelnițele, doar că sunt puțin mai complexe, și a le determina să facă ceea ce dorim noi nu este 
întotdeauna ușor.

Dacă sarcina pe care vrem să o delegăm computerului este una obișnuită, bine înțeleasă, cum ar fi să ne afișeze
lista de emailuri sau să acționeze ca și un calculator de birou, putem să deschidem aplicația corespunzătoare
și să începem lucrul. Dar pentru sarcini de lucru unice sau fără un final clar, probabil că nu vom găsi o aplicație.

Acesta este momentul in care _programarea_ ar putea intra în scenă. _Programarea_ este acțiunea de a construi un 
_program_ - un set de instrucțiuni precise care să îi spună computerului ce are de făcut. Deoarece computerele
sunt niște fiare destul de tâmpite dar foarte pedante, programarea este fundamental plictisitoare și frustrantă.

{{index [programming, "joy of"], speed}}

Din fericire, dacă puteți trece peste acest fapt și poate chiar să vă încânte rigoarea necesară pentru a gândi în 
termenii cu care pot opera niște mașini tâmpite, programarea vă poate da multe satisfacții. Vă permite să finalizați în câteva secunde acțiuni care ar dura foarte mult dacă le-ați executa manual. _Programarea_ este un mod de a determina un 
computer să efectueze operații pe care nu le putea efectua anterior. Și să nu uităm, este un exercițiu extraordinar pentru
gândirea abstractă.

Cea mai mare parte a programării se realizează cu ajutorul limbajelor de programare. Un _limbaj de programare_ este un
limbaj construit artificial care este utilizat pentru instruirea computerului. E interesant că cel mai eficient mod pe care l-am descoperit pentru a comunica cu un computer împrumută atât de multe din modul în care comunicăm între noi. Ca și limbile de comunicare umane, limbajele pentru computere permit combinarea cuvintelor și a frazelor în moduri noi, ceea ce face posibilă exprimarea unor concepte complet noi.

{{index [JavaScript, "availability of"], "casual computing"}}

La un moment dat, interfețele bazate pe limbaj, cum ar fi prompterele BASIC si DOS din anii 1980 și 1990 erau principalele metode de interacțiune cu computerele. Ele au fost înlocuite pe scară largă de către interfețe vizuale, care sunt mai ușor de învățat dar oferă mai puțină libertate. Limbajele pentru computer sunt încă prezente, dacă știți cum să le găsiți. Un asemenea limbaj este JavaScript, prezent în orice browser modern și deci disponibil pe aproape orice dispozitiv.

{{indexsee "web browser", browser}}

Această carte va încerca să vă familiarizeze suficient cu acest limbaj pentru a realiza programe utile și amuzante cu ajutorul său.
## Despre programare

{{index [programming, "difficulty of"]}}

Pe lângă a explica JavaScript, voi introduce principiile de bază ale programării. Programarea este dificilă. Regulile fundamentale sunt simple și clare, dar programele construite cu ajutorul acestor reguli tind să devină suficient de complexe pentru a introduce propriile lor reguli și un nou nivel de complexitate. Veți construi propriul dumneavoastră labirint, într-un anume mod, și ați putea cu ușurință să vă pierdeți în interiorul său.

{{index learning}}

Vor fi momente în care citirea acestei cărți va fi teribil de frustrantă. Dacă sunteți începător în programare, va exista o mulțime de material nou pe care să îl înțelegeți. Mare parte din acest material va fi apoi _combinat_ in moduri care necesită să realizați conexiuni adiționale.

Depinde de dumneavoastră să depuneți efortul necesar. Când vă luptați să urmăriți acțiunile din carte, nu vă grăbiți să trageți vreo concluzie cu privirile la propriile abilități. Sunteți în regulă, trebuie doar să persistați. Luați o pauză, recitiți anumite părți din material și asigurați-vă că citiți și înțelegeți programele date ca exemplu și exercițiile. Învățarea nu este o activitate ușoară, dar tot ceea ce ați învățat vă aparține și va facilita următorii pași de învățare.

{{quote {author: "Ursula K. Le Guin", title: "The Left Hand of Darkness"}

{{index "Le Guin, Ursula K."}}

Când acțiunile devin neprofitabile, strângeți informații; când informațiile devin neprofitabile, dormiți.

quote}}

{{index [program, "nature of"], data}}

Un program înseamnă mai multe lucruri. Este o bucată de text tastată de către un programator, este forța directoare care determină computerul să facă ceea ce face, este un set de date în memoria computerului, dar și controlează acțiunile efectuate asupra aceleiași memorii. Analogiile care încercă să compare programele cu obiecte care ne sunt familiare tind să eșueze rapid. O analogie care reușește să fie potrivită doar la suprafață este cea cu o mașină - multe piese separate par sș fie implicate, și pentru ca să înțelegem cum funcționează întregul, trebuie să înțelegm modurile în care aceste părți se interconectează și contribuie la buna funcționare a întregului.

Un computer este o mașină fizică al cărei rol este de a găzdui aceste mașini imateriale. Computerele în sine pot să execute doar acțiuni elementare și chiar stupide. Motivul pentru care sunt atât de utile este capacitatea lor de a executa aceste operații elementare la viteze extraordinar de mari. Un program poate combina în mod ingenios un număr enorm de asemenea operații elementare pentru a realiza operații extrem de complicate.

{{index [programming, "joy of"]}}

Un program este rezultatul gândirii. Nu este costisitor să îl construim, nu are greutate și crește ușor sub mâinile noastre în timp ce tastăm.

Dar dacă nu avem grijă, dimensiunea unui program, precum și complexitatea sa, ne vor scăpa de sub control. Menținerea programelor sub control este cea mai mare problemă în programare. Când un program funcționează, totul este foarte frumos. Arta programării este abilitatea de a controla complexitatea. Marele program este subordonat - simplificat din punct de vedere al complexității sale.

{{index "programming style", "best practices"}}

Unii programatori cred ca această complexitate este gestionată cel mai bine prin utilizarea unui set restrâns de tehnici bine cunoscute în programele lor. Ei au construit reguli stricte ("best practices") care prescriu forma pe care trebuie să o aibă programele și stau cu mare atenție în mica lor zonă de securitate.

{{index experiment}}

Aceasta nu este doar o practică plictisitoare, este și una ineficientă. Probleme noi solicită adesea soluții noi. Domeniul programării este unul tânăr și încă se dezvoltă rapid și este suficient de variat pentru a avea loc pentru abordări complet diferite. Există multe greșeli teribile care pot să apară în conceperea programelor și ar trebui să le faceți pentru a le înțelege cu adevărat. Bunul simț cu privire la cum trebuie să arate un program bun se dezvoltă prin practică, nu prin învățarea unei liste de reguli.

## De ce limbajul contează

{{index "programming language", "machine code", "binary data"}}

La început, atunci când se năștea "computing"-ul, nu existau limbaje de programare. Programele arătau cam așa:

```{lang: null}
00110001 00000000 00000000
00110001 00000001 00000001
00110011 00000001 00000010
01010001 00001011 00000010
00100010 00000010 00001000
01000011 00000001 00000000
01000001 00000001 00000001
00010000 00000010 00000000
01100010 00000000 00000000
```

{{index [programming, "history of"], "punch card", complexity}}

Acesta este un program care adună numerele de la 1 la 10 și afișează rezultatul: `1 + 2 + ... + 10 = 55`. Ar putea rula pe o mașină simplă, ipotetică. Pentru a programa computerele de la începuturi, era necesar să se seteze în poziția corectă un număr mare de comutatoare sau să se practice goluri în carduri din carton, carduri care apoi urmau să fie încărcate în computer. Cred că vă puteți imagina cât de plictistoare și supusă erorilor era întreaga procedură. Chiar și scrierea unui program simplu necesita multă inteligență și disciplină. Cele complexe erau aproape de neconceput.

{{index bit, "wizard (mighty)"}}

Desigur, introducerea manuală a acestora șabloane ciudate de biți (valori 0 sau 1) îi insufla programatorului un profund sentiment de a fi un vrăjitor atot-puternic. Și asta cu siguranță însemna ceva din punct de vedere al satisfacției la locul de muncă.

{{index memory, instruction}}

Fiecare linie a programului anterior conține o singură instrucțiune. Ar putea fi descrisă în limbaj natural astfel:

 1. Memorează numărul 0 în locația 0.
 2. Memorează numărul 1 în locația 1.
 3. Memorează valoarea din locația de memorie 1 în locația de memorie 2.
 4. Scade numărul 11 din valoarea stocată in locația de memorie 2.
 5. Dacă valoarea din locația de memorie 2 este numărul 0, continuă cu instrucțiunea 9.
 6. Adaugă valoarea din locația de memorie 1 la locația de memorie 0.
 7. Adaugă numărul 1 la valoarea din locația de memorie 1.
 8. Continuă cu instrucțiunea 3.
 9. Afișează valoarea din locația de memorie 0.

{{index readability, naming, binding}}

Deși această exprimare este mult mai lizibilă decât "ciorba de biți" inițială, este încă o exprimare destul de obscură. Utilizarea numelor în loc de numere pentru locațiile de memorie, ne ajută și mai mult.

```{lang: "text/plain"}
 Setează “total” la valoarea 0.
 Setează “count” la valoarea 1.
[loop]
 Setează “compare” la valoarea lui “count”.
 Scade 11 din “compare”.
 Dacă “compare” este 0, continuă la eticheta [end].
 Adaugă “count” la “total”.
 Adaugă 1 la “count”.
 Continuă la eticheta [loop].
[end]
 Afișează “total”.
```

{{index loop, jump, "summing example"}}

Puteți înțelege acum cum funcționează programul? Primele două instrucțiuni atribuie valorile de inițializare pentru două locații de memorie: `total` va fi utilizat pentru a construi rezultatul calculului, iar `count` va ține evidența numărului curent ce urmează a fi prelucrat. Liniile care utilizează `compare` sunt probabil cele mai ciudate. Programul vrea să verifice dacă `count` este egal cu 11 pentru a decide dacă poate să își încheie execuția. Deoarece mașina noastră ipotetică este destul de primitivă, poate doar să testeze dacă un număr este egal cu zero și să ia o decizie pe această bază. Astfel că utilizăm locația numită `compare` pentru a calcula valoarea expresiei `count-11` și a lua decizii în funcție de această valoare. Următoarele două linii adaugă valoarea lui `count` la rezultatul final și apoi incrementează `count` cu 1 pentru că programul a "decis" că încă nu trebuie să se oprească.

Iată cum arată același program scris in JavaScript:

``` {lang: "javascript"}
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
// → 55
```

{{index "while loop", loop, [braces, block]}}

Această versiune aduce câteva îmbunătățiri. Cea mai importantă, nu mai avem nevoie să specificăm felul în care vrem să sară programul înainte și înapoi. Construcția `while` are grijă de această repetiție. Se continuă execuția blocului plasat între acolade atâta timp cât condiția precizată rămâne adevărată. Condiția este `count <= 10` care înseamnă "_count_ este mai mic sau egal cu 10". Nu mai avem nevoie să creem o variabilă temporară pe care să o comparăm cu zero, ceea ce era doar un detaliu neinteresant. O parte din puterea limbajelor de programare constă în faptul că pot avea grijă de detaliile neinteresante în locul nostru.

{{index "console.log"}}

La sfârșitul programului, după ce s-a terminat execuția construcției `while`, operația `console.log` este utilizată pentru a afișa rezultatul.

{{index "sum function", "range function", abstraction, function}}

În sfârșit, iată cum am fi putut scrie programul dacă am fi avut la dispoziție operațiile convenabile `range` (care să genereze o colecție de numere între anumite limite) și `sum` care să calculeze suma valorilor dintr-o colecție de numere:

```{startCode: true}
console.log(sum(range(1, 10)));
// → 55
```

{{index readability}}

Morala acestei povestiri este că același program poate fi exprimat în formă lungă sau scurtă, într-un mod neinteligibil sau ușor de citit. Prima versiune a programului a fost extrem de obscură, în timp ce ultima versiune este aproape limbaj natural: `log` `sum` `range` pentru numerele de la 1 la 10. Vom vedea în capitolele următoare cum putem defini operații cum ar fi `sum` și `range`.

{{index ["programming language", "power of"], composability}}

Un limbaj de programare bun ajută programatorului, permimțându-i să se exprime cu privire la acțiunile pe care trebuie să le execute programatorul la un nivel mai înalt. Ajută la omiterea detaliilor, oferă blocuri convenabile de construcție (cum ar fi `while` și `console.log`), permite definirea propriilor blocuri de cosntrucție (cum ar fi `sum` și `range`) și permite compunerea cu ușurință a blocurilor respective.

## Ce este JavaScript?

{{index history, Netscape, browser, "web application", JavaScript, [JavaScript, "history of"], "World Wide Web"}}

{{indexsee WWW, "World Wide Web"}}

{{indexsee Web, "World Wide Web"}}

JavaScript a fost introdus în 1995 ca o modalitate de a adăuga programe în paginile web, în browserul Netscape Navigator. De atunci, limbajul a fost adoptat de către toate celelalte browsere grafice majore. A făcut posibilă crearea aplicațiilor web moderne - aplicații cu care interacționați direct, fără a fi nevoie să reîncărcați pagina de fiecare dată. JavaScript este utilizat și în site-urile web mai tradiționale pentru a adăuga interactivitate si un comportament mai inteligent al site-ului.

{{index Java, naming}}

Este important să menționăm că JavaScript nu are aproape nimic în comun cu limbajul de programare numit Java. Similaritatea numelor a fost inspirată de aspecte legate de marketing și nu de o judecată atentă. Când a fost introdus JavaScript, limbajul Java era promovat intens și câștiga popularitate. Cineva s-a gândit că ar fi o idee bună să profite de acest succes. Și acum am rămas cu această confuzie de nume.

{{index ECMAScript, compatibility}}

După adoptarea sa în afara Netscape, a fost elaborat un document standard care să descrie modul în care limbajul JavaScript trebuie să funcționeze astfel încât diversele soft-uri care pretindeau că suporta JavaScript să se refere de fapt la același limbaj.Acesta este standardul ECMAScript, după numele organizației Ecma International care a realizat standardizarea. În practică, denumirile ECMAScript și JavaScript pot fi ambele utilizate - ele sunt două nume diferite ale aceluiași limbaj de programare.

{{index [JavaScript, "weaknesses of"], debugging}}

Unii vă vor spune lucruri teribile despre JavaScript. Multe dintre acestea sunt adevărate. Atunci când a trebuit să scriu cod JavaScript pentru prima dată, am ajuns repede să îl desconsider. Accepta aproape orice tastam, dar interpreta într-un mod complet diferit de ceea ce intenționam eu să fac. Însă cu siguranță exista și o legătura cu faptul că nu aveam nici cea mai mică idee despre ceea ce făceam, dar există și o problemă reală aici: JavaScript este extraordinar de tolerant în ceea ce permite. Ideea din spatele acestui design a fost aceea de a face programarea în JavaScript mai ușoară pentru începători. În realitate însă, îngreunează procesul de identificare a problemelor deoarece sistemul nu le va identifica.

{{index [JavaScript, "flexibility of"], flexibility}}

Această flexibilitate are și avantaje. Permite spațiu pentru o mulțime de tehnici care sunt imposibil de implementat în alte limbaje mai rigide și, așa cum veți vedea în Capitolul 10, o putem utiliza pentru a depăși unele dintre dezavantajele JavaScript. După ce am invățat în mod corespunzător limbajul și am lucrat cu el pentru o vreme, mi-am dat seama că, de fapt, _îmi place_ JavaScript.

{{index future, [JavaScript, "versions of"], ECMAScript, "ECMAScript 6"}}

Au existat mai multe versiuni ale JavaScript. ECMAScript versiunea 3 era suportat pe larg în perioada de ascensiune a limbajului JavaScript către poziția dominantă de azi (în mare, între 2000 și 2010). În această perioadă, se lucra activ la o ambițioasă versiune 4 care includea în plan un număr de imbunătățiri radicale și extensii ale limbajului. Modificarea unui limbaj utilizat pe scară largă într-o manieră atât de radicală s-a demonstrat a fi politic dificilă și din acest motiv versiunea 4 a fost abandonată în 2008, trecându-se la mult mai puțin ambițioasa versiune 5, care a făcut doar câteva îmbunătățiri necontroversate, versiune publicată în 2009. Apoi, în 2015, a apărut versiunea 6, o actualizare majoră ce includea unele dintre ideile planificate pentru versiunea 4. De atunci, în fiecare an am avut noi actualizări.

Faptul că limbajul evoluează înseamnă că browserele trebuie în permanență să evolueze și ele, iar dacă utilizați un browser mai vechi, s-ar putea să nu suporte fiecare caracteristică. Designerii limbajului au mare grijă să nu introducă modificări care ar putea strica programele mai vechi, astfel încât noile browsere să poată rula programele mai vechi. Această carte folosește versiunea 2017 a limbajului JavaScript.

{{index [JavaScript, "uses of"]}}

Browserele web nu sunt singurele platforme în care se utilizează JavaScript. Unele baze de date, cum ar fi MongoDB sau CouchDB utilizează JavaScript ca și limbajul lor de scripting și interogare. Câteva platforme pentru programarea desktop și server, dintre care cel mai notabil este proiectul NodeJS (subiectul Capitolului 20), furnizează un mediu de programare JavaScript în afara browserului.

## Codul și ce să facem cu el

{{index "reading code", "writing code"}}

_Codul_ este textul care reprezintă un program. Majoritatea capitolelor din această carte conțin o cantitate destul de mare de code. Cred că citirea și scrierea codului sunt indispensabile pentru a învăța cum să programăm. Încercați nu doar să aruncați o privire asupra exemplelor ci să le citiți atent și să încercați să le înțelegeți. Aceasta va fi o activitate lentă si cu multă confuzie, la început, dar vă promit că în curând o să fiți mult mai comfortabili. Tot la fel stau lucrurile și în privința exercițiilor. Nu presupuneți că le-ați înțeles decât în momentul în care ați reușit să creați o soluție care funcționează.

{{index interpretation}}

Vă recomand să încercați să rezolvați excercițiile cu ajutorul unui interpretor de JavaScript real. În felul acesta veți primi feedback imediat cu privire la funcționarea codului pe care îl scrieți și, sper, veți fi tentați să experimentați dincolo de ceea ce vă cere exercițiul.


{{if interactive

Când citiți această carte într-un browser, puteți edita și rula toate exemplele. Executați un click pe codul exemplificat.
if}}

{{if book

{{index download, sandbox, "running code"}}

The easiest way to run the example code in the book, and to experiment
with it, is to look it up in the online version of the book at
[_https://eloquentjavascript.net_](https://eloquentjavascript.net/). There,
you can click any code example to edit and run it and to see the
output it produces. To work on the exercises, go to
[_https://eloquentjavascript.net/code_](https://eloquentjavascript.net/code),
which provides starting code for each coding exercise and allows you
to look at the solutions.

if}}

{{index "developer tools", "JavaScript console"}}

Dacă vreți să rulați programele din această carte în afara website-ului, trebuie să țineți cont de câteva atenționări. Multe exemple sunt complete și ar trebui să funcționeze în orice mediu JavaScript. Dar codul în capitolele care abordează subiecte mai avansate este scris pentru un anume mediu (browserul sau NodeJS) și poate rula doar în acel mediu. În plus, mai multe capitole definesc programe mai mari și bucățile de cod prezentate depind una de alta sau de fișiere externe. [sandbox](https://eloquentjavascript.net/code) din site-ul original conține linkuri către toate scripturile și fișierele de date necesare pentru a rula codul dintr-un anume capitol.

## Prezentare generală a acestei cărți

Acestă carte conține trei părți. Primele 12 capitole prezintă limbajul JavaScript. Următoarele 7 capitole sunt despre browserele web și modul în care JavaScript poate fi utilizat pentru programarea lor. In final, ultimele două capitole sunt dedicate NodeJS, un alt mediu în care putem realiza programe JavaScript.

Pe parcursul cărții, veți descoperi 5 _capitole de proiecte_ care vor prezenta programe de dimensiuni mai mari și vă vor da o idee despre programarea reală. În ordinea apariției, vom lucra la construcția unui _robot pentru livrări_, a unui _limbaj de programare_, a unui _joc de platformă_, a unui _program de desenare cu pixeli_ și, în final, a unui _site web dinamic_.

Prima parte a cărții începe cu patru capitole care introduc structura de bază a limbajului JavaScript. Ele vă introduc în [structurile de control](program_structure) (cum ar fi `while` despre care ați aflat în această introducere), [funcții](functions) (cum să scrieți propriile blocuri de construcție) și [structuri de date](data). După parcurgerea lor veți fi capabili să scrieți programe simple. Apoi, capitolele [?](higher_order) și [?](object) vă vor introduce tehnici de utilizare a funcțiilor și obiectelor pentru a scrie cod mai _abstract_ și a păstra complexitatea sub control.

După un prim [proiect de capitol](robot), prima parte continuă cu capitole despre [gestiunea erorilor și repararea bugurilor](error), [expresii regulate](regexp) (un instrument important pentru lucrul cu text), [modularitatea](modules) (o altă metodă de apărare împotriva complxității) și [programarea asincronă](async) (gestiunea evenimentelor de lungă durată). [Al doilea proiect de capitol](language) finalizează prima parte a cărții.

In a doua parte a cărții, capitolele de la [?](browser) până la [?](paint) descriu instrumente la care JavaScript are acces în browser. Veți învăța să afișați elemente pe ecran (capitolele [?](dom) și [?](canvas)), să răspundeți la acțiunile utilizatorului ([?](event)) și să comunicați în rețea ([?](http)). Și în această parte există două capitole de proiect.

După aceea, capitolul [?](node) descrie NodeJS și capitolul [?](skillsharing) construiește un mic website cu ajutorul acestui instrument.

{{if commercial

Finally, [Chapter ?](fast) describes some of the considerations that
come up when optimizing JavaScript programs for speed.

if}}

## Convenții tipografice

{{index "factorial function"}}

În această carte, textul scris cu font `monospaced` va reprezenta elemente ale unor programe - uneori, sunt fragmente complet funcționale, alteori se referă doar la o anumită parte a unui program. Programele sunt scrise după cum urmează:

```
function factorial(n) {
  if (n == 0) {
    return 1;
  } else {
    return factorial(n - 1) * n;
  }
}
```

{{index "console.log"}}

Uneori, pentru a afișa ieșirea pe care o produce un program, rezultatul este prezentat la sfârșitul programului, ca în exemplul de mai jos:

```
console.log(factorial(8));
// → 40320
```

Mult succes!
