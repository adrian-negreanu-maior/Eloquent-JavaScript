{{meta {load_files: ["code/journal.js", "code/chapter/04_data.js"], zip: "node/html"}}}

# Structuri de date: Object și Array

{{quote {author: "Charles Babbage", title: "Passages from the Life of a Philosopher (1864)", chapter: true}

În două ocazii am fost întrebat, "Vă rog, Mr. Babbage, dacă introduceți în mașină cifrele greșite, va apărea rezultatul corect?" [...] Nu sunt capabil să înțeleg corect confuzia de idei care poate provoca o asemenea întrebare.

quote}}

{{index "Babbage, Charles"}}

{{figure {url: "img/chapter_picture_4.jpg", alt: "Picture of a weresquirrel", chapter: framed}}}

{{index object, "data structure"}}

Numbers, Booleans și Strings sunt atomi cu ajutorul cărora se construiesc structurile de date. Multe tipuri de informație necesită pentru reprezentare mai mulți atomi. _Obiectele_ ne permit să grupăm valori - inclusiv alte obiecte - pentru a construi structuri mai complexe.

Programele pe care le-am construit până acum au fost limitate de faptul că operau doar asupra unor tipuri simple de date. În acest capitol vom introduce structuri de date de bază. La sfârșitul lui veți avea suficiente cunoștințe pentru a începe să scrieți programe utile.

De-a lungul capitolului vom lucra cu un exemplu de program mai mult sau mai puțin realist, vom introduce conceptele pe măsură ce sunt aplicabile problemei curente de rezolvat. Codurile din exemple vor fi adesea construite cu funcții sau bindinguri introduse anterior în text.

{{if book

The online coding ((sandbox)) for the book
([_https://eloquentjavascript.net/code_](https://eloquentjavascript.net/code))
provides a way to run code in the context of a specific chapter. If
you decide to work through the examples in another environment, be
sure to first download the full code for this chapter from the sandbox
page.

if}}

## Veverița-vârcolac

{{index "weresquirrel example", lycanthropy}}

Din când în când, de regulă între 8PM și 10PM, Jacques constată că se transformă într-un mic rozător blănos cu o coadă stufoasă.

Pe de o parte, Jacques este destul de bucuros că nu suferă de licantropie clasică. Transformarea într-o veveriță cauzează mai puține probleme decât transformarea într-un lup. În loc să se îngrijoreze că ar putea să își mănânce accidental vecinul (ceea ce ar fi incomod), se îngrijorează că ar putea fi mâncat de către pisica vecinului. După ce în două ocazii s-a trezit pe o ramură firavă din coroana unui stejar, dezbrăcat și dezorientat, a decis să blocheze ușile și ferestrele camerei sale în timpul nopții și să pună câteva nuci pe podea ca să îl țină ocupat.

Astfel rezolvă problemele cu pisica și copacul. Dar Jacques ar prefera sa scape cu totul din această situație. Aparițiile neregulate ale transformării l-au determinat să suspecteze că ceva declanșează problema. Pentru o vreme, a crezut că transformarea are loc doar în zilele în care a fost prezent lângă un stejar. Dar evitarea stejarilor nu a rezolvat problema.

{{index journal}}

Trecând la o abordare mai științifică, Jacques a început să păstreze un jurnal zilnic al tuturor activităților dintr-o zi și dacă și-a schimbat forma. Astfel speră să identifice condițiile care îi declanșează transformările.

Primul lucru de care are nevoie este o structură de date pentru a stoca aceste informații.

## Mulțimi de date

{{index ["data structure", collection], [memory, organization]}}

Pentru a lucra cu un calup de date digitale, mai întâi trebuie să identificăm un mod de a-l reprezenta în memoria computerului. De exemplu, vrem să reprezentăm colecția de numere 2,3,5,7 și 11.

{{index string}}

Putem fi creativi cu ajutorul unui string. În fond, un string poate avea orice lungime astfel încât putem plasa o mare cantitate de date într-un string. Astfel vom utiliza `"2 3 5 7 11"` ca și reprezentare. Însă este ciudat deoarece va trebui să găsim o modalitate de a extrage cifrele și a le converti înapoi în numere pentru a putea accesa informația inițială.

{{index [array, creation], "[] (array)"}}

Din fericire, JavaScript are un tip de date specific pentru a memora secvențe de valori. Acesta se numește _array_ și se scrie ca o listă de valori, între paranteze drepte, separate prin virgulă.

```
let listOfNumbers = [2, 3, 5, 7, 11];
console.log(listOfNumbers[2]);
// → 5
console.log(listOfNumbers[0]);
// → 2
console.log(listOfNumbers[2 - 1]);
// → 3
```

{{index "[] (subscript)", [array, indexing]}}

Notația pentru referirea elementelor dintr-un array folosește tot paranteze drepte. O pereche de paranteze drepte imediat după o expresie, cu o altă expresie între parantezele drepte, va referi elementul expresiei din stânga care corespunde _indexului_ evaluat de către expresia din paranteze.

{{id array_indexing}}
{{index "zero-based counting"}}

Primul index al unui array este 0, nu 1. Prin urmare, primul element este returnat cu `listOfNumbers[0]`. Numărarea începând din zero are o lungă tradiție în tehnologie și oarecum are sens, dar veți avea nevoie de timp pentru adaptare. Gândiți-vă la index ca și cum ar reprezenta numărul de itemi peste care se sare, numărând de la începutul array-ului.

{{id properties}}

## Proprietăți

{{index "Math object", "Math.max function", ["length property", "for string"], [object, property], "period character", [property, access]}}

Am văzut câteva expresii care arătau suspect, cum ar fi `myString.length` (pentru a determina lungimea unui string) sau `Math.max` (funcția pentru determinarea maximului). Acestea sunt expresii care accesează o _proprietate_ a unei valori. În primul caz, accesăm proprietatea `length` a valorii `myString`. În al doilea caz, accesăm proprietatea numită `max` a obiectului `Math` (care este o colecție de constante și funcții matematice).

{{index [property, access], null, undefined}}

Aproape toate valorile din JavaScript au proprietăți. Excepțiile sunt `null` și `undefined`. Dacă încercați să accesați o proprietate a uneia dintre aceste non-valori, veți obține o eroare.

```{test: no}
null.length;
// → TypeError: null has no properties
```

{{indexsee "dot character", "period character"}}
{{index "[] (subscript)", "period character", "square brackets", "computed property", [property, access]}}

Cele două moduri în care ne putem referi la o proprietate în JavaScript sunt utilizarea unui punct sau a parantezelor drepte. Ambele expresii `value.x` și `value[x]` accesează o proprietate a `value` - dar nu neapărat aceeași proprietate.  Diferența apare din modul în care este interpretat `x`. Atunci când folosim notația cu punct, cuvântul de după punct este numele ad-literam al proprietății. Atunci când folosim  paranteze pătrate, expresia dintre paranteze este _evaluată_ pentru a determina numele proprietății. Deci, `value.x` se referă la proprietatea cu numele "x" în timp ce `value[x]` mai întâi evaluează expresia x și apoi convertește rezultatul într-un string, folosit ca și nume al proprietății.

Deci, dacă știți că numele proprietății care vă interesează este _color_, puteți scrie `value.color`. Dacă vreți să extrageți proprietatea al cărei nume este stocat în bindingul `i` veți scrie `value[i]`. Numele proprietăților sunt stringuri. Ele pot fi orice string însă notația cu punct funcționează doar pentru acele nume care sunt valide ca și nume pentru bindinguri. Prin urmare, dacă vreți să accesați o proprietate numită _2_ sau _John Doe_, trebuie să utilizați paranteze pătrate: `value[2]` sau `value["John Doe"]`.

Elementele unui array sunt memorate ca si proprietăți ale array-ului, folosind numere ca și nume ale proprietăților. Deoarece nu putem folosi notația cu punct pentru numere și oricum folosim de regulă un binding care reține indexul, vom folosi notația cu paranteze pentru a le referi.

{{index ["length property", "for array"], [array, "length of"]}}

Proprietatea `length` a unui array ne returnează numărul de elemente conținute. Numele acestei proprietăți este un nume valid pentru un binding și cunoaștem acest nume, astfel încât vom scrie `array.length` deoarece este mai ușor de scris decât `array["length"]`.

{{id methods}}

## Metode

{{index [function, "as property"], method, string}}

Atât valorile de tip string cât și cele de tip array conțin, pe lângă proprietatea `length`, și alte proprietăți care memorează valori de tip funcție.

```
let doh = "Doh";
console.log(typeof doh.toUpperCase);
// → function
console.log(doh.toUpperCase());
// → DOH
```

{{index "case conversion", "toUpperCase method", "toLowerCase method"}}

Orice string are o proprietate `toUpperCase`. Atunci când este apelată, aceasta va returna o copie a stringului în care toate literele sunt convertite la litere mari. Există și proprietatea `toLowerCase` care funcționează asemănător dar pentru litere mici.

{{index "this binding"}}

Interesant, deși proprietatea to `toUpperCase` este apelată fără nici un argument, funcția are cumva acces la stringul `"Doh"`, valoarea a cărei proprietăți este apelată. Această funcționalitate este descrisă în [Capitolul ?](object#obj_methods).

Proprietățile care conțin funcții sunt de regulă numite _metode_ ale valorii căreia îi aparțin. `toUpperCase` este o metodă a unui string.

{{id array_methods}}

Exemplul care urmează demonstrează două metode utile pentru manipularea array-urilor:

```
let sequence = [1, 2, 3];
sequence.push(4);
sequence.push(5);
console.log(sequence);
// → [1, 2, 3, 4, 5]
console.log(sequence.pop());
// → 5
console.log(sequence);
// → [1, 2, 3, 4]
```

{{index collection, array, "push method", "pop method"}}

Metoda `push` adaugă valori la sfârșitul unui array iar metoda `pop` elimină ultima valoare dintr-un array și o returnează.

{{index ["data structure", stack]}}

Aceste denumiri sunt termeni tradiționali pentru operațiile asupra unei stive. O stivă, în programare, este o structură de date care permite introducerea valorilor (push) și extragerea lor în ordine inversă ordinii de introducere (pop) astfel încât ultimul element adăugat va fi întotdeauna primul extras. Asemenea structuri sunt destul de obișnuite în programare - poate vă amintiți despre stiva de apel a funcțiilor din [capitolul anterior](functions#stack), care este o implementare a aceleiași idei.

## Obiecte

{{index journal, "weresquirrel example", array, record}}

Să revenim la veverița-vârcolac. Un set de intrări zilnice în jurnal poate fi reprezentat sub forma unui array. Dar intrările nu constau doar dintr-un număr sau un string - fiecare intrare trebuie să conțină o listă de activități și o valoare booleană care indică dacă Jacques s-a transformat sau nu într-o veveriță. Ideal, am dori să grupăm toate aceste informații într-o singură valoare și apoi să punem toate acele valori grupate într-un array de intrări în jurnal.

{{index [syntax, object], [property, definition], [braces, object], "{} (object)"}}

Valorile de tip _obiect_ sunt colecții arbitrare de proprietăți. O modalitate de a crea un obiect este utilizarea acoladelor ca și expresie:

```
let day1 = {
  squirrel: false,
  events: ["work", "touched tree", "pizza", "running"]
};
console.log(day1.squirrel);
// → false
console.log(day1.wolf);
// → undefined
day1.wolf = false;
console.log(day1.wolf);
// → false
```

{{index [quoting, "of object properties"], "colon character"}}

În interiorul acoladelor avem o listă de proprietăți separate prin virgulă. Fiecare proprietate are un nume și o valoare, separate prin `:`. Atunci când scriem un obiect pe mai multe linii, indentarea lor ca și în exemplu ajută pentru lizibilitate. Proprietățile ale căror nume nu sunt nume valide pentru bindinguri sau numere valide, trebuie definite ca și stringurile:

```
let descriptions = {
  work: "Went to work",
  "touched tree": "Touched a tree"
};
```

{{index [braces, object]}}

Prin urmare, acoladele au două semnificații în JavaScript. În cadrul unei instrucțiuni compuse, ele marchează începutul și sfârșitul blocului de instrucțiuni. În orice alt context, ele descriu un obiect. Din fericire, rareori este util să începem o instrucțiune cu un obiect între acolade, astfel încât ambiguitatea dintre cele două utilizări nu este o problemă.

{{index undefined}}

Încercarea de a accesa o valoare care nu există va returna valoarea `undefined`.

{{index [property, assignment], mutability, "= operator"}}

Putem asocia o valoare unei proprietăți cu ajutorul operatorului `=`. Această operație va înlocui valoarea proprietății dacă proprietatea deja există sau va crea o nouă proprietate asupra obiectului, dacă nu există deja o proprietate cu numele respectiv.

{{index "tentacle (analogy)", [property, "model of"], [binding, "model of"]}}

Ca să revenim pe scurt asupra modelului nostru cu tentaculele pentru bindinguri, bindingurile pentru proprietăți sunt similare. Ele _adună_ valori, dar alte bindiguri și proprietăți s-ar putea referi la aceleași valori. Putem privi un obiect ca pe o "caracatiță" cu oricâte tentacule, fiecare având tatuat un nume.

{{index "delete operator", [property, deletion]}}

Operatorul `delete` taie unul dintre tentaculele acestei caracatițe. Acesta este un operator unar care, atunci când este aplicat asupra unei proprietăți a obiectului, va elimina acea proprietate din obiect. Aceasta nu este o operație frecvent utilizată, însă este posibilă.

```
let anObject = {left: 1, right: 2};
console.log(anObject.left);
// → 1
delete anObject.left;
console.log(anObject.left);
// → undefined
console.log("left" in anObject);
// → false
console.log("right" in anObject);
// → true
```

{{index "in operator", [property, "testing for"], object}}

Operatorul binar `in`, aplicat asupra unui string sau a unui obiect, vă returnează dacă obiectul are sau nu o proprietate cu acel nume. Diferența dintre a seta o proprietate la valoarea `undefined` și a o șterge este dată de faptul că în primul caz obiectul are în continuare proprietatea respectivă (a cărei valoare nu prezintă prea mult interes) în timp ce în al doilea caz proprietatea nu mai există iar `in` va returna `false`.

{{index "Object.keys function"}}

Pentru a determina numele tuturor proprietăților unui obiect, puteți folosi funcția `Object.keys`. Aceasta primește ca argument un obiect și returnează un array de stringuri - numele proprietăților obiectului.

```
console.log(Object.keys({x: 0, y: 0, z: 2}));
// → ["x", "y", "z"]
```

Există și o funcție `Object.assign` care copiază toate proprietățile unui obiect într-un alt obiect.

```
let objectA = {a: 1, b: 2};
Object.assign(objectA, {b: 3, c: 4});
console.log(objectA);
// → {a: 1, b: 3, c: 4}
```

{{index array, collection}}

Array-urile sunt doar niște obiecte specializate pentru a stoca secvențe de elemente. Dacă evaluați `typeof []` rezultatul va fi `"object"`. Le puteți privi ca pe niște caracatițe plate cu toate tentaculele într-un singur rând, etichetate cu numere.

{{index journal, "weresquirrel example"}}

Vom reprezenta jurnalul pe care îl menține Jacques sub forma unui array de obiecte.

```{test: wrap}
let journal = [
  {events: ["work", "touched tree", "pizza",
            "running", "television"],
   squirrel: false},
  {events: ["work", "ice cream", "cauliflower",
            "lasagna", "touched tree", "brushed teeth"],
   squirrel: false},
  {events: ["weekend", "cycling", "break", "peanuts",
            "beer"],
   squirrel: true},
  /* and so on... */
];
```

## Mutabilitatea

În curând vom trece la programarea propriu-zisă. Mai întâi vom discuta despre încă un aspect teoretic important de înțeles.

{{index mutability, "side effect", number, string, Boolean, [object, mutability]}}

Am văzut că valorile obiectelor pot fi modificate. Tipurile de valori despre care am discutat în capitolele anterioare (numere, stringuri și valori booleene) sunt toate _imutabile_ - este imposibil să schimbăm valorile acelor tipuri. Le putem combina pentru a deriva noi valori dar, atunci când considerăm o anumită valoare de tip string, ea rămâne nemodificată. Textul din interiorul ei nu poate fi modificat. Dacă avem un string care conține `"cat"`, nu este posibil ca să scriem cod care să schimbe un singur caracter în codul nostru pentru a obține stringul `"rat"`.

Obiectele au un comportament diferit. _Puteți_ să le modificați proprietățile astfel încât o valoare de tip obiect să aibă conținuturi diferite la momente diferite.

{{index [object, identity], identity, [memory, organization], mutability}}

Când avem două numere, 120 și 120, le putem considera perfect identice, fie că se referă sau nu la aceeiași biți. În cazul obiectelor, există o diferență între a avea două referințe către același obiect sau două obiecte diferite dar care au  aceleași proprietăți. Să considerăm codul următor:

```
let object1 = {value: 10};
let object2 = object1;
let object3 = {value: 10};

console.log(object1 == object2);
// → true
console.log(object1 == object3);
// → false

object1.value = 15;
console.log(object2.value);
// → 15
console.log(object3.value);
// → 10
```

{{index "tentacle (analogy)", [binding, "model of"]}}

Bindingurile `object1` și `object2` se referă la _același_ obiect și, din acest motiv, o schimbare asupra valorii referite de `object1` va modifica și valoarea referită de `object2`. Bindingul `object3` se referă la un obiect diferit care conține aceleași proprietăți ca și `object1` dar are un ciclu de viață separat.

{{index "const keyword", "let keyword", [binding, "as state"]}}

Bindingurile pot fi modificabile sau constante, dar acesta este un alt aspect, separat de modul în care valorile lor se comportă. Deși valorile de tip număr nu se schimbă, puteți declara un binding cu `let` care va urmări progresul unui număr care se schimbă prin modificarea valorii la care se referă bindingul. Similar, chiar dacă un binding `const` către un obiect nu poate fi modificat și va continua să se refere la același obiect, _conținutul_ acelui obiect poate fi modificat. 

```{test: no}
const score = {visitors: 0, home: 0};
// This is okay
score.visitors = 1;
// This isn't allowed
score = {visitors: 1, home: 1};
```

{{index "== operator", [comparison, "of objects"], "deep comparison"}}

Când comparați obiecte cu ajutorul operatorului JavaScript `==`, acesta va compara prin identitate: va produce `true` doar dacă cele două obiecte au exact aceeași valoare. Compararea unor obiecte diferite va returna `false`, chiar dacă ele au proprietăți identice. Nu există operație de comparare profundă ("deep comparison") predefinită în JavaScript, care să compare două obiecte prin conținutul lor, dar o puteți defini (ceea ce de fapt este unul dintre [exercițiile](data#exercise_deep_compare) de la sfârșitul acestui capitol).

## Jurnalul licantropului

{{index "weresquirrel example", lycanthropy, "addEntry function"}}

Jacques își pornește interpretorul de JavaScript și își setează mediul de care are nevoie pentru a își menține jurnalul.

```{includeCode: true}
let journal = [];

function addEntry(events, squirrel) {
  journal.push({events, squirrel});
}
```

{{index [braces, object], "{} (object)", [property, definition]}}

Observați că obiectul pe care il adaugă în jurnal arată puțin ciudat. În loc să adauge proprietăți declarate cam așa: `events: events`, doar specifică numele proprietății. Aceasta este o scurtătură ce înseamnă același lucru - dacă numele unei proprietăți în notația cu acolade nu este urmat de o valoare, această valoare este preluată din bindingul cu același nume.

În fiecare seara la ora 10PM sau dimineața zilei următoare, Jacques își înregistrează ziua:

```
addEntry(["work", "touched tree", "pizza", "running",
          "television"], false);
addEntry(["work", "ice cream", "cauliflower", "lasagna",
          "touched tree", "brushed teeth"], false);
addEntry(["weekend", "cycling", "break", "peanuts",
          "beer"], true);
```

Imediat ce va avea suficiente date va face apel la statistică pentru a afla care dintre aceste evenimente ar putea fi legat de transformarea sa în veveriță.

{{index correlation}}

_Factorul de corelare_ este o măsura a dependenței între variable statistice. O variabilă statistică nu este același lucru ca și o variabilă în programare. În statistică, de regulă, avem un set de _măsurători_ și fiecare variabilă este măsurată pentru fiecare măsurătoare. Corelarea între variabile este de regulă exprimată ca o valoare între -1 și 1. Corelare 0 înseamnă că cele două variabile nu sunt relaționate. O corelare 1 arată că cele două variabile sunt perfect relaționate - cunoscând valoarea uneia dintre ele putem afla valoarea celeilalte. -1 înseamnă de asemenea că cele două variabile sunt perfect relaționate, dar sunt opuse - de exemplu, când una este adevărată, cealaltă este falsă.

{{index "phi coefficient"}}

Pentru a măsura factorul de corelare între două variabile booleene, putem utiliza _ccoeficientul phi_ (_ϕ_). Formula pentru calculul acestuia se bazează pe un tabel de frecvență care conține de câte ori a fost observată fiecare combinație a variabilelor. Rezultatul formulei este un număr între -1 și 1 care descrie corelarea.

De exemplu, să considerăm evenimentul "mănânc o pizza" și să construim un tabel de frecvență ca și în figura de mai jos, unde fiecare număr arată de câte ori a apărut combinația respectivă în timpul măsurătorilor:

{{figure {url: "img/pizza-squirrel.svg", alt: "Mănânc o pizza vs. mă transform în veveriță", width: "7cm"}}}

Dacă denumim tabelul _n_, putem calcula _ϕ_ cu formula:

{{if html

<div>
<table style="border-collapse: collapse; margin-left: 1em;"><tr>
  <td style="vertical-align: middle"><em>ϕ</em> =</td>
  <td style="padding-left: .5em">
    <div style="border-bottom: 1px solid black; padding: 0 7px;"><em>n</em><sub>11</sub><em>n</em><sub>00</sub> −
      <em>n</em><sub>10</sub><em>n</em><sub>01</sub></div>
    <div style="padding: 0 7px;">√<span style="border-top: 1px solid black; position: relative; top: 2px;">
      <span style="position: relative; top: -4px"><em>n</em><sub>1•</sub><em>n</em><sub>0•</sub><em>n</em><sub>•1</sub><em>n</em><sub>•0</sub></span>
    </span></div>
  </td>
</tr></table>
</div>

if}}

{{if tex

[\begin{equation}\varphi = \frac{n_{11}n_{00}-n_{10}n_{01}}{\sqrt{n_{1\bullet}n_{0\bullet}n_{\bullet1}n_{\bullet0}}}\end{equation}]{latex}

if}}

Notația [_n_~01~]{if html}[[$n_{01}$]{latex}]{if tex} se referă la numărul de măsurători în care prima variabilă este falsă (0) iar cea de a doua adevărată (1). În tabelul nostru, [_n_~01~]{if html}[[$n_{01}$]{latex}]{if tex} is această valoare este 9.

Valoarea [_n_~1•~]{if html}[[$n_{1\bullet}$]{latex}]{if tex} reprezintă suma tuturor măsurătorilor în care prima variabilă este `true`, care este 5 în exemplul nostru (suma pe a doua linie a tabelului). În același fel, [_n_~•0~]{if html}[[$n_{\bullet0}$]{latex}]{if tex} se referă la suma tuturor măsurătorilor  în care cea de a doua variabilă este `false` (suma pe prima coloană).

{{index correlation, "phi coefficient"}}

Pentru exemplul nostru, numărătorul expresiei se evaluează 1×76−4×9 = 40, iar numitorul va fi rădăcina pătrată a numărului 5×85×10×80, sau [√340000]{if html}[[$\sqrt{340000}$]{latex}]{if tex}. În final obținem valoarea _ϕ_ ≈ 0.069, care este o valoare mică. Consumul de pizza nu pare să aibă o influență asupra transformărilor.

## Calcularea corelației

{{index [array, "as table"], [nesting, "of arrays"]}}

În JavaScript putem reprezenta un tabel 2 x 2 ca un array de 4 elemente (`[76, 9, 4, 1]`). Putem să utilizăm și alte reprezentări, cum ar fi un array ce conține două elemente de tip array cu câte două elemente fiecare: (`[[76, 9], [4, 1]]`) sau un obiect care conține proprietăți cu nume precum `"11"` sau
`"01"`, dar un array de patru elemente este simplu și permite o reprezentare elegantă a formulei de calcul. Vom interpreta indicii array-ului ca și numere în baza 2 reprezentate pe doi biți, prima cifră (cea mai semnificativă) reprezentând variabila `squirrel`, iar cea de a doua (cea mai puțin semnificativă) variabila pentru eveniment. De exemplu, numărul binar `10` se referă la cazul în care Jacques s-a transformat, dar evenimentul nu a avut loc. Această situație a fost observată de 4 ori. Deoarece `10` reprezintă 2 în notația decimală, vom memora această valoarea în elementul de index 2 al array-ului.

{{index "phi coefficient", "phi function"}}

{{id phi_function}}

Aceasta este funcția care calculează coeficientul _ϕ_ din array:

```{includeCode: strip_log, test: clip}
function phi(table) {
  return (table[3] * table[0] - table[2] * table[1]) /
    Math.sqrt((table[2] + table[3]) *
              (table[0] + table[1]) *
              (table[1] + table[3]) *
              (table[0] + table[2]));
}

console.log(phi([76, 9, 4, 1]));
// → 0.068599434
```

{{index "square root", "Math.sqrt function"}}

Aceasta este o translatare directă a formulei pentru _ϕ_ în JavaScript. `Math.sqrt` este funcția pentru calculul rădăcinii pătrate, parte a obiectului `Math` într-un mediu JavaScript standard. Pentru a obține valori cum ar fi [n~1•~]{if html}[[$n_{1\bullet}$]{latex}]{if tex} va trebui să adunăm câte două elemente ale tabloului deoarece suma rândurilor sau coloanelor nu este memorată în interiorul structurii noastre.

{{index "JOURNAL data set"}}

Jacques și-a păstrat jurnalul timp de trei luni. Rezultatele sunt disponibile la adresa
[_https://eloquentjavascript.net/code#4_](https://eloquentjavascript.net/code#4).

{{index "tableFor function"}}

Pentru a construi un tabel de frecvență pentru un anumit eveniment din jurnal, trebuie să parcurgem toate intrările din jurnal și să calculăm de câte ori a apărut evenimentul, împreună cu transformările:

```{includeCode: strip_log}
function tableFor(event, journal) {
  let table = [0, 0, 0, 0];
  for (let i = 0; i < journal.length; i++) {
    let entry = journal[i], index = 0;
    if (entry.events.includes(event)) index += 1;
    if (entry.squirrel) index += 2;
    table[index] += 1;
  }
  return table;
}

console.log(tableFor("pizza", JOURNAL));
// → [76, 9, 4, 1]
```

{{index [array, searching], "includes method"}}

Array-urile au o metodă `includes` care verifică dacă valoarea data ca și argument există în array. Funcția  noastră o utilizează pentru a determina dacă numele evenimentului face parte din lista de evenimente pentru o zi dată.

{{index [array, indexing]}}

Corpul buclei din `tableFor` determină în care dintre cele patru contoare trebuie să plasăm fiecare valoare din jurnal, verificând dacă intrarea conține evenimentul respectiv și dacă transformarea a avut loc sau nu.

Acum avem la dispoziție instrumentele pentru a calcula factorii de corelare pentru fiecare eveniment. Ne mai rămâne doar să facem calculele și să vedem dacă obținem un rezultat relevant.

{{id for_of_loop}}

## Parcurgerea array-urilor

{{index "for loop", loop, [array, iteration]}}

În funcția `tableFor` există o buclă:

```
for (let i = 0; i < JOURNAL.length; i++) {
  let entry = JOURNAL[i];
  // Do something with entry
}
```

Acest tip de buclă este foarte obișnuit în JavaScript - parcurgerea unui array element cu element este o operație frecventă iar pentru a o implementa vom utiliza un contor care parcurge toată mulțimea de indecși și selectează pe rând fiecare element.

În JavaScript modern avem o modalitate mai simplă de a scrie asemenea bucle:

```
for (let entry of JOURNAL) {
  console.log(`${entry.events.length} events.`);
}
```

{{index "for/of loop"}}

Când o buclă `for` arată astfel, cu cuvântul `of` după definiția variabilei, va itera peste toate elementele valorii plasate după `of`. Această structură funcționează nu doar pentru array-uri, dar și pentru stringuri și alte structuri de date. Vom detalia cum funcționează în [capitolul ?](object).

{{id analysis}}

## Analiza finală

{{index journal, "weresquirrel example", "journalEvents function"}}

Trebuie să calculăm o corelare pentru fiecare tip de eveniment care apare în setul de date. Pentru aceasta, mai întâi trebuie să determinăm lista evenimentelor care apar în jurnal:

{{index "includes method", "push method"}}

```{includeCode: "strip_log"}
function journalEvents(journal) {
  let events = [];
  for (let entry of journal) {
    for (let event of entry.events) {
      if (!events.includes(event)) {
        events.push(event);
      }
    }
  }
  return events;
}

console.log(journalEvents(JOURNAL));
// → ["carrot", "exercise", "weekend", "bread", …]
```

Parcurgând lista tuturor intrărilor din jurnal și colectând toate evenimentele care nu apar în array-ul `events`, funcția va colecta toate tipurile de evenimente din jurnal.

Apoi vom putea calcula factorul de corelare pentru fiecare eveniment.

```{test: no}
for (let event of journalEvents(JOURNAL)) {
  console.log(event + ":", phi(tableFor(event, JOURNAL)));
}
// → carrot:   0.0140970969
// → exercise: 0.0685994341
// → weekend:  0.1371988681
// → bread:   -0.0757554019
// → pudding: -0.0648203724
// and so on...
```

Majoritatea factorilor de corelare par să fie aproape de 0. Transformarea pare însă să aibă o oarecare legătură cu weekendurile. Haideți să filtrăm rezultatele pentru a afișa doar factorii mai mari decât 0.1 sau mai mici decât -0.1.

```{test: no, startCode: true}
for (let event of journalEvents(JOURNAL)) {
  let correlation = phi(tableFor(event, JOURNAL));
  if (correlation > 0.1 || correlation < -0.1) {
    console.log(event + ":", correlation);
  }
}
// → weekend:        0.1371988681
// → brushed teeth: -0.3805211953
// → candy:          0.1296407447
// → work:          -0.1371988681
// → spaghetti:      0.2425356250
// → reading:        0.1106828054
// → peanuts:        0.5902679812
```

Aha! Există doi factori care au o corelație cu mult mai puternică decât toți ceilalți. Consumul de arahide are un efect puternic pozitiv asupra șansei de transformare în veveriță iar spălarea dinților are un efect negativ semnificativ.

Interesant. Haideți să mai încercăm un experiment.

```
for (let entry of JOURNAL) {
  if (entry.events.includes("peanuts") &&
     !entry.events.includes("brushed teeth")) {
    entry.events.push("peanut teeth");
  }
}
console.log(phi(tableFor("peanut teeth", JOURNAL)));
// → 1
```

Acesta este un rezultat puternic. Fenomenul apare cu siguranță atunci când Jacques mănâncă arahide și nu se spală pe dinți. Dacă nu ar fi fost atât de atent la igiena dentară, nu ar fi determinat niciodată cauza transformărilor sale.

## Mai multe despre array-uri

{{index [array, methods], [method, array]}}

Haideți să ne familiarizăm cu alte câteva concepte relativ la obiecte. Să începem cu alte câteva metode generale utile pentru manipularea array-urilor

{{index "push method", "pop method", "shift method", "unshift method"}}

Ne-am întâlnit cu `push` and `pop`, care adaugă sau elimină elemente la sfârșitul unui array, [anterior](data#array_methods) în acest capitol. Metodele similare pentru adăugarea și eliminarea elementelor la începutul unui array se numesc `unshift` și `shift`.

```
let todoList = [];
function remember(task) {
  todoList.push(task);
}
function getTask() {
  return todoList.shift();
}
function rememberUrgently(task) {
  todoList.unshift(task);
}
```

{{index "task management example"}}

Programul de mai sus gestionează o coadă de sarcini. Adăugați sarcini la sfârșitul cozii apelând `remember("groceries")`, iar atunci când sunteți pregătiți să începeți o nouă activitate, veți apela `getTask()` pentru a obține și a elimina primul element din coadă. Funcția `rememberUrgently` adaugă un element la începutul cozii.

{{index [array, searching], "indexOf method", "lastIndexOf method"}}

Pentru a căuta o anumită valoare, avem la dispoziție pentru array-uri metoda `indexOf`. Metoda caută în întregul array, de la stânga la dreapta, și returnează indexul la care a fost găsită valoarea căutată sau -1 dacă nu a găsit valoarea. Pentru a căuta de la dreapta la stânga, puteți folosi metoda `lastIndexOf`.

```
console.log([1, 2, 3, 2, 1].indexOf(2));
// → 1
console.log([1, 2, 3, 2, 1].lastIndexOf(2));
// → 3
```

Ambele metode acceptă un al doilea parametru care permite setarea indicelui de la care începe căutarea.

{{index "slice method", [array, indexing]}}

O altă metodă fundamentală pentru array-uri este `slice` care primește indicii de început și de sfârșit și returnează un array ce conține elementele dintre cei doi indecși. Indicele de început este inclus iar cel de sfârșit este exclus.

```
console.log([0, 1, 2, 3, 4].slice(2, 4));
// → [2, 3]
console.log([0, 1, 2, 3, 4].slice(2));
// → [2, 3, 4]
```

{{index [string, indexing]}}

Atunci când indexul de sfârșit nu este precizat, `slice` va lua toate elementele de la indexul de start până la sfârșitul array-ului. Dacă se omite și indicele de început, `slice` va copia toate elementele array-ului.

{{index concatenation, "concat method"}}

Metoda `concat` se poate folosi pentru a alipi array-urile și a crea un nou array, similar cu efectul operatorului `+` pentru stringuri.

Următorul exemplu vă prezintă `concat` și `slice` în acțiune. Primind ca argumente un array și un index, funcția returnează un nou array care este o copie a array-ului original având elementul de la indexul dat eliminat din array:

```
function remove(array, index) {
  return array.slice(0, index)
    .concat(array.slice(index + 1));
}
console.log(remove(["a", "b", "c", "d", "e"], 2));
// → ["a", "b", "d", "e"]
```

Dacă transmiteți metodei `concat` un element care nu este array, acea valoare va fi adăugată la noul array ca și cum ar fi un array format dintr-un singur element.

## Stringuri și proprietățile lor

{{index [string, properties]}}

Putem citi proprietăți cum ar fi `length` sau `toUpperCase` de pe valorile de tip string. Însă dacă încercăm să adăugăm o nouă proprietate, aceasta nu va fi memorată.

```
let kim = "Kim";
kim.age = 88;
console.log(kim.age);
// → undefined
```

Valorile de tip string, number și boolean nu sunt obiecte și, cu toate că limbajul nu afișează erori dacă încercați să setați noi proprietăți pentru ele, acele proprietăți nu vor fi memorate. Așa cum am precizat anterior, aceste valori sunt imutabile și nu pot fi modificate.

{{index [string, methods], "slice method", "indexOf method", [string, searching]}}

Dar aceste tipuri au proprietăți implicite. Orice valoare de tip string are câteva metode disponibile. Unele proprietăți foarte utile sunt `slice` și `indexOf`, care seamănă cu metodele cu același nume pentru array-uri.

```
console.log("coconuts".slice(4, 7));
// → nut
console.log("coconut".indexOf("u"));
// → 5
```

O diferență este aceea că metoda `indexOf` a unui string poate căuta un string ce conține mai multe caractere în timp ce metoda cu același nume pentru array-uri caută doar un singur element.

```
console.log("one two three".indexOf("ee"));
// → 11
```

{{index [whitespace, trimming], "trim method"}}

Metoda `trim` elimină spațiile goale (spaces, newlines, tabs, și alte caractere asemănătoare) de la începutul și de la sfârșitul unui string.

```
console.log("  okay \n ".trim());
// → okay
```

Funcția `zeroPad` din [capitolul anterior](functions) există de asemenea ca și metodă. Se numește `padStart` și primește două argumente, lungimea dorită și caracterul cu care se completează stringul.

```
console.log(String(6).padStart(3, "0"));
// → 006
```

{{id split}}

Puteți împărți un string în funcție de fiecare apariție a unui alt string cu metoda `split` (și veți obține un array de stringuri). Apoi puteți recombina părțile folosind metoda `join`.

```
let sentence = "Secretarybirds specialize in stomping";
let words = sentence.split(" ");
console.log(words);
// → ["Secretarybirds", "specialize", "in", "stomping"]
console.log(words.join(". "));
// → Secretarybirds. specialize. in. stomping
```

{{index "repeat method"}}

Un string poate fi repetat folosind metoda `repeat`, care crează un nou string ce conține mai multe copii ale stringului original, concatenate.

```
console.log("LA".repeat(3));
// → LALALA
```

{{index ["length property", "for string"], [string, indexing]}}

Am folosit deja proprietatea `length` pentru stringuri. Accesarea caracterelor individuale ale unui string seamănă cu accesarea elementelor unui array (cu o precizare despre care vom discuta în [capitolul ?](higher_order#code_units)).

```
let string = "abc";
console.log(string.length);
// → 3
console.log(string[1]);
// → b
```

{{id rest_parameters}}

## Parametri "rest"

{{index "Math.max function"}}

Uneori este util ca o funcție să accepte oricâte argumente. De exemplu, `Math.max` calculează maximul dintre _toate_ argumentele transmise.

{{index "period character", "max example", spread}}

Pentru a scrie o asemenea funcție, punem `...` în fața ultimului parametru, astfel:

```{includeCode: strip_log}
function max(...numbers) {
  let result = -Infinity;
  for (let number of numbers) {
    if (number > result) result = number;
  }
  return result;
}
console.log(max(4, 1, 9, -2));
// → 9
```

Când apelați o asemenea funcție, _parametrul rest_ este legat de un array ce conține toate argumentele suplimentare. Dacă există și alți parametri înaintea lui, valorile lor nu fac parte din acest array. Atunci când, ca și în `max`, este singurul parametru al funcției, va memora toate argumentele.

{{index [function, application]}}

Puteți utiliza o notație similară pentru a _apela_ o funcție cu un array de argumente.

```
let numbers = [5, 1, 7];
console.log(max(...numbers));
// → 7
```

Aceasta _distribuie (spread)_ array-ul în apelul funcției, transmițând elementele array-ului ca și argumente separate. Putem include astfel un array împreună cu alte argumente, astfel: `max(9, ...numbers, 2)`.

{{index [array, "of rest arguments"], "square brackets"}}

Notația cu paranteze pătrate a array-urilor acceptă la rândul ei operatorul de distribuire pentru a include elementele unui array într-un alt array.

```
let words = ["never", "fully"];
console.log(["will", ...words, "understand"]);
// → ["will", "never", "fully", "understand"]
```

## Obiectul Math

{{index "Math object", "Math.min function", "Math.max function", "Math.sqrt function", minimum, maximum, "square root"}}

Așa cum am văzut, `Math` este o cutie cu funcții utilitare pentru numere, cum ar fi `Math.max` (maxim), `Math.min` (minim), și `Math.sqrt` (rădăcina pătrată) și multe altele.

{{index namespace, [object, property]}}

{{id namespace_pollution}}

Obiectul `Math` este un container utilizat pentru a grupa funcționalități similare. Există un singur obiect `Math` și acesta nu este util aproape niciodată ca și valoare. Mai degrabă el reprezintă un _spațiu de nume_ astfel încât funcțiile și valorile nu sunt bindinguri globale.

{{index [binding, naming]}}

Prea multe bindiguri globale "poluează" spațiul de nume. Cu cât au fost deja folosite mai multe nume, cu atât mai probabil este ca să suprascriem accidental valoarea unui binding existent. De exemplu, este probabil ca să intenționăm să scriem o funcție cu numele `max` în unul din programele noastre. Deoarece funcția predefinită `max` din JavaScript este ascunsă în interiorul obiectului `Math`, nu trebuie să ne îngrijoreze suprascrierea accidentală a acesteia.

{{index "let keyword", "const keyword"}}

Multe limbaje de programare vă vor opri, sau cel puțin vă vor avertiza, atunci când veți încerca să definiți un binding cu un nume care deja a fost utilizat. JavaScript procedează la fel atunci când este vorba despre bindinguri definite cu `let` sau `const`, dar nu și pentru cele definite implicit sau declarate cu `var` sau `function`.

{{index "Math.cos function", "Math.sin function", "Math.tan function", "Math.acos function", "Math.asin function", "Math.atan function", "Math.PI constant", cosine, sine, tangent, "PI constant", pi}}

Înapoi la obiectul `Math`. Dacă aveți nevoie să efectuați calcule trigonometrice, `Math` vă poate ajuta. El conține `cos` (cosinus), `sin` (sinus) și `tan` (tangenta), precum și funcțiile lor inverse, `acos`, `asin` și `atan`. Numărul π (pi) - de fapt cea mai bună aproximare care poate fi reprezentată într-un număr în JavaScript - este definit ca  `Math.PI`. Este o veche tradiție în programare ca numele constantelor să fie scrise cu litere mari.

```{test: no}
function randomPointOnCircle(radius) {
  let angle = Math.random() * 2 * Math.PI;
  return {x: radius * Math.cos(angle),
          y: radius * Math.sin(angle)};
}
console.log(randomPointOnCircle(2));
// → {x: 0.3667, y: 1.966}
```

Dacă funcțiile _sinus_ și _cosinus_ nu vă sunt familiare, nu vă îngrijorați. Atunci când le vom utiliza, în [capitolul ?](dom#sin_cos), le vom explica.

{{index "Math.random function", "random number"}}

Exemplul anterior utilizează funcția `Math.random`. Aceasta este o funcție care generează un număr pseudoaleator între 0 (inclusiv) și 1(exclusiv) la fiecare apel.

```{test: no}
console.log(Math.random());
// → 0.36993729369714856
console.log(Math.random());
// → 0.727367032552138
console.log(Math.random());
// → 0.40180766698904335
```

{{index "pseudorandom number", "random number"}}

Deși computerele sunt mașini deterministe (reacționează întotdeauna la fel pentru un set dat de valori de intrare), este posibil să le utilizăm pentru a produce valori ce par aleatoare. Pentru aceasta, mașina memorează o valoare ascunsă și de fiecare dată când avem nevoie de un nou număr face niște calcule complicate asupra acestei valori ascunse pentru a crea o nouă valoare. Apoi memorează noua valoare și deduce un nou număr pe care îl returnează. În acest mod, poate genera numere greu de prezis într-un mod care _pare_ aleator.

{{index rounding, "Math.floor function"}}

Dacă avem nevoie de un număr întreg în loc de un număr real, putem utiliza `Math.floor` (care trunchiază rezultatul la cel mai apropiat număr întreg) asupra rezultatului returnat de `Math.random`.

```{test: no}
console.log(Math.floor(Math.random() * 10));
// → 2
```

Înmulțirea rezultatului cu 10 ne returnează un număr mai mare sau egal decât 0 și mai mic strict decât 10. Deoarece `Math.floor` trunchiază, această expresie va produce cu șanse egale, orice număr de la 0 la 9.

{{index "Math.ceil function", "Math.round function", "Math.abs function", "absolute value"}}

Aveți la dispoziție și funcțiile `Math.ceil` (care rotunjește în sus la cel mai apropiat număr întreg), `Math.round` (pentru rotunjirea la cel mai apropiat număr întreg), și `Math.abs`, care returnează valoarea absolută a unui număr (numerele pozitive sunt returnate așa cum au fost primite iar cele negative se neagă înainte de a fi returnate).

## Destructurarea

{{index "phi function"}}

Să revenim la funcția `phi` pentru un moment.

```{test: wrap}
function phi(table) {
  return (table[3] * table[0] - table[2] * table[1]) /
    Math.sqrt((table[2] + table[3]) *
              (table[0] + table[1]) *
              (table[1] + table[3]) *
              (table[0] + table[2]));
}
```

{{index "destructuring binding", parameter}}

Unul dintre motivele pentru care această funcție este dificil de citit este că avem un binding care se referă la array-ul nostru, dar am prefera să avem bindinguri către elementele array-ului nostru, cum ar fi `let n00 = table[0]` și așa mai departe. Din fericire, avem o modalitate simplă de a face acest lucru în JavaScript.

```
function phi([n00, n01, n10, n11]) {
  return (n11 * n00 - n10 * n01) /
    Math.sqrt((n10 + n11) * (n00 + n01) *
              (n01 + n11) * (n00 + n10));
}
```

{{index "let keyword", "var keyword", "const keyword", [binding, destructuring]}}

Această metodă funcționează și pentru bindinguri create cu `let`, `var`, sau `const`. Dacă știți că valoarea pe care o veți primi este un array, puteți utiliza paranteze pătrate pentru a privi în interiorul valori și a crea bindinguri pentru conținutul său.

{{index [object, property], [braces, object]}}

O sintaxă asemănătoare funcționează pentru obiecte, utilizând acolade în loc de paranteze pătrate.

```
let {name} = {name: "Faraji", age: 23};
console.log(name);
// → Faraji
```

{{index null, undefined}}

De menționat că, dacă încercați să destructurați `null` sau `undefined`, veți primi o eroare, la fel cum veți primi o eroare și dacă încercați să accesați o proprietate a acelor valori.

## JSON

{{index [array, representation], [object, representation], "data format", [memory, organization]}}

Deoarece proprietățile doar referă valorile în loc să le conțină, obiectele și array-urile sunt stocate în memoria computerului ca și secvențe de biți ce memorează _adresele_ - locurile din memorie - unde este localizat conținutul lor. Astfel, un array ce conține un alt array constă din cel puțin o zonă de memorie pentru array-ul din interior și alta pentru array-ul exterior, ce conține (printre alte lucruri) un număr binar reprezentând poziția array-ului interior.

Dacă vreți să salvați date într-un fișier sau să le trimiteți unui alt computer din rețea, trebuie să convertiți cumva aceste mixuri de adrese de memorie într-o descriere care poate fi salvată sau transmisă. Ați _putea_ transmite toată memoria computerului împreuna cu adresa valorii care vă interesează, dar aceasta nu pare să fie o abordare prea bună.

{{indexsee "JavaScript Object Notation", JSON}}
{{index serialization, "World Wide Web"}}

Ce putem face este să _serializăm_ datele. Adică să le convertim într-o descriere. Un format popular de serializare este numit JSON (pronunțat ca și "Jason"), prescurtarea pentru JavaScript Object Notation. Este un format foarte popular pentru stocarea datelor și ca și format de comunicare pe web, chiar și în alte limbaje de programare.

{{index [array, notation], [object, creation], [quoting, "in JSON"], comment}}

JSON arată similar modului de a scrie array-uri și obiecte în JavaScript, cu câteva restricții. Toate numele proprietăților trebuie să fie înconjurate de ghilimele și numai expresiile simple de date sunt permise - nu apeluri de funcții, legături sau orice altceva care implică de fapt calcule. Comentariile nu sunt permise în JSON.

O intrare de jurnal ar putea fi scrisă astfel în JSON:

```{lang: "application/json"}
{
  "squirrel": false,
  "events": ["work", "touched tree", "pizza", "running"]
}
```

{{index "JSON.stringify function", "JSON.parse function", serialization, deserialization, parsing}}

JavaScript ne pune la dispoziție funcțiile `JSON.stringify` și `JSON.parse` pentru conversia datelor în și din acest format. Prima funcție primește o valoare JavaScript si returnează un string în format JSON. Ceea de a doua primește ca argument un string JSON valid și îl convertește în valoarea inițială.

```
let string = JSON.stringify({squirrel: false,
                             events: ["weekend"]});
console.log(string);
// → {"squirrel":false,"events":["weekend"]}
console.log(JSON.parse(string).events);
// → ["weekend"]
```

## Rezumat

Obiectele și array-urile (care sunt un anumit tip de obiect) oferă modalități pentru a grupa mai multe valori într-o singură valoare. Conceptual, acest lucru ne permite să punem o grămadă de lucruri asemănătoare într-o pungă și să o folosim ca atare, în loc să ne înconjurăm brațele în jurul tuturor lucrurilor individuale și să încercăm să le ținem separat.

Cele mai multe valori din JavaScript au proprietăți, excepțiile fiind `null` și `undefined`. Proprietățile sunt accesate folosind `value.prop` sau `value["prop"]`. Obiectele tind să utilizeze nume pentru proprietățile lor și să aibă un set relativ fix de proprietăți. Array-urile, pe de altă parte, conțin în mod obișnuit un număr variabil de valori conceptual identice și utilizează numere (începând cu 0) ca și nume pentru proprietățile lor.

Sunt disponibile și câteva proprietăți denumite pentru array-uri, cum ar fi `length`, precum și câteva metode. Metodele sunt funcții care sunt memorate în proprietăți și de regulă acționează asupra valorii a căror proprietate sunt.

Puteți itera peste un array folosind o buclă `for` specială - `for (let element of array)`.

## Exerciții

### Suma unui interval

{{index "summing (exercise)"}}

[Introducerea](intro) acestei cărți visa la următorul mod elegant de a calcula suma unui interval de numere:

```{test: no}
console.log(sum(range(1, 10)));
```

{{index "range function", "sum function"}}

Scrieți o funcție `range` care primește două argumente, `start` și `end`, și returnează un array ce conține toate numerele de la `start` la `end` (inclusiv).

Apoi scrieți o funcție `sum` care va returna suma valorilor in array. Rulați exemplul de mai sus și verificați dacă se afișează valoarea 55.

{{index "optional argument"}}

Ca și sarcină suplimentară, modificați funcția `range` ca să primească un al treilea argument, opțional, care să precizeze pasul de incrementare. Dacă al treilea argument lipsește, pasul de incrementare să aibă valoarea 1 și comportamentul funcției să fie cel descris inițial. Apelul funcției în forma `range(1, 10, 2)` trebuie să returneze `[1, 3, 5, 7, 9]`. Asigurați-vă că rezultatul este corect și pentru valori negative ale pasului, astfel încât `range(5, 2, -1)` va produce `[5, 4, 3, 2]`.

{{if interactive

```{test: no}
// Your code here.

console.log(range(1, 10));
// → [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log(range(5, 2, -1));
// → [5, 4, 3, 2]
console.log(sum(range(1, 10)));
// → 55
```

if}}

{{hint

{{index "summing (exercise)", [array, creation], "square brackets"}}

Construirea unui array se realizează cel mai simplu prin inițializarea unui binding cu valoarea  `[]` (un array gol) și apelarea repetată a metodei `push` pentru a adăuga valori. Nu uitați să returnați array-ul la sfârșitul funcției.

{{index [array, indexing], comparison}}

Deoarece limita de final este inclusivă, trebuie să utilizați operatorul `<=` și nu operatorul `<` pentru a verifica finalul buclei.

{{index "arguments object"}}

Parametrul pentru pasul de incrementare/decrementare poate fi un parametru opțional care are valoarea implicită 1 (folosiți operatorul `=`).

{{index "range function", "for loop"}}

Pentru ca funcția `range` să poată lucra și cu valori pozitive și cu valori negative ale pasului, probabil cel mai ușor ar fi să scrieți două bucle diferite - una pentru incrementare și una pentru decrementare, deoarece în funcție de semnul pasului, va trebui să folosiți `>=` sau `<=` pentru a verifica terminarea buclei.

De asemenea, atenție la valoarea implicită a pasului. Aceasta ar trebui să fie -1 atunci când valoarea de final este mai mică decât valoarea inițială. Astfel, `range(5, 2)` va returna un array, în loc să se blocheze într-o buclă infinită. Pentru setarea valorii implicite a unui parametru ne putem referi la parametrii anteriori.

hint}}

### Inversarea unui array

{{index "reversing (exercise)", "reverse method", [array, methods]}}

Array-urile au o metodă `reverse` care modifică un array prin inversarea ordinii elementelor sale. Pentru acest exercițiu, scrieți două funcții: `reverseArray` și `reverseArrayInPlace`. 
Prima va primi ca argument un array și va returna un _nou_ array conținând aceleași elemente, dar în ordine inversă. Cea de a doua funcție va face ceea ce face `reverse`: va _modifica_ array-ul primit ca argument prin inversarea ordinii elementelor sale. Nu aveți voie să folosiți funcția `reverse` (cu scopul de a fi creativi).

{{index efficiency, "pure function", "side effect"}}

Revenind la notițele despre efecte secundare și funcții pure din [capitolul anterior](functions#pure), care dintre cele două variante credeți că este utilă în mai multe situații? Care credeți că se execută mai rapid?

{{if interactive

```{test: no}
// Your code here.

console.log(reverseArray(["A", "B", "C"]));
// → ["C", "B", "A"];
let arrayValue = [1, 2, 3, 4, 5];
reverseArrayInPlace(arrayValue);
console.log(arrayValue);
// → [5, 4, 3, 2, 1]
```

if}}

{{hint

{{index "reversing (exercise)"}}

Putem implementa `reverseArray` în două moduri. Primul este de a parcurge array-ul primit ca argument de la început la sfârșit și să utilizăm metoda `unshift` pentru a insera fiecare element la începutul noului array. A doua metodă ar presupune să iterăm array-ul original de la sfârșit la început și să utilizăm metoda `push`. Pentru a itera peste un array de la sfârșit la început va trebui să folosiți `for (let i = array.length - 1; i >= 0; i--)`.

{{index "slice method"}}

Inversarea unui array pe loc este mai dificilă. Trebuie să aveți grijă să nu suprascrieți elemente de care s-ar putea să aveți nevoie mai târziu. Utilizarea unor apeluri către `reverseArray` sau altă metodă de a copia întregul array (cum ar fi `array.slice(0)`) funcționează dar în acest mod trișați (ar trebui să nu folosim structuri de date suplimentare).

Ați putea aborda însă altfel problema: _interschimbăm_ primul și ultimul element, al doilea și penultimul, și așa mai departe. Veți itera pe jumătate din lungimea array-ului și veți interschimba elementul de la index `i` cu cel de pe poziția `array.length-i-1`. Interschimbarea o puteți realiza cu un binding local care va reține primul element din pereche, apoi al doilea element înlocuiește primul și apoi din bindingul temporar setați al doilea element la valoarea inițială a primului element.

hint}}

{{id list}}

### O listă

{{index ["data structure", list], "list (exercise)", "linked list", array, collection}}

Obiectele, ca și containere generice de valori, pot fi utilizate pentru a construi tot felul de structuri de date. O structură de date comună este o _listă_ ( a nu se confunda cu un array). O listă este un set de obiecte imbricate, primul obiect reținând o referință către al doilea, al doilea către al treilea și așa mai departe.

```{includeCode: true}
let list = {
  value: 1,
  rest: {
    value: 2,
    rest: {
      value: 3,
      rest: null
    }
  }
};
```

Obiectul rezultat va fi un lanț asemănător celui din figura de mai jos:

{{figure {url: "img/linked-list.svg", alt: "O listă liniară simplu înlănțuită",width: "8cm"}}}

{{index "structure sharing", [memory, structure sharing]}}

Interesant despre liste este faptul că pot partaja părți din structura lor. De exemplu, dacă aș crea două valori noi `{value: 0, rest: list}` și `{value: -1, rest: list}` (cu `list` referindu-se la bindingul definit anterior, ele sunt liste independente dar partajează structura care construiește ultimele trei elemente. Lista originală este de asemenea o listă validă de trei elemente.

Scrieți o funcție `arrayToList` care construiește o structură de tip listă asemănătoare cu cea prezentată, dacă primește argumentul `[1, 2, 3]`. De asemenea, scrieți funcția `listToArray` care realizează operația inversă. Apoi adăugați o funcție helper `prepend` care primește un element și o listă și adaugă elementul îm fața listei și o alta `nth` care primește o listă și un număr și returnează elementul de pe poziția dată a listei (primul element fiind pe poziția 0) sau `undefined` dacă nu există un asemenea element.

{{index recursion}}

Dacă nu ați implementat deja, încercați și o implementare recursivă a `nth`.

{{if interactive

```{test: no}
// Your code here.

console.log(arrayToList([10, 20]));
// → {value: 10, rest: {value: 20, rest: null}}
console.log(listToArray(arrayToList([10, 20, 30])));
// → [10, 20, 30]
console.log(prepend(10, prepend(20, null)));
// → {value: 10, rest: {value: 20, rest: null}}
console.log(nth(arrayToList([10, 20, 30]), 1));
// → 20
```

if}}

{{hint

{{index "list (exercise)", "linked list"}}

Construcția unei liste este mai ușoară de la sfârșit spre început. Astfel, `arrayToList` ar putea itera în ordine inversă pe array (vedeți exercițiul anterior) și, pentru fiecare element, să adauge un obiect la listă. Puteți utiliza un binding local pentru a memora o parte din listă care a fost construită deja și veți utiliza o atribuire în genul `list = {value: X, rest: list}` pentru a adăuga un element.

{{index "for loop"}}

Pentru a parcurge o listă (în `listToArray` și `nth`), puteți specifica o buclă `for` astfel:

```
for (let node = list; node; node = node.rest) {}
```

Înțegeți cum funcționează? Fiecare iterație a buclei folosește `node` pentru a se referi la sublista curentă iar în corpul buclei putem citi valoarea `value` pentru a obține elementul curent. La sfârșitul unui pas de iterare, `node` se va referi la sublista următoare. Când se ajunge la valoarea `null`, înseamnă că am parcurs toată lista și bucla se încheie.

{{index recursion}}

Versiunea recursivă a `nth` va proceda în mod similar, va observa porțiuni din ce în ce mai mici din listă și în același timp va număra în jos, până când indexul ajunge la valoarea 0, când va returna proprietatea `value` a nodului curent. Pentru a obține elementul zero al listei, ne referim la proprietatea `value` a nodului din capul listei. Pentru a obține elementul _N_+1 luăm elementul _N_ din lista `rest` memorată în proprietatea cu același nume a nodului din capul listei originale.

hint}}

{{id exercise_deep_compare}}

### Deep comparison

{{index "deep comparison (exercise)", [comparison, deep], "deep comparison", "== operator"}}

Operatorul `==` compară obiectele prin identitate. Dar uneori avem nevoie să comparăm valorile proprietăților lor.

Scrieți o funcție `deepEqual` care primește două valori și returnează `true` numai dacă ele au aceleași valori sau sunt obiecte cu aceleași proprietăți iar valorile proprietăților sunt egale atunci când se compară cu un apel recursiv al funcției `deepEqual`.

{{index null, "=== operator", "typeof operator"}}

Pentru a determina dacă valorile pot fi comparate direct (prin utilizarea operatorului `===`) sau valorile proprietăților lor trebuie să fie comparate, putem utiliza operatorul `typeof`. Dacă acesta produce valoarea `"object"` pentru ambele valori, atunci trebuie să facem o comparare pe adâncime. Dar trebuie să țineți cont de o excepție: `typeof null` produce tot `"object"`.

{{index "Object.keys function"}}

Funcția `Object.keys` va fi utilă pentru a parcurge proprietățile obiectelor în comparare.

{{if interactive

```{test: no}
// Your code here.

let obj = {here: {is: "an"}, object: 2};
console.log(deepEqual(obj, obj));
// → true
console.log(deepEqual(obj, {here: 1, object: 2}));
// → false
console.log(deepEqual(obj, {here: {is: "an"}, object: 2}));
// → true
```

if}}

{{hint

{{index "deep comparison (exercise)", [comparison, deep], "typeof operator", "=== operator"}}

Pentru a testa dacă aveți de a face cu un obiect real puteți testa o condiție `typeof x == "object" && x != null`. Atenție, comparați proprietățile doar dacă _ambele_ argumente sunt obiecte. În toate celelalte cazuri puteți returna imediat rezultatul aplicării operatorului `===`.

{{index "Object.keys function"}}

Utilizați `Object.keys` pentru a determina lista proprietăților. Trebuie să determinați dacă ambele obiecte au același număr de proprietăți (lungimea listelor de proprietăți este aceeași). Apoi, când parcurgeți una dintre cele două liste, verificați că fiecare dintre proprietățile din prima listă are un corespondent în cea de a doua.

{{index "return value"}}

Pentru a returna valoarea corectă din funcție, returnați imediat `false` când apare o nepotrivire și apoi returnați `true` la sfârșitul funcției.

hint}}
