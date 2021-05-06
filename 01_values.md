{{meta {docid: values}}}

# Valori, tipuri și operatori

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Dincolo de suprafața mașinii, programul se mișcă. Fără efort, se dilată și se contractă. Într-o armonie perfectă, electronii se împrăștie și se regrupează. Formele de pe monitor sunt doar niște unduiri pe apă. Esența rămâne invizibilă.

quote}}

{{index "Yuan-Ma", "Book of Programming"}}

{{figure {url: "img/chapter_picture_1.jpg", alt: "Picture of a sea of bits", chapter: framed}}}

{{index "binary data", data, bit, memory}}

În lumea computerului există doar date. Puteți citi date, puteți să le modificați, puteți să creați date noi - dar ceea ce nu reprezintă date nu poate fi menționat. Toate aceste date sunt memorate sub forma unor secvențe lungi de biți și sunt, prin urmare, fundamental identice.

{{index CD, signal}}

_Biții_ sunt orice obiecte care pot reprezenta două valori. De regulă, sunt descriși cu ajutorul valorilor 0 și 1. În interiorul computerului, ei iau forma unor sarcini electrice ridicate sau scăzute, a unui semnal puternic sau slab, sau a unui punct strălucitor sau mat pe suprafața unui CD/DVD. Orice informație discretă poate fi redusă la o secvență de zero și unu și deci, reprezentată ca o succesiune de biți.

{{index "binary number", radix, "decimal number"}}

De exemplu, putem reprezenta numărul 13 pe biți. Funcționează ca și în reprezentarea zecimală, doar că în loc de 10 cifre diferite, avem la dispoziție doar două și ponderea fiecărei cifre este o putere a lui 2, crescătoare de la dreapta la stânga. Exemplul de mai jos prezintă valorile fiecăruia dintre primii 8 biți pe care i-am putea folosi pentru reprezentarea numărului 13, precum și puterile lui 2 asociate fiecărui bit (rândul al doilea):

```{lang: null}
   0   0   0   0   1   1   0   1
 128  64  32  16   8   4   2   1
```

Deci, numărul binar 00001101 are cifre nenule pentru 8, 4 și 1, care adunate ne dau valoarea 13.

## Valori

{{index [memory, organization], "volatile data storage", "hard drive"}}

Imaginați-vă o mare de biți, sau chiar un ocean. Un computer modern are mai mult de 30 miliarde de biți în memoria sa volatilă (memoria de lucru). Memoria de stocare non-volatilă (hard-disk sau echivalentul său) tinde să conțină cu câteva ordine de mărime mai mulți biți.

Pentru a putea lucra cu un număr atât de mare de biți fără a ne pierde, trebuie să îi separăm în bucăți care reprezintă componente ale informației. Într-un mediu JavaScript, aceste bucăți se numesc _valori_. Deți toate valorile sunt formate din biți, ele au roluri diferite. Fiecare valoare are un _tip_ care îi determină rolul. Unele valori sunt numere, altele sunt texte, altele sunt funcții ...

{{index "garbage collection"}}

Pentru a crea o valoare trebuie doar să îi invocați numele. Ceea ce este foarte convenabil. Nu trebuie să adunați material de construcție pentru valorile de care aveți nevoie și nici să plătiți pentru ele. Doar solicitați una si _boom_, vă este disponibilă. Desigur, ele nu sunt create prin magie. Fiecare valoare trebuie să fie stocată undeva și dacă vă gândiți să utilizați un număr uriaș de valori simultan, s-ar putea să rămâneți fără memorie. Din fericire, aceasta este o problemă doar dacă intenționați să aveți disponibile toate valorile simultan. Imediat ce nu mai aveți nevoie de o valoare, aceasta va dispărea, iar biții care au fost utilizați pentru reprezentarea ei vor fi reciclați ca și material pentru construcția următoarei generații de valori. 

Acest capitol vă introduce elementtele atomice ale programelor JavaScript, adică tipurile simple de valori și operatorii care pot acționa asupra acestor valori.

## Numere

{{index [syntax, number], number, [number, notation]}}

Valorile de tip _Number_ sunt valori numerice. Într-un program JavaScript ele sunt scrise după cum urmează:

```
13
```

{{index "binary number"}}

Prin utilizarea valorii într-un program, va apărea șablonul de biți al numărului 13 în memoria computerului.

{{index [number, representation], bit}}

JavaScript utilizează un număr fix de biți, 64, pentru a stoca fiecare valoare numerică în memorie. Numărul de șabloane care pot fi construite cu 64 biți, ceea ce înseamnă că numărul de valori care pot fi reprezentate este limitat. Folosind _N_ zecimale (cifre) putem reprezenta 10^N^ numere diferite. Similar, cu 64 de cifre binare putem reprezenta 2^64^ numere diferite, ceea ce reprezintă aproximativ 18 quintilioane (18 urmat de 18 zerouri). Ceea ce este foarte mult.

Memoria computerelor era mult mai mică și programatorii utilizau grupuri de 8 sau 16 biți pentru a reprezenta numerele. Era relativ ușor să se atingă accidental un _overflow_ cu numere atât de mici - adică să se ajungă la un număr care nu "încape" în numărul dat de biți. Azi, chiar și un computer de buzunar are o cantitate mare de memorie astfel încât putem folosi bucăți de 64 biți și trebuie să ne preocupe eroarea de depășire (_overflow_) doar dacă avem de a face cu valori cu adevărat astronomice.

{{index sign, "floating-point number", "sign bit"}}

Nu toate numerele mai mici decât 18 quintilioane pot fi reprezentate ca si numere in JavaScript. Biții stochează și numere negative, deci un bit indică semnul numărului. O problemă și mai mare este că trebuie să putem reprezenta și numere reale. Pentru aceasta, o parte dintre biți sunt utilizați pentru a memora poziția punctului zecimal. În realitate, cel mai mare număr întreg care poate fi reprezentat este în jurul valorii de 9 cvadrilioane (15 zerouri) ceea ce încă este foarte mult.

{{index [number, notation], "fractional number"}}

Numerele fracționare sunt reprezentate prin folosirea unui punct.

```
9.81
```

{{index exponent, "scientific notation", [number, notation]}}

Pentru numere foarte mari sau foarte mici, putem folosi notația științifică prin adăugarea unui _e_ ( de la _exponent_), urmat de exponentul numărului.

```
2.998e8
```

Aceasta este valoarea 2.998 × 10^8^ = 299,800,000.

{{index pi, [number, "precision of"], "floating-point number"}}

Calculele cu numere întregi (numite și _integers_) mai mici decât valoarea menționată de 9 cvadrilioane sunt garantate a fi întotdeauna exacte. Din păcate, aceeași garanție nu este valabilă pentru numerele fracționare, în general. Așa cum numărul π (pi) nu poate fi scris exact cu un număr finit de zecimale, multe numere pierd din precizie atunci când trebuie să fie reprezentate cu doar 64 biți. Este păcat, dar acesta cauzează probleme practice doar în anumite situații. Ceea ce este important, trebuie să conștientizați această problemă și să tratați numerele fracționare ca și aproximări, nu ca și valori exacte.

### Aritmetica

{{index [syntax, operator], operator, "binary operator", arithmetic, addition, multiplication}}

Cele mai frecvente operații cu numere sunt operațiile aritmetice. Aceste operații, cum ar fi adunarea sau înmulțirea iau două numere și produc un număr nou din ele. Iată cum se folosesc aceste operații în JavaScript:

```
100 + 4 * 11
```

{{index [operator, application], asterisk, "plus character", "* operator", "+ operator"}}

Simbolurile  `+` și `*` se numesc _operatori_. Primul este folosit pentru adunare iar cel de al doilea pentru înmulțire. Plasarea unui operator între două valori va aplica acel operator asupra celor două valori pentru a produce o nouă valoare.

{{index grouping, parentheses, precedence}}

Oare înseamnă exemplul anterior că "adunăm 4 cu 100 și apoi multiplicăm rezultatul cu 11" sau de fapt se va efectua înmulțirea înainte de adunare? După cum probabil v-ați dat seama, mai întâi se efectuează operația de înmulțire. Dar, ca și în matematică, puteți modifica acest comportament prin utilizarea parantezelor:

```
(100 + 4) * 11
```

{{index "hyphen character", "slash character", division, subtraction, minus, "- operator", "/ operator"}}

Pentru scădere se utilizează operatorul `-` iar împărțirea se poate efectua folosind operatorul `/`.

Când operatorii apar într-o expresie fără paranteze, ordinea în care aceștia se aplică este determinată de către _precedența_ operatorilor. Exemplul arată că înmulțirea se efectuează înainte de adunare. Operatorul `/` are aceeași precedență ca și operatorul `*`. Similar pentru `+` și `-`. Atunci când mai mulți operatori cu aceeași precedență apar Într-o expresie fără paranteze, cum ar fi `1 - 2 + 1`, aceștia se aplică asupra valorilor de la stânga la dreapta, ca și în: `(1 - 2) + 1`.

Nu trebuie să vă faceți prea multe griji despre regulile de precedență. Când aveți dubii, folosiți parantezele.

{{index "modulo operator", division, "remainder operator", "% operator"}}

Mai există încă un operator pe care probabil nu îl veți recunoaște imediat. Simbolul `%` este utilizat pentru a calcula _restul împărțirii a două numere întregi_. `X % Y` reprezintă restul împărțirii lui `X` la `Y`. De exemplu, `314 % 100` are rezultatul `14`, iar `144 % 12` este `0`. Prioritatea acestui operator este aceeași ca și a operatorilor de înmulțire și împărțire. De multe ori acest operator este denumit _modulo_.

### Numere speciale

{{index [number, "special values"]}}

Există trei valori speciale în JavaScript care sunt considerate numere dar nu se comportă ca și numerele normale.

{{index infinity}}

Primele două sunt `Infinity` și `-Infinity`, care reprezintă infinitul pozitiv, respectiv negativ.`Infinity - 1` este tot `Infinity`. Nu vă încredeți prea mult în calcule bazate pe infinit. Nu prea sună bine din punct de vedere al matematicii și vor conduce destul de repede la celălalt număr special: `NaN`.

{{index NaN, "not a number", "division by zero"}}

`NaN` reprezintă prescurtarea de la "not a number", deși _este_ o valoare de tip numeric. Veți obține acest rezultat, de exemplu, atunci când încercați să calculați `0 / 0` (zero împărțit la zero), `Infinity - Infinity`, sau orice altă operație numerică ce nu conduce la un rezultat cu sens.

## Stringuri

{{indexsee "grave accent", backtick}}

{{index [syntax, string], text, character, [string, notation], "single-quote character", "double-quote character", "quotation mark", backtick}}

Următorul tip de date de bază este _String_. Stringurile sunt utilizate pentru a reprezenta text. Ele sunt scrise prin includerea conținutului lor între simboluri de citare:

```
`Down on the sea`
"Lie on the ocean"
'Float on the ocean'
```

Puteți utiliza apostroafe, ghilimele sau backticks, cu singura restricție ca simbolul de start și cel de sfârșit să fie aceleași.

{{index "line break", "newline character"}}

Aproape orice poate fi plasat între simboluri de citare și JavaScript îl va transforma într-un string. Există însă câteva caractere care fac excepție de la regulă. Vă puteți imagina că nu poate fi chiar așa de simplu să plasăm simboluri de citare în interiorul unui string. De asemenea, caracterul _linie nouă_ (cel pe care îl obțineți apăsând tasta Enter) poate fi inclus fără secvență-escape doar atunci când folosim backticks.

{{index [escaping, "in strings"], ["backslash character", "in strings"]}}

Pentru a putea introduce asemenea caractere într-un string, se utilizează următoarea notație: întotdeauna când este prezent un caracter backslash `\` în interiorul unui text, se precizează că după acest simbol urmează un caracter cu semnificație specială. Acestă notație se numește _escaping_. Un simbol de citare precedat de backslash nu are semnificația de terminare a stringului ci va face parte din string. Un caracter `n` dupa backslash este interpretat ca o linie nouă. Similar, un caracter `t` după un backslash reprezintă un caracter "tab". Să considerăm următorul string:

```
"This is the first line\nAnd this is the second"
```

În realitate, acesta conține următorul text:

```{lang: null}
This is the first line
And this is the second
```

Există situații în care un backslash într-un string trebuie să fie doar un backslash, nu un caracter special. Dacă plasăm doua caractere backslash unul după altul, ele vor fi colapsate și doar unul dintre ele rămâne ca și caracter normal în stringul rezultat. Iată cum putem scrie stringul "_A newline
character is written like `"`\n`"`._":

```
"A newline character is written like \"\\n\"."
```

{{id unicode}}

{{index [string, representation], Unicode, character}}

Și stringurile trebuie modelate ca o succesiune de biți pentru a putea exista în interiorul unui computer. În JavaScript, pentru aceasta se folosește standardul _Unicode_. Acest standard asociază un număr fiecărui caracter de care ați avea nevoie vreodată. Dacă avem câte un număr asociat fiecărui caracter, un string poate fi descris ca o secvență de numere.

{{index "UTF-16", emoji}}

Asta se întâmplă în JavaScript. Dar există o complicație: reprezentarea JavaScript utilizează 16 biți pentru fiecare element al unui string, ceea ce înseamnă că pot fi descrise până la 2^16^ caractere. Dar Unicode mai multe caractere, cam de două ori mai multe, în acest moment. Astfel că, unele caractere, cum ar fi multe emoji, folosesc "două caractere" în stringul JavaScript. Vom reveni asupra acestui aspect în capitolul [?](higher_order#code_units).

{{index "+ operator", concatenation}}

Stringurile nu pot fi împărțite, multiplicate sau scăzute, dar operatorul `+` _poate_ fi folosit. Acesta nu adună ci _concatenează_, adică alipește două stringuri. Următoarea linie va produce stringul  `"concatenate"`:

```
"con" + "cat" + "e" + "nate"
```

Valorile de tip string au mai multe funcții asociate (_metode_) care pot fi utilizate pentru efectuarea unor operații. Mai multe despre acestea în capitolul [?](data#methods).

{{index interpolation, backtick}}

Stringurile definite cu apostroafe sau ghilimele se comportă la fel - singura diferență este dată de caracterul pentru care trebuie aplicat "escape" în interiorul lor. Stringurile plasate între backticks, de regulă numite _template literals_, pot să facă și alte lucruri. Pe lângă abilitatea de a se întinde pe mai multe linii, ele pot să includă alte valori.

```
`half of 100 is ${100 / 2}`
```

Atunci când scriem ceva între `${}` într-un template literal, se va calcula rezultatul acelei expresii, care apoi va fi convertit într-un string și inclus în poziția respectivă. Exemplul va produce stringul "_half of 100 is 50_".

## Operatori unari

{{index operator, "typeof operator", type}}

Nu toți operatorii sunt simboluri. Unii dintre ei se exprimă prin cuvinte. Un exemplu este operatorul `typeof`, care produce o valoare de tip string ce reprezintă tipul valorii primite

```
console.log(typeof 4.5)
// → number
console.log(typeof "x")
// → string
```

{{index "console.log", output, "JavaScript console"}}

{{id "console.log"}}

Vom utiliza `console.log` în exemplele de cod pentru a arăta că vrem să afițăm rezultatul evaluării unei anumite expresii. Mai multe despre această intsrucțiune în [capitolul următor](program_structure).

{{index negation, "- operator", "binary operator", "unary operator"}}

Toți operatorii prezentați anterior operau asupra a două valori, însă `typeof` operează doar asupra unei singure valori. Operatorii care operează asupra a două valori se numesc _operatori binari_, iar cei care acționează asupra unei singure valori se numesc _operatori unari_. Operatorul `-` poate fi utilizat atât ca operator binar cât și ca operator unar.


```
console.log(- (10 - 2))
// → -8
```

## Valori booleene

{{index Boolean, operator, true, false, bit}}

Este util să avem o valoare care dinstinge între doar două posibilități, cum ar fi "da" și "nu" sau "pornit" și "oprit". Pentru acest scop, JavaScript pune la dispoziție tipul _Boolean_, care are doar două valori, `true` și `false`, utilizate exact cum le-am scris.

### Comparații

{{index comparison}}

Iată un exemplu de a produce valori booleene:

```
console.log(3 > 2)
// → true
console.log(3 < 2)
// → false
```

{{index [comparison, "of numbers"], "> operator", "< operator", "greater than", "less than"}}

Simbolurile `>` și `<` sunt simbolurile tradiționale pentru "mai mare decât" și "mai mic decât". Aceștia sunt operatori binari. Aplicarea lor conduce la o valoare booleană care, în acest caz precizează daca rezultatul comparației a fost adevărat sau fals.

Stringurile pot fi comparate în același mod.

```
console.log("Aardvark" < "Zoroaster")
// → true
```

{{index [comparison, "of strings"]}}

Modul în care stringurile sunt ordonate este în principiu alfabetic, dar nu exact ceea ce ați aștepta conform unui dicționar: literele mari sunt întotdeauna "mai mici" decât literele mici, `"Z" < "a"`, iar caracterele non-alfabetice (!, -, ...) sunt și ele incluse în ordonare. Când compară stringuri, JavaScript merge de la stânga la dreapta, caracter cu caracter, comparând codurile Unicode.

{{index equality, ">= operator", "<= operator", "== operator", "!= operator"}}

Alți operatori similari sunt `>=` (mai mare sau egal), `<=` (mai mic sau egal), `==` (egal cu), `!=` (diferit)

```
console.log("Itchy" != "Scratchy")
// → true
console.log("Apple" == "Orange")
// → false
```

{{index [comparison, "of NaN"], NaN}}

O singură valoare din JavaScript nu este egală cu ea însăși, și anume `NaN` ("not a number").

```
console.log(NaN == NaN)
// → false
```

`NaN` se presupune că simbolizează rezultatul unui calcul fără sens și, prin urmare, nu este egal cu rezultatul nici unui alt calcul fără sens.

### Operatori logici

{{index reasoning, "logical operators"}}

Există și operații care se pot aplica asupra valorilor Booleene. JavaScript oferă trei operatori logici pentru aceasta: _and_, _or_ și _not_. Aceștia pot fi utilizați pentru logica booleană.

{{index "&& operator", "logical and"}}

Operatorul `&&` reprezintă _și_ logic. Este un operator binar și rezultatul său este `true` dacă și numai dacă ambele valori asupra cărora operează sunt `true`.

```
console.log(true && false)
// → false
console.log(true && true)
// → true
```

{{index "|| operator", "logical or"}}

Operatorul `||` reprezintă _sau_ logic. Este tot un operator binar și rezultatul său este `true` dacă cel puțin una dintre cele două valori asupra cărora operează este `true`.

```
console.log(false || true)
// → true
console.log(false || false)
// → false
```

{{index negation, "! operator"}}

_Negația_ se scrie sub forma unui semn de exclamare (`!`). Este un operator unar care schimbă valoarea asupra căreia acționează - `!true` produce `false`, și `!false`
este `true`.

{{index precedence}}

Atunci când combinăm acești operatori booleeni cu operatori aritmetici și de alte tipuri, nu este întotdeauna evident dacă trebuie sau nu să utilizăm paranteze. În practică, de regulă este suficient să știți că, dintre operatorii prezentați până acum, `||` are cea mai mică prioritate, urmat de `&&`, apoi de operatorii de comparare și apoi ceilalți operatori. Această ordine de prioritate a fost aleasă astfel încât în expresiile cel mai frecvent scrise, cum ar fi cea din exemplul de mai jos, să fie posibilă utilizarea unui număr cât mai mic de paranteze.

```
1 + 1 == 2 && 10 * 10 > 50
```

{{index "conditional execution", "ternary operator", "?: operator", "conditional operator", "colon character", "question mark"}}

Ultimul operator logic despre care vom discuta nu este nici unar, nici binar, ci este un operator _ternar_, ce operează asupra a trei valori. Este scris astfel:

```
console.log(true ? 1 : 2);
// → 1
console.log(false ? 1 : 2);
// → 2
```

Acesta este numit operator _condițional_ (și uneori pur și simplu operator _ternar_ deoarece este singurul de acest fel al limbajului JavaScript). Valoarea din stânga simbolului `?` alege care dintre celelalte două valori va fi returnată ca și rezultat. Atunci când prima valoare este `true`, cea de a doua valoare va fi returnată, iar atunci când prima valoare este `false` se va returna cea de a treia valoare.

## Valori "empty"

{{index undefined, null}}

JavaScript definește două valori speciale, scrise `null` și `undefined`, care sunt utilizate pentru a menționa absența unei valori _semnificative_. Ele sunt valori, dar nu transportă informație.

Multe operații ale limbajului care nu produc o valoare semnificativă returnează `undefined` deoarece trebuie să returneze o valoare.

Diferența de semnificație între `undefined` și `null` este un accident de design al JavaScript și de cele mai multe ori, este irelevantă. În cazurile în care trebuie să vă preocupe aceste valori, vă recomand să le tratați ca fiind interschimbabile (echivalente).

## Conversia automată a tipului

{{index NaN, "type coercion"}}

În Introducere, am menționat că JavaScript face tot posibilul pentru a accepta aproape orice program pe care trebuie să îl interpreteze, chiar și programele care fac operații neobișnuite. Pentru a demonstra această afirmația, analizați exemplele de mai jos:

```
console.log(8 * null)
// → 0
console.log("5" - 1)
// → 4
console.log("5" + 1)
// → 51
console.log("five" * 2)
// → NaN
console.log(false == 0)
// → true
```

{{index "+ operator", arithmetic, "* operator", "- operator"}}

Când un operator se aplică asupra tipului "greșit" de valoare, JavaScript va încerca să convertească acea valoare la tipul de care are nevoie, utilizând un set de reguli care de cele mai multe ori nu sunt ceea ce vă doriți. Această operație se numește _type coercion_. Valoarea `null` din prima expresie devine `0`, iar `"5"` din a doua expresie devine `5` (din string în număr). Totuși, n cea de a treia expresie, `+` încercă mai întâi concatenarea stringurilor, nu adunarea numerică, astfel încât `1` este convertit în `"1"` (din număr în string).

{{index "type coercion", [number, "conversion to"]}}

Atunci când o valoare care nu se potrivește cu un număr intr-un mod evident (cum ar fi `"five"` sau `undefined`) este convertită la un număr, veți obține rezultatul `NaN`. Operațiile aritmetice asupra `NaN` produc în continuare `NaN`. Dacă se returnează această valoare într-o instrucțiune la care nu vă așteptați să producă acest rezultat, căutați o conversie accidentală a tipului în expresia respectivă.

{{index null, undefined, [comparison, "of undefined values"], "== operator"}}

Când comparați valori de același tip utilizând `==`, rezultatul este predictibil: veți obține `true` când cele două valori sunt aceleași, cu excepția cazului `NaN`. Dar când tipurile celor două valori diferă, JavaScript utilizează un set complicat și confuz de reguli pentru a determina ceea ce are de făcut. În cele mai multe cazuri, încearcă să convertească una dintre valori la tipul celeilalte valori. Totuși, atunci când una dintre părți produce `null` sau `undefined`, rezultatul va fi true numai dacă și cealaltă parte produce `null` sau `undefined`.

```
console.log(null == undefined);
// → true
console.log(null == 0);
// → false
```

Acest comportament este de multe ori util. Când intenționați să testați dacă o valoare este cu adevărat o valoare și nu `null` sau `undefined`, puteți compara acea valoare cu `null` (folosind operatorul `==` sau `!=`).

{{index "type coercion", [Boolean, "conversion to"], "=== operator", "!== operator", comparison}}

Dar dacă intenționați să testați dacă o valoare se referă exact la valoarea `false`? Expresii cum ar fi `0 == false` și `"" == false` sunt adevărate ca urmare a conversiei automate a tipului. Atunci când vreți să nu se efectueze conversiile automate ale tipului, JavaScript vă pune la dispoziție alți doi operatori: `===` și `!==`. Primul testează dacă cele două valori sunt _exact_ egale iar cel de-al doilea testează dacă sunt diferite, fără a efectua conversii de tip. Prin urmare, `"" === false` este `false`, așa cum ne așteptam.

Vă recomand să utilizași acești operatori de comparare defensiv, pentru a preveni conversii de tip neașteptate care să vă producă bătăi de cap. Dar atunci când sunteți siguri că cele două valori comparate au același tip, nu e nici o diferență dacă utilizați operatorii de comparare mai puțin stricți (`==` și `!=`).

### Evaluarea de scurt-circuit a operatorilor logici

{{index "type coercion", [Boolean, "conversion to"], operator}}

Operatorii logici `&&` și `||` gestionează valorile de tipuri diferite într-un mod ciudat. Ei vor converti valoarea din partea stângă la tipul boolean pentru a decide ce trebuie să facă, dar în funcție de operator și de rezultatul acestei conversii, vor returna fie valoarea din stânga (left-hand value) fie valoarea din dreapta (right-hand value).

{{index "|| operator"}}

Operatorul `||`, de exemplu, va returna valoarea din stânga imediat ce aceasta poate fi convertită la `true` și va returna valoarea din dreapta în caz contrar. Aceasta evaluare are efectul așteptat când valorile sunt Booleene și efectueză operații similare pentru alte tipuri de valori.

```
console.log(null || "user")
// → user
console.log("Agnes" || "user")
// → Agnes
```

{{index "default value"}}

Putem utiliza această funcționalitate pentru a seta valori implicite. Dacă aveți o valoare care ar putea fi "empty", puteți folosi după aceasta `||` și apoi să precizați valoarea implicită. Regulile de conversie a stringurilor și numerelor în valori Booleene stabilesc că `0`, `NaN`, și stringul gol (`""`) sunt considerate `false`, în timp ce toate celelalte valori sunt `true`. Astfel, `0 || -1` produce `-1`, iar `"" || "!?"` returnează `"!?"`.

{{index "&& operator"}}

Operatorul `&&` funcționează similar, dar pentru valoarea `false`. Când valoarea din stânga are semnificația de `false` returnează acea valoarea, iar în caz contrar returnează valoarea din dreapta.

O altă proprietate importantă a acestor operatori este că partea dreaptă se evaluează numai dacă este necesar. În momentul în care evaluarea ajunge la situația `true || X`, indiferent de valoarea lui `X`, rezultatul va fi `true` și `X` nu va fi evaluat. Similar, pentru `false && X`, rezultatul este `false` și `X` va fi ignorat. Aceasta este _evaluarea de scurt-circuit_.

{{index "ternary operator", "?: operator", "conditional operator"}}

Operatorul condițional funcționează similar. Dintre cea de a doua si cea de a treia valoare, doar cea selectată va fi evaluată.

## Rezumat

În acest capitol am analizat patru tipuri de valori în JavaScript: numere, stringuri, booleene si valori nedefinite.

Asemenea valori sunt create prin tastarea numelui lor (`true`, `null`) sau a valorii (`13`, `"abc"`). Puteți combina și transforma valorile cu ajutorul operatorilor. Am venit în contact cu câțiva dintre ei: operatori binari pentru operații aritmetice (`+`, `-`, `*`, `/` și `%`), concatenarea stringurilor (`+`), comparare (`==`, `!=`, `===`, `!==`, `<`, `>`, `<=`, `>=`), si logică (`&&`, `||`), precum și câțiva operatori unari (`-` pentru a schimba semnul valorii unui număr, `!` pentru a nega valorile logice) și `typeof` pentru a determina tipul unei valori), precum și singurul operator ternar al limbajului JavaScript (`?:`) utilizt pentru selectarea condiționată a uneia dintre cele două valori.

Ceea ce am studiat până acum vă oferă suficiente informații pentru a utiliza JavaScript ca și cum ar fi un calculator de buzunar, dar nu mult mai mult de atât. Următorul [capitol](program_structure) va începe să adune aceste cunoștințe împreună, în programe de bază.
