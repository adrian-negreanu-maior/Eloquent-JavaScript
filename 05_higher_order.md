{{meta {load_files: ["code/scripts.js", "code/chapter/05_higher_order.js", "code/intro.js"], zip: "node/html"}}}

# Funcții de ordin superior

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Tzu-li și Tzu-ssu se lăudau despre mărimea ultimelor lor programe. 'Două sute de mii de linii de cod' a spus Tzu-li, 'fără comentarii!'. Tzu-ssu i-a răspuns 'Pssh, al meu are aproape un milion de linii deja'. Master Yuan-Ma a spus 'Cel mai bun program al meu are cinci sute de linii'. Auzind asta, Tzu-li și Tzu-ssu au fost iluminați.

quote}}

if}}

{{quote {author: "C.A.R. Hoare", title: "1980 ACM Turing Award Lecture", chapter: true}

{{index "Hoare, C.A.R."}}

Există două moduri de a concepe programe pentru computer: Unul este de a le scrie atât de simple încât să fie evident că nu au deficiențe. Iar celalt este de a le construi atât de complicate încât să nu existe deficiențe evidente.

quote}}

{{figure {url: "img/chapter_picture_5.jpg", alt: "Litere din diferite scrieri", chapter: true}}}

{{index "program size"}}

Un program de dimensiuni mari este un program costisitor și nu doar din cauza timpului necesar pentru a-l construi. Dimensiunea este aproape întotdeauna legată de complexitate și complexitatea provoacă multă confuzie în rândul programatorilor. Programatorii confuzi, la rândul lor, vor introduce greșeli (_buguri_) în program. Un program mare va oferi multe locuri în care bugurile acestea să se ascundă și va fi greu să fie găsite.

{{index "summing example"}}

Să revenim pe scurt la ultimele două exemple din introducere. Primul este complet și are șase linii:

```
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
```

Cel de-al doilea se bazează pe două funcții externe și are o singură linie.

```
console.log(sum(range(1, 10)));
```

Care dintre ele este mai probabil să conțină un bug?

{{index "program size"}}

Dacă luăm în considerare dimensiunea funcțiilor `sum` și `range` cel de-al doilea program este chiar mai mare decât primul. Dar este mult mai probabil să fie corect.

{{index [abstraction, "with higher-order functions"], "domain-specific language"}}

Este mai probabil să fie corect pentru că soluția este exprimată într-un vocabular adecvat problemei ce trebuie să fie rezolvată. Sumarea pe un interval de numere nu este despre bucle și contoare. Este despre intervale și sume.

Definițiile acestui vocabular (funcțiile `sum` și `range`) vor folosi bucle, contoare și alte detalii. Dar, deorece ele exprimă concepte mai simple decât programul ca și întreg, este mai ușor să fie implementate corect.

## Abstractizarea

În contextul programării, acești termeni de vocabular se numesc _abstractizări_. Abstractizările ascund detalii și ne permit să vorbim despre probleme la un nivel mai înalt (mai abstract).

{{index "recipe analogy", "pea soup"}}

Ca și o analogie, să comparăm aceste două rețete pentru a prepara supa de mazăre. Prima este exprimată astfel:

{{quote

Puneți o cupă de boabe uscate pentru fiecare persoană într-un vas. Adăugați apă până sunt acoperite boabele. Lăsați boabele în apă timp de cel puțin 12 ore. Scoateți boabele din apă și puneți-le într-un vas adecvat. Adăugați 4 căni de apă pentru fiecare persoană. Acoperiți vasul și lăsați-l la foc mic timp de două ore. Adăugați câte o jumătate de ceapă pentru fiecare persoană. Tăiați ceapa în cubulețe cu un cuțit. Apoi adăugați-o în vas. Adăugați câte o tulpină de țelină pentru fiecare persoană. Tăiați-o în bucăți cu un cuțit. Adăugați-o în vas. Adăugați câte un morcov pentru fiecare persoană. Tăiați-l în bucăți. Cu un cuțit! Adăugați-l în vas. Fierbeți timp de încă 10 minute.

quote}}

Iar aceasta este cea de a doua rețetă:

{{quote

Pentru o persoană: o cupă de boabe de mazăre, o jumătate de ceapă, o tulpină de țelină și un morcov.

Înmuiați boabele de mazăre timp de 12 ore. Fierbeți la foc încet pentru 2 ore în 4 căni de apă (de persoană). Tăiați și adăugați legumele. Fierbeți timp de încă 10 minute.

quote}}

{{index vocabulary}}

Cea de a doua rețetă este mai scurtă și mai ușor de interpretat. Dar trebuie să înțelegeți mai mulți termeni despre gătit, cum ar fi _a înmuia_, _a înăbuși_, _a tăia_ și, banuiesc, _legume_.

În programare, nu ne putem aștepta ca toate cuvintele de care avem nevoie să fie disponibile în dicționar. Astfel, am putea ajunge la situația primei rețete - trebuie să definim pașii exacți pe care computerul trebuie să îi execute, unul câte unul, ignorând conceptele de nivel superior pe care aceștia le exprimă.

{{index abstraction}}

Este un talent util ca și programator ca să vă dați seama atunci când lucrați la un nivel prea scăzut de abstractizare.

## Abstractizarea repetițiilor

{{index [array, iteration]}}

Funcțiile, așa cum am văzut, sunt un bun mod de a construi abstracții. Dar uneori nu sunt potrivite.

{{index "for loop"}}

Frecvent în programe repetăm un lucru de un număr dat de ori. Puteți scrie o buclă `for` astfel:

```
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

Putem abstractiza "să facem ceva de _N_ ori" ca o funcție? Desigur, este ușor să scriem o funcție care apelează `console.log` de _N_ ori.

```
function repeatLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

{{index [function, "higher-order"], loop, [function, "as value"]}}

{{indexsee "higher-order function", "function, higher-order"}}

Dar dacă vrem să executăm o altă operație în loc de `console.log`? Deoarece ceea ce vrem să facem poate fi reprezentat ca o funcție și funcțiile sunt doar valori, am putea transmite acțiunea dorită ca și argument al funcției.

```{includeCode: "top_lines: 5"}
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log);
// → 0
// → 1
// → 2
```

Nu e necesar să transmitem o funcție predefinită funcției `repeat`. Adesea este mai ușor să creem o valoare de tip funcție pe loc:

```
let labels = [];
repeat(5, i => {
  labels.push(`Unit ${i + 1}`);
});
console.log(labels);
// → ["Unit 1", "Unit 2", "Unit 3", "Unit 4", "Unit 5"]
```

{{index "loop body", [braces, body], [parentheses, arguments]}}

Această sintaxă este o structură asemănătoare unei bucle `for` - mai întâi descrie tipul de buclă și apoi definește un corp. Dar corpul este definit acum ca o valoare de tip funcție, inclusă între parantezele apelului la funcția `repeat`. De aceea trebuie să fie inclusă între acolade și paranteze de închidere. În cazuri ca și acest exemplu simplu, când corpul este o singură expresie, puteți omite acoladele și să scrieți bucla pe o singură linie.

## Funcții de ordin superior

{{index [function, "higher-order"], [function, "as value"]}}

Funcțiile care operează asupra altor funcții, fie primite ca și argumente, fie returnate, sunt numite _funcții de ordin superior_. Deoarece știm deja că funcțiile sunt valori, nu există nimic remarcabil relativ la faptul că asemenea funcții există. Termenul provine din matematică, unde distincția între funcții și alte valori este tratată mai serios.

{{index abstraction}}

Funcțiile de ordin superior ne permit să abstractizăm _acțiunile_, nu doar valorile. Ele vin în mai multe forme. De exemplu, putem avea funcții care crează noi funcții.

```
function greaterThan(n) {
  return m => m > n;
}
let greaterThan10 = greaterThan(10);
console.log(greaterThan10(11));
// → true
```

Și putem avea funcții care modifică alte funcții.

```
function noisy(f) {
  return (...args) => {
    console.log("calling with", args);
    let result = f(...args);
    console.log("called with", args, ", returned", result);
    return result;
  };
}
noisy(Math.min)(3, 2, 1);
// → calling with [3, 2, 1]
// → called with [3, 2, 1] , returned 1
```

Putem chiar să scriem funcții care crează noi modalități de control al execuției.

```
function unless(test, then) {
  if (!test) then();
}

repeat(3, n => {
  unless(n % 2 == 1, () => {
    console.log(n, "is even");
  });
});
// → 0 is even
// → 2 is even
```

{{index [array, methods], [array, iteration], "forEach method"}}

Există o metodă predefinită pe array, `forEach`, care ne oferă funcționalitate asemănătoare cu bucla `for`/`of` ca și o funcție de nivel superior.

```
["A", "B"].forEach(l => console.log(l));
// → A
// → B
```

## Seturi de caractere

O zonă în care funcțiile de ordin superior sunt extrem de utile este prelucrarea datelor. Pentru a prelucra date, mai întâi trebuie să le obținem. În acest capitol vom utiliza multțimi de date pentru seturi de caractere, cum ar fi Latin, Cyrillic și Arabic.

Vă amintiți despre Unicode din [capitolul ?](values#unicode), sistemul care asociază un număr fiecărui caracter din orice limbă scrisă? Standardul conține 140 seturi diferite de caractere - 81 încă folosite azi iar 59 adăugate din considerente istorice.

Deși pot citi fluent doar caractere Latin, respect faptul că oamenii scriu texte în cel puțin 80 alte de moduri de scriere, dintre care multe sunt chair de nerecunoscut pentru mine. De exemplu, iată cum arată o scriere de mână Tamil:

{{figure {url: "img/tamil.png", alt: "Scrierea Tamil"}}}

{{index "SCRIPTS data set"}}

Setul de caractere exemplificat conține unele informații despre cele 140 de secțiuni ale Unicode. Este disponibil la adresa ([_https://eloquentjavascript.net/code#5_](https://eloquentjavascript.net/code#5))]

```{lang: "application/json"}
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

Un asemenea obiect ne dă informații despre setul de caractere, domeniul de valori asociate în Unicode, direcția în care se scrie, originea aproximativă în timp, dacă mai este utilizat sau nu și un link util pentru a afla mai multe informații. Direcția poate fi `"ltr"` (de la stânga la dreapta - left to right), `"rtl"` (de la dreapta la stânga - right to left - cum se scriu textele în arabă sau ebraică) sau `"ttb"` (de sus în jos - top to bottom - ca și în mongolă).

{{index "slice method"}}

Proprietatea `ranges` conține un array de intervale Unicode, fiecare definit ca un array de două elemente ce conține limita inferioară și cea superioară. Orice coduri aparținând acestor intervale fac parte din respectivul set de caractere. Limita inferioară este inclusivă (codul 994 face parte din setul Coptic) iar limita superioară este exclusivă (codul 1008 nu face parte).

## Filtrarea array-urilor

{{index [array, methods], [array, filtering], "filter method", [function, "higher-order"], "predicate function"}}

Pentru a determina scrierile din setul de date care sunt încă utilizate, ar putea fi util următorul script, care filtrează elementele din array care nu trec un test:

```
function filter(array, test) {
  let passed = [];
  for (let element of array) {
    if (test(element)) {
      passed.push(element);
    }
  }
  return passed;
}

console.log(filter(SCRIPTS, script => script.living));
// → [{name: "Adlam", …}, …]
```

{{index [function, "as value"], [function, application]}}

Funcția utilizează argumentul `test`, o valoare de tip funcție, pentru a închide un "gol" în calcule - procesul de a decide care dintre elemente să fie colectat.

{{index "filter method", "pure function", "side effect"}}

Observați că funcția `filter`, în loc să elimine elemente din array-ul inițial, construiește un array nou ce conține doar elementele care trec testul. Această funcție este _pură_. Ea nu modifică array-ul primit.

Ca și `forEach`, `filter` este o metodă implicită pentru array-uri. Exemplul definește funcția doar pentru a demonstra cum funcționează aceasta intern. În continuare, o vom folosi astfel:

```
console.log(SCRIPTS.filter(s => s.direction == "ttb"));
// → [{name: "Mongolian", …}, …]
```

{{id map}}

## Transformarea cu `map`

{{index [array, methods], "map method"}}

Să presupunem că avem un array de obiecte ce reprezintă seturile de caractere, produs prin filtrarea array-ului `SCRIPTS`. Dar vrem să construim un array de nume, care este mai ușor de inspectat.

{{index [function, "higher-order"]}}

Metoda `map` transformă un array prin aplicarea unei funcții asupra fiecărui element al array-ului și construirea unui nou array cu valorile returnate. Noul array va avea aceeași lungime ca și array-ul de intrare, dar conținutul său va fi _mapat_ într-o nouă formă de către funcție.

```
function map(array, transform) {
  let mapped = [];
  for (let element of array) {
    mapped.push(transform(element));
  }
  return mapped;
}

let rtlScripts = SCRIPTS.filter(s => s.direction == "rtl");
console.log(map(rtlScripts, s => s.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", …]
```

Ca și `forEach` și `filter`, `map` este o metodă standard pentru array-uri.

## Rezumatul cu `reduce`

{{index [array, methods], "summing example", "reduce method"}}

O altă operație frecventă cu array-urile este de a calcula o singură valoare din colecția de elemente. Exemplul nostru recursiv, sumarea unei colecții de nume, este un exemplu bun. Alt exemplu este determinarea setului de caractere ce conține cele mai multe elemente.

{{indexsee "fold", "reduce method"}}

{{index [function, "higher-order"], "reduce method"}}

Operația de ordin superior care reprezintă acest șablon se numește _reduce_ (uneori denumită și _fold_). Ea construiește o valoare prin alegerea repetată a câte unui element din array și combinarea acestuia cu valoarea curentă. Când adunăm numerele, începem cu numărul zero și, pentru fiecare număr, îl adăugăm la sumă.

Parametrii funcției `reduce` sunt, pe lângă array-ul de prelucrat, o funcție de combinare și o valoare de start. Această funcție este puțin mai greu de înțeles decât `filter` sau `map`, așa că ar trebui să analizați cu atenție exemplul:

```
function reduce(array, combine, start) {
  let current = start;
  for (let element of array) {
    current = combine(current, element);
  }
  return current;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

{{index "reduce method", "SCRIPTS data set"}}

Metoda standard pentru array-uri `reduce`, care corespunde funcției de mai sus, mai are o convenție. Dacă array-ul conține cel puțin un element, puteți să nu transmiteți argumentul `start`. Metoda va considera primul element ca și valoare de start și va reduce array-ul începând cu al doilea element.

```
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

{{index maximum, "characterCount function"}}

Pentru a utiliza `reduce` (de două ori) ca să găsim scrierea cu cele mai multe caractere, putem scrie un cod asemănător celui de mai jos:

```
function characterCount(script) {
  return script.ranges.reduce((count, [from, to]) => {
    return count + (to - from);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return characterCount(a) < characterCount(b) ? b : a;
}));
// → {name: "Han", …}
```

Funcția `characterCount` reduce intervalele asociate unei scrieri prin sumarea dimensiunilor lor. Observați utilizarea destructurării în lista de parametri a funcției reducătoare. Cel de al doilea apel al funcției `reduce` utilizează apoi valoarea returnată de prima funcție pentru a determina scrierea cu cele mai multe caractere prin compararea repetată a două scrieri și a returna scrierea mai cuprinzătoare.

Scrierea Han are mai mult de 89000 de caractere asociate în standardul Unicode, ceea ce o face de departe cel mai cuprinzător sistem de scriere din setul de date. Han este o scriere utilizată uneori pentru texte în chineză, japoneză sau coreană. Aceste limbi utilizează în comun multe caractere deși tind să le utilizeze în mod diferit. Consorțiul Unicode (U.S.) a decis să le grupeze într-un singur sistem de scriere pentru a economisi numărul de coduri utilizate. Aceasta este așa numita _unificare Han_ și incă înfurie pe unii utilizatori.

## Compozabilitate

{{index loop, maximum}}

Să vedem cum am fi scris soluția din exemplul de mai sus fără a utiliza funcții de ordin superior (ca să determinăm scrierea cu cele mai multe caractere). Codul nu arată chiar așa de rău.

```{test: no}
let biggest = null;
for (let script of SCRIPTS) {
  if (biggest == null ||
      characterCount(biggest) < characterCount(script)) {
    biggest = script;
  }
}
console.log(biggest);
// → {name: "Han", …}
```

Se utilizează câteva bindinguri în plus și programul este cu patru linii mai lung. Dar este încă foarte lizibil.

{{index "average function", composability, [function, "higher-order"], "filter method", "map method", "reduce method"}}

{{id average_function}}

Funcțiile de ordin superior încep să strălucească atunci când este necesar să _compunem_ operațiile. De exemplu, haideți să scriem codul care determină media anului de origine pentru scrieri utilizate în curent și scrieri istorice din setul de date.

```
function average(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(average(
  SCRIPTS.filter(s => s.living).map(s => s.year))));
// → 1165
console.log(Math.round(average(
  SCRIPTS.filter(s => !s.living).map(s => s.year))));
// → 204
```

Deci, scrierile istorice sunt în medie mai vechi decât cele încă utilizate în prezent. Aceasta nu este o statistică cu o semnificație extraordinară sau surprinzătoare. Dar sper că sunteți de acord că nu este dificil de urmărit codul care calculează acest rezultat. Îl puteți privi ca pe o conductă: începem cu tot setul de date, îl filtrăm pentru scrierile de care avem nevoie, extragem anii, calculăm media și rotunjim rezultatul.

Ați fi putut cu siguranță să scrieți toată logica într-o singură buclă:

```
let total = 0, count = 0;
for (let script of SCRIPTS) {
  if (script.living) {
    total += script.year;
    count += 1;
  }
}
console.log(Math.round(total / count));
// → 1165
```

Dar este mai dificil de înțeles ce și cum urmează să fie calculat. Și, deoarece rezultatele intermediare nu sunt reprezentate ca și valori coerente, ar fi mult mai dificil de a extrage o valoare cum ar fi funcția `average` într-o bucată separată de cod.

{{index efficiency, [array, creation]}}

În ceea ce privește operațiile efectuate de către computer, cele două abordări sunt destul de diferite. Prima va construi noi array-uri când se execută `filter` și `map` în timp ce cea de a doua va calcula niște numere, efectuând mai puține operații. De regulă vă puteți permite prima abordare, cea mai lizibilă, dar atunci când prelucrați array-uri de mari dimensiuni, abordarea mai puțin abstractă ar putea să merite câștigul de viteză.

## Stringuri și codurile caracterelor

{{index "SCRIPTS data set"}}

O utilizare a setului de date ar putea fi pentru a determina scrierea pe care o utilizează o anumită bucată de text. Haideți să construim un program care realizează aceasta.

Amintiți-vă ca fiecare scriere utilizează un array de intervale asociate cu ea. Astfel, dat fiind un cod al unui caracter, putem utiliza o funcție ca și cea care urmează pentru a determina scrierea din care face parte:

{{index "some method", "predicate function", [array, methods]}}

```{includeCode: strip_log}
function characterScript(code) {
  for (let script of SCRIPTS) {
    if (script.ranges.some(([from, to]) => {
      return code >= from && code < to;
    })) {
      return script;
    }
  }
  return null;
}

console.log(characterScript(121));
// → {name: "Latin", …}
```

Metoda `some` este o altă funcție de ordin superior. Ia primește ca argument o funcție de test și ne returnează dacă acea funcție de test returnează true pentru măcar unul dintre elementele array-ului.

{{id code_units}}

Dar cum putem obține codurile caracterelor dintr-un string?

În [capitolul ?](values) am precizat că stringurile JavaScript sunt codificate ca o secvență de numere pe 16 biți. Acestea sunt numite _unități de cod_. Inițial, un cod Unicode se presupunea că poate fi codificat într-o asemenea unitate (ceea ce permitea codificarea a aproximativ 65000 caractere). Când a devenit evident că numărul acesta este prea mic, au apărut voci care susțineau că trebuie să folosim mai multă memorie pentru fiecare caracter. Pentru a adresa aceste probleme, a fost inventat formatul UTF-16, utilizat pentru stringurile JavaScript. Acest standard descrie caracterele cele mai frecvent utilizate ca o singură unitate de cod pe 16 biți dar pentru unele caractere utilizează o pereche de unități de cod.

{{index error}}

UTF-16 este considerat o idee proastă în prezent. Pare a fi conceput intenționat pentru a provoca greșeli. Este ușor să scriem programe care pretind că nu există nici o diferență între unități de cod și caractere. Dacă scrierea pe care o utilizați nu conține caractere reprezentate pe două unități, acesta va părea că funcționează fără probleme. Dar imediat ce un alt programator încearcă să utilizeze un asemenea program utilizând caractere mai puțin folosite (cum ar fi cele din limba chineză), programul eșuează. Din fericire, odată cu apariția caracterelor emoji toată lumea a început să folosească caractere reprezentate pe două unități de cod și presiunea de a rezolva asemenea probleme este distribuită mai uniform.

{{index [string, length], [string, indexing], "charCodeAt method"}}

Din păcate, operații evidente asupra stringurilor JavaScript, cum ar fi determinarea lungimii cu ajutorul proprietății `length` precum și accesarea conținutului lor în paranteze pătrate, utilizează doar unități de cod.

```{test: no}
// Two emoji characters, horse and shoe
let horseShoe = "🐴👟";
console.log(horseShoe.length);
// → 4
console.log(horseShoe[0]);
// → (Invalid half-character)
console.log(horseShoe.charCodeAt(0));
// → 55357 (Code of the half-character)
console.log(horseShoe.codePointAt(0));
// → 128052 (Actual code for horse emoji)
```

{{index "codePointAt method"}}

Metoda JavaScript `charCodeAt` vă returnează o unitate de cod, nu un caracter complet. Metoda `codePointAt`, adăugată ulterior, vă returnează de fapt un caracter Unicode complet. Am putea să o utilizăm pentru a obține caracterele unui string. Dar argumentul transmis metodei `codePointAt` este tot un index într-o secvență de unități de cod. Prin urmare, pentru a parcurge toate caracterele dintr-un string, va trebui să răspundem la întrebarea dacă un caracter folosește una sau două unități de cod.

{{index "for/of loop", character}}

În [capitolul anterior](data#for_of_loop), am menționat că o buclă `for`/`of` poate fi utilizată și pentru stringuri. Ca și `codePointAt`, acest tip de buclă a fost introdus atunci când programatorii erau conștienți cu privire la problemele generate de utilizarea UTF-16. Când folosiți acest tip de buclă pentru a itera pe un string, veți obține caracterele reale, nu unități de cod.

```
let roseDragon = "🌹🐉";
for (let char of roseDragon) {
  console.log(char);
}
// → 🌹
// → 🐉
```

Când aveți un caracter (care va fi un string de una sau două unități de cod), puteți utiliza `codePointAt(0)` pentru a obține codul său.

## Recunoașterea textului

{{index "SCRIPTS data set", "countBy function", [array, counting]}}

Avem funcția `characterScript` și o modalitate de a itera corect caracterele. Următorul pas este de a contoriza caracterele care corespund fiecărei scrieri. Următoarea abstracție pentru numărare va fi utilă:

```{includeCode: strip_log}
function countBy(items, groupName) {
  let counts = [];
  for (let item of items) {
    let name = groupName(item);
    let known = counts.findIndex(c => c.name == name);
    if (known == -1) {
      counts.push({name, count: 1});
    } else {
      counts[known].count++;
    }
  }
  return counts;
}

console.log(countBy([1, 2, 3, 4, 5], n => n > 2));
// → [{name: false, count: 2}, {name: true, count: 3}]
```

Funcția `countBy` primește o colecție (orice poate fi iterat cu `for`/`of`) și o funcție care determină numele grupului pentru un element dat. Va returna un array de obiecte, fiecare reprezentând numele unui grup și numărul de elemente ce fac parte din acel grup.

{{index "findIndex method", "indexOf method"}}

Utilizăm o altă metodă - `findIndex`. Această metodă este asemănătoare cu `indexOf`, dar, în loc să caute o anumită valoare, caută prima valoare pentru care funcția dată returnează `true`. Ca și `indexOf`, returnează -1 dacă nu găsește un asemenea element.

{{index "textScripts function", "Chinese characters"}}

Folosind `countBy` putem scrie funcția care determină care scrieri sunt folosite într-un text.

```{includeCode: strip_log, startCode: true}
function textScripts(text) {
  let scripts = countBy(text, char => {
    let script = characterScript(char.codePointAt(0));
    return script ? script.name : "none";
  }).filter(({name}) => name != "none");

  let total = scripts.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No scripts found";

  return scripts.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(textScripts('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

{{index "characterScript function", "filter method"}}

Prima funcție numără caracterele după nume, utilizând `characterScript` pentru a le asocia unui nume și folosind valoarea implicită `"none"` pentru caracterele care nu fac parte din nici o scriere. Apelul către funcția `filter` elimină caracterele din grupul `"none"` deoarece acestea nu prezintă interes.

{{index "reduce method", "map method", "join method", [array, methods]}}

Pentru a calcula procentajele, mai întâi trebuie să determinăm numărul total de caractere pentru fiecare scriere, ceea ce putem face folosind `reduce`. Dacă nu determinăm nici un asemenea caracter, funcția returnează un string specific. Altfel, transformă intrările contorizate în stringuri lizibile, folosind `map` și apoi combinându-le cu `join`.

## Rezumat

Abilitatea de a transmite valori de tip funcție altor funcții este un aspect profund util în JavaScript. Astfel putem scrie funcții care modelează calcule cu "goluri" în ele. Codul care apelează aceste funcții va umple golurile cu ajutorul unor valori de tip funcție.

Pentru array-uri avem la dispoziție câteva metode utile de ordin superior. Putem utiliza bucla `forEach` pentru a itera peste elementele unui array. metoda `filter` ne returnează un array ce conține doar elementele care trec peste testul funcției predicat. Putem transforma fiecare element al unui array folosind metoda `map`. Putem folosi `reduce` pentru a combina elementele unui array într-o singură valoare. Metoda `some` testează dacă există măcar un singur element în array care verifică funcția predicat. Iar `findIndex` determină poziția primului element care satisface predicatul.

## Exerciții

### Aplatizarea (Flattening)

{{index "flattening (exercise)", "reduce method", "concat method", [array, flattening]}}

Utilizați metoda `reduce` în combinație cu metoda `concat` pentru a "aplatiza" un array de array-uri într-un array pe un singur nivel ce conține toate elementele array-urilor originale.

{{if interactive

```{test: no}
let arrays = [[1, 2, 3], [4, 5], [6]];
// Your code here.
// → [1, 2, 3, 4, 5, 6]
```
if}}

### Propria versiune de buclă

{{index "your own loop (example)", "for loop"}}

Scrieți o funcție de ordin superior `loop` care execută o operație asemănătoare unei instrucțiuni `for`. Va primi ca și argumente o valoare, o funcție de test, o funcție de actualizare și o funcție pentru corpul acțiunii. La fiecare iterație, mai întâi va rula funcția test asupra valorii curente a buclei și se va opri dacă testul este fals. Apoi va apela funcția pentru corpul buclei, având ca și parametru valoarea curentă. Apoi va apela funcția pentru actualizare pentru a crea o nouă valoare și va reîncepe execuția.

Când definiți funcția, puteți utiliza o buclă obișnuită pentru iterare.

{{if interactive

```{test: no}
// Your code here.

loop(3, n => n > 0, n => n - 1, console.log);
// → 3
// → 2
// → 1
```

if}}

### Totul

{{index "predicate function", "everything (exercise)", "every method", "some method", [array, methods], "&& operator", "|| operator"}}

Analog metodei `some`, array-urile au și o metodă `every`. Aceasta returnează `true` doar dacă funcția dată returnează true pentru fiecare element al array-ului. Dintr-un anume punct de vedere, `some` este o versiune de `||` asupra unui array iar `every` este o versiune asemănătoare operatorului `&&`.

Implementați `every` ca și o funcție care primește un array și o funcție predicat ca și argumente. Scrieți două versiuni, una care utilizează o buclă și alta care utilizează metoda `some`.

{{if interactive

```{test: no}
function every(array, test) {
  // Your code here.
}

console.log(every([1, 3, 5], n => n < 10));
// → true
console.log(every([2, 4, 16], n => n < 10));
// → false
console.log(every([], n => n < 10));
// → true
```

if}}

{{hint

{{index "everything (exercise)", "short-circuit evaluation", "return keyword"}}

Ca și operatorul `&&`, metoda `every` poate să încheie evaluarea elementelor atunci când determină că elementul curent nu se potrivește. Astfel încât versiunea bazată pe buclă poate încheia execuția folosind `break` sau `return` imediat ce detectează că, pentru un anume element, funcția predicat returnează `false`. Dacă bucla se execută până la final fără să detecteze un astfel de element, știm că toate elementele au fost potrivite și putem returna `true`.

Pentru a construi `every` cu ajutorul `some` putem aplica _legile lui De Morgan_ care afirmă că `a && b` este egal cu `!(!a || !b)`. Generalizând pentru array-uri, putem spune că toate elementele din array se potrivesc dacă nu există nici un element în array care nu se potrivește.

hint}}

### Direcția dominantă de scriere

{{index "SCRIPTS data set", "direction (writing)", "groupBy function", "dominant direction (exercise)"}}

Scrieți o funcție care calculează direcția dominantă de scriere într-un text. Vă reamintesc că fiecare obiect ce descrie o scriere are o proprietate `direction` care poate fi `"ltr"` (stânga-dreapta), `"rtl"` (dreapta-stânga) sau `"ttb"` (sus-jos).

{{index "characterScript function", "countBy function"}}

Direcția dominantă este direcția majorității caracterelor care au o scriere asociată. Probabil funcțiile `characterScript` și `countBy` pe care le-am implementat anterior ar putea fi utile.

{{if interactive

```{test: no}
function dominantDirection(text) {
  // Your code here.
}

console.log(dominantDirection("Hello!"));
// → ltr
console.log(dominantDirection("Hey, مساء الخير"));
// → rtl
```
if}}

{{hint

{{index "dominant direction (exercise)", "textScripts function", "filter method", "characterScript function"}}

Soluția ar putea fi foarte asemănătoare cu prima parte a exemplului `textScripts`. Din nou va trebui să numărați caracterele pe baza unui criteriu bazat pe `characterScript` și apoi să eliminați prin filtrare caracterele neinteresante.

{{index "reduce method"}}

Determinarea direcției cu cel mai mare număr de caractere se poate determina cu `reduce`. Dacă nu vă e clar cum, studiați exemplul anterior din acest capitol, în care am folosit `reduce` pentru a determina scrierea cu cele mai multe caractere.

hint}}
