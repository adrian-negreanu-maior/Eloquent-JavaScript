# Expresii regulate

{{quote {author: "Jamie Zawinski", chapter: true}

Unele persoane, când se confruntă cu o problemă, gândesc: 'Știu, o să folosesc expresii regulate'. Acum au două probleme.

quote}}

{{index "Zawinski, Jamie"}}

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Yuan-Ma a spus: 'Când tăiați împotriva fibrei lemnului, este necesară multă putere. Când programați împotriva fibrei problemei, este necesar mult cod.'

quote}}

if}}

{{figure {url: "img/chapter_picture_9.jpg", alt: "A railroad diagram", chapter: "square-framed"}}}

{{index evolution, adoption, integration}}

Instrumentele și tehnicile de programare supraviețuiesc și se răspândesc intr-un mod haotic, evolutionar. Nu întotdeauna câștigă cele mai drăguțe sau mai inteligente, ci cele care funcționează suficient de bine într-o nișă sau care sunt integrate cu o altă piesă de succes din tehnologie.

{{index "domain-specific language"}}

În acest capitol vom discuta despre un asemenea instrument, _expresiile regulate_. Expresiile regulate sunt un mod de a descrie șabloane în datele de tip string. Ele formează un mic limbaj separat care face parte din JavaScript și alte limbaje și sisteme.

{{index [interface, design]}}

Expresiile regulate sunt atât teribil de ciudate cât și incredibil de utile. Sintaxa lor este criptică iar intefața de programare pusă la dispoziție de către JavaScript este stângace. Dar ele reprezintă un instrument puternic pentru inspectarea și procesarea stringurilor. Înțelegerea corespunzătoare a expresiilor regulate vă va transforma într-un programator mai eficient.

## Crearea unei expresii regulate

{{index ["regular expression", creation], "RegExp class", "literal expression", "slash character"}}

O expresie regulată este un tip de obiect. Ea poate fi construită fie cu ajutorul constructorului `RegExp` fie ca o valoare literală prin includerea șablonului între caractere `/`.

```
let re1 = new RegExp("abc");
let re2 = /abc/;
```

Ambele expresii de mai sus reprezintă același șablon: un _a_ urmat de _b_ și apoi de _c_.

{{index ["backslash character", "in regular expressions"], "RegExp class"}}

Când se utilizează un constructor `RegExp`, șablonul este scris ca un string, astfel încât se aplică regulile uzuale pentru `\`.

{{index ["regular expression", escaping], [escaping, "in regexps"], "slash character"}}

Cea de a doua notație, în care șablonul se definește între caractere `/` tratează caracterele `\` oarecum diferit. Mai întâi, deoarece `/` marchează finalul șablonului, trebuie să utilizăm `\` înainte de un `/` care vrem să facă parte din șablon. În plus, caracterele `\` care nu sunt parte a unui cod cu semnificație specială vor fi _păstrate_ și vor schimba semnificația șablonului (nu trebuie să le dublăm). Unele caractere, cum ar fi `?` și `+` au semnificații speciale în expresiile regulate și trebuie să le prefixăm cu `\` dacă vrem să reprezinte caractere normale.

```
let eighteenPlus = /eighteen\+/;
```

## Testarea pentru potrivire

{{index matching, "test method", ["regular expression", methods]}}

Obiectele de tip expresii regulate au câteva metode. Cea mai simplă este `test`. Dacă i se transmite ca argument un string, va returna o valoare booleană care ne va preciza dacă stringul conține o potrivire pentru șablonul definit de expresie.

```
console.log(/abc/.test("abcde"));
// → true
console.log(/abc/.test("abxde"));
// → false
```

{{index pattern}}

O expresie regulată ce conține doar caractere fără semnificație specială reprezintă acea secvență de caractere. Dacă _abc_ apare oriunde în stringul pe care îl testăm, metoda `test` va returna `true`.

## Seturi de caractere

{{index "regular expression", "indexOf method"}}

Verificarea dacă un string conține expresia _abc_ se poate face pur și simplu printr-un apel al `indexOf`. Expresiile regulate ne permit să exprimăm șabloane mai complicate.

Să presupunem că vrem să potrivim orice număr. Într-o expresie regulată, plasarea unui set de caractere între paranteze pătrate înseamnă că acea parte a expresiei regulate potrivește oricare dintre caracterele dintre paranteze.

Ambele expresii de mai jos potrivesc toate stringurile care conțin o cifră:

```
console.log(/[0123456789]/.test("in 1992"));
// → true
console.log(/[0-9]/.test("in 1992"));
// → true
```

{{index "hyphen character"}}

Între parantezele pătrate, un caracter `-` între două caractere se poate utiliza pentru a defini un interval de caractere, unde ordinea este determinată de codul Unicode al caracterelor. Caracterele de la 0 la 9 sunt ordonate (codurile de la 48 la 57), astfel încât `[0-9]` acoperă toate cifrele și potrivește oricare cifră.

{{index [whitespace, matching], "alphanumeric character", "period character"}}

Unele grupuri de caractere frecvent utilizate au propriile scurtături. Cifrele sunt unul dintre aceste grupuri: `\d` înseamnă același lucru ca și `[0-9]`.

{{index "newline character", [whitespace, matching]}}

{{table {cols: [1, 5]}}}

| `\d`    | Orice caracter cifră
| `\w`    | Orice caracter alfanumeric (literă sau cifră)
| `\s`    | Orice caracter spațiu (space, tab, newline, și altele)
| `\D`    | Orice caracter care _nu_ este o cifră
| `\W`    | Orice caracter care nu este alfanumeric
| `\S`    | Orice caracter care nu este spațiu
| `.`     | Orice caracter cu excepția newline

Deci, veți putea potrivi un format dată-și-oră cum ar fi 01-30-2003 15:20 folosind următoarea expresie:

```
let dateTime = /\d\d-\d\d-\d\d\d\d \d\d:\d\d/;
console.log(dateTime.test("01-30-2003 15:20"));
// → true
console.log(dateTime.test("30-jan-2003 15:20"));
// → false
```

{{index ["backslash character", "in regular expressions"]}}

Arată îngrozitor, nu-i așa? Jumătate dintre caractere sunt `\`, ceea ce produce un zgomot de fundal care face dificilă sarcina de a identifica șablonul definit. Vom vedea o versiune îmbunătățită a acestei expresii [puțin mai târziu](regexp#date_regexp_counted).

{{index [escaping, "in regexps"], "regular expression", set}}

Aceste coduri pot fi utilizate și între parantezele pătrate. De exemplu, `[\d.]` înseamnă _orice cifră sau caracterul `.`_ Dar caracterul `.` își pierde semnificația specială. Același lucru este valabil și pentru alte caractere cu semnificație specială, cum ar fi `+`.

{{index "square brackets", inversion, "caret character"}}

Pentru a _inversa_ un set de caractere - adică să precizăm că se potrivește oricare caracter, cu _excepția_ celor din mulțime, putem folosi `^` după paranteza dreaptă deschisă.

```
let notBinary = /[^01]/;
console.log(notBinary.test("1100100010100110"));
// → false
console.log(notBinary.test("1100100010200110"));
// → true
```

## Repetarea părților unui șablon

{{index ["regular expression", repetition]}}

Știm cum să potrivim o singură cifră. Dar dacă vrem să căutăm un număr - adică o secvență de una sau mai multe cifre?

{{index "plus character", repetition, "+ operator"}}

Atunci când plasăm simbolul `+` după ceva într-o expresie regulată, indicăm astfel că elementul se poate repeta de mai multe ori. Prin urmare, `/\d+/` potrivește unul sau mai multe caractere de tip cifră.

```
console.log(/'\d+'/.test("'123'"));
// → true
console.log(/'\d+'/.test("''"));
// → false
console.log(/'\d*'/.test("'123'"));
// → true
console.log(/'\d*'/.test("''"));
// → true
```

{{index "* operator", asterisk}}

Simbolul `*` are o semnificație similară dar permite și potrivirea de zero ori. Un element cu `*` după el nu previne potrivirea pentru un șablon - va potrivi zero instanțe dacă nu găsește nici un text potrivit.

{{index "British English", "American English", "question mark"}}

Un semn de întrebare face ca o parte a unui șablon să fie _opțională_, adică poate să nu apară deloc sau să apară o singură dată. În exemplul următor, caracterul _u_ poate să apară, dar șablonul este potrivit și dacă _u_ lipsește.

```
let neighbor = /neighbou?r/;
console.log(neighbor.test("neighbour"));
// → true
console.log(neighbor.test("neighbor"));
// → true
```

{{index repetition, [braces, "in regular expression"]}}

Pentru a preciza că o parte a șablonului trebuie să apară de un anumit număr de ori, folosim acolade. Plasând `{4}` după un element, de exemplu, solicităm ca elementul respectiv să apară de exact patru ori. Putem și să specificăm un interval pentru numărul de apariții, astfel: `{2,4}` înseamnă că elementul trebuie să apară de cel puțin două ori și de cel mult patru ori.

{{id date_regexp_counted}}

Iată o altă versiune a șablonului pentru dată-și-oră care permite potrivirea mai flexibilă a zilei, lunii și orei. De asemenea, aș zice că este puțin mai ușor de descifrat:

```
let dateTime = /\d{1,2}-\d{1,2}-\d{4} \d{1,2}:\d{2}/;
console.log(dateTime.test("1-30-2003 8:45"));
// → true
```

Puteți și să specificați intervale deschise când utilizați acoladele, prin omiterea numărului de după virgulă. Astfel, `{5,}` înseamnă "de cinci sau mai multe ori".

## Grouping subexpressions

{{index ["regular expression", grouping], grouping, [parentheses, "in regular expressions"]}}

Pentru a utliza operatori cum ar fi `*` sau `+` pe mai multe elemente, trebuie să utilizați paranteze. O parte a unei expresii regulate care este inclusă între paranteze reprezintă un singur element din punct de vedere al operatorilor care îi urmează.

```
let cartoonCrying = /boo+(hoo+)+/i;
console.log(cartoonCrying.test("Boohoooohoohooo"));
// → true
```

{{index crying}}

Primul și al doilea caracter `+` din exemplul de mai sus se aplică doar asupra celui de-al doilea _o_ din _boo_ și _hoo_. Cel de-al treilea `+` se aplică întregului grup `(hoo+)`, potrivind una sau mai multe secvențe de acest fel.

{{index "case sensitivity", capitalization, ["regular expression", flags]}}

Caracterul `i` de la sfârșitul expresiei din exemplu specifică faptul că nu se va face diferența dintre literele mari și mici (case insensitive), ceea ce permite potrivirea _B_ din stringul de intrare, deși sablonul este format doar cu litere mici.

## Potriviri și grupuri

{{index ["regular expression", grouping], "exec method", [array, "RegExp match"]}}

Metoda `test` este cel mai simplu mod de a potrivi o expresie regulată. Ea doar returnează dacă expresia a fost potrivită sau nu și nimic altceva. Expresie regulate au și o metodă `exec` care va returna `null` dacă nu a găsit nici o potrivire și un obiect cu informații despre potrivire, dacă aceasta a reușit.

```
let match = /\d+/.exec("one two 100");
console.log(match);
// → ["100"]
console.log(match.index);
// → 8
```

{{index "index property", [string, indexing]}}

Un obiect returnat de către `exec` are o proprietate `index` care ne spune _unde_ în string se află începutul potrivirii. În rest, obiectul arată ca un array de stringuri, al cărui prim element este stringul care a fost potrivit. În exemplul de mai sus, se returnează secvența de cifre pe care o căutam.

{{index [string, methods], "match method"}}

Valorile de tip string au o metodă `match` care se comportă similar.

```
console.log("one two 100".match(/\d+/));
// → ["100"]
```

{{index grouping, "capture group", "exec method"}}

Când expresia regulată conține subexpresii grupate în paranteze, textul care a potrivit acele grupuri va fi și el parte din array. Potrivirea completă este întotdeauna primul element. Următorul element este partea potrivită de către primul grup (cel al cărui paranteză deschisă apare prima în expresie), apoi al doilea grup, și așa mai departe.

```
let quotedText = /'([^']*)'/;
console.log(quotedText.exec("she said 'hello'"));
// → ["'hello'", "hello"]
```

{{index "capture group"}}

Când un grup nu poate fi potrivit deloc (de exemplu, când este urmat de un simbol `?`), poziția sa în array-ul rezultat va memora `undefined`. Similar, când un grup este potrivit de mai multe ori, numai ultima potrivire va face parte din array.

```
console.log(/bad(ly)?/.exec("bad"));
// → ["bad", undefined]
console.log(/(\d)+/.exec("123"));
// → ["123", "3"]
```

{{index "exec method", ["regular expression", methods], extraction}}

Grupurile pot fi utile pentru a extrage părți dintr-un string. Dacă nu vrem doar să verificăm dacă un string conține o dată, ci vrem să o extragem și să construim un obiect care o reprezintă, putem pune paranteze în jurul șablonului de cifre și apoi selectăm data din rezultatul returnat de `exec`.

Dar mai întâi facem un ocol și discutăm modul implicit în care JavaScript reprezintă valorile dată-și-oră.

## Clasa Date

{{index constructor, "Date class"}}

JavaScript are o clasă standard pentru reprezentarea datelor calendaristice - sau, mai degrabă, a punctelor în timp. Se numește `Date`. Dacă pur și simplu creați un nou obiect de acest tip, folosind `new`, obțineți data și ora curentă.

```{test: no}
console.log(new Date());
// → Mon Nov 13 2017 16:19:11 GMT+0100 (CET)
```

{{index "Date class"}}

Dar puteți și să creați un obiect pentru o data specificată:

```
console.log(new Date(2009, 11, 9));
// → Wed Dec 09 2009 00:00:00 GMT+0100 (CET)
console.log(new Date(2009, 11, 9, 12, 59, 59, 999));
// → Wed Dec 09 2009 12:59:59 GMT+0100 (CET)
```

{{index "zero-based counting", [interface, design]}}

JavaScript utilizează o convenție în care lunile sunt numerotate începând cu zero (December este 11), dar zilele sunt numerotate începând cu 1. Atenție la această confuzie stupidă.

Ultimele patru argumente (ora, minutul, secunda și milisecundele) sunt opționale și inițializate cu zero dacă nu sunt transmise.

{{index "getTime method"}}

Timestamp-urile sunt memorate ca și numărul de milisecunde scurse de la începutul anului 1970, în zona UTC. Aceasta urmărește convenția setată de "Unix time", care a fost inventată cam în aceași perioadă. Puteți utiliza valori negative pentru timpi anteriori 1970. Metoda `getTime` a unui obiect returnează acest număr. Este un număr mare, așa cum vă puteți imagina.

```
console.log(new Date(2013, 11, 19).getTime());
// → 1387407600000
console.log(new Date(1387407600000));
// → Thu Dec 19 2013 00:00:00 GMT+0100 (CET)
```

{{index "Date.now function", "Date class"}}

Dacă transmiteți constructorului `Date` un singur argument, acesta este interpretat ca și un contor în milisecunde (timestamp). Valoarea acestui contor pentru momentul curent o puteți determina prin crearea unui nou obiect `Date` și apelarea metodei sale `getTime` sau prin apelarea funcției `Date.now`.

{{index "getFullYear method", "getMonth method", "getDate method", "getHours method", "getMinutes method", "getSeconds method", "getYear method"}}

Obiectele `Date` au metode cum ar fi `getFullYear`, `getMonth`, `getDate`, `getHours`, `getMinutes`, și `getSeconds` pentru a extrage părți ale lor. Pe lângă `getFullYear` există și `getYear`, care returnează anul minus 1900 (`98` sau `119`) și care este cam inutilă.

{{index "capture group", "getDate method", [parentheses, "in regular expressions"]}}

Plasând paranteze în jurul părților din expresie care ne interesează, putem să creem un obiect Date dintr-un string.

```
function getDate(string) {
  let [_, month, day, year] =
    /(\d{1,2})-(\d{1,2})-(\d{4})/.exec(string);
  return new Date(year, month - 1, day);
}
console.log(getDate("1-30-2003"));
// → Thu Jan 30 2003 00:00:00 GMT+0100 (CET)
```

{{index destructuring, "underscore character"}}

Bindingul `_` este ignorat și utilizat doar pentru a prelua elementul ce reprezintă potrivirea completă din array-ul returnat de `exec`.

## Limite ale cuvintelor și șirurilor

{{index matching, ["regular expression", boundary]}}

Din păcate, funcția `getDate` de mai sus va extrage și data fără sens 00-1-3000 din stringul `"100-1-30000"`. O potrivire poate avea loc oriunde în string astfel încât va începe la al doilea caracter și se va termina la penultimul.

{{index boundary, "caret character", "dollar sign"}}

Dacă vrem să specificăm că potrivirea trebuie să aibă loc pe tot stringul, putem adăuga marcajele `^` și `$`. Primul precizează începutul stringului iar cel de-al doilea sfârșitul. Astfel, `/^\d+$/` potrivește un string format doar din cifre, `/^!/` potrivește orice string care începe cu un semn de exclamare, iar `/x^/` nu potrivește nici un string (nu putem avea un _x_ înaintea începutului stringului).

{{index "word boundary", "word character"}}

Dacă, pe de altă parte, vrem doar să ne asigurăm că data începe și se termină la limita unui cuvânt, putem folosi marcajul `\b`. O limită a unui cuvânt poate fi începutul sau sfârșitul string-ului sau orice punct din string care are un caracter alfanumeric într-o parte (ca și în `\w`) și un caracter non-alfanumeric în cealaltă parte.

```
console.log(/cat/.test("concatenate"));
// → true
console.log(/\bcat\b/.test("concatenate"));
// → false
```

{{index matching}}

Menționez că un marcaj de limită nu se potrivește peste un anume caracter. Doar obligă potrivirea expresiei regulate doar atunci când o anumită condiție este adevărată în poziția în care apare în șablon.

## Șsabloane alternative

{{index branching, ["regular expression", alternatives], "farm example"}}

Să presupunem că vrem să știm dacă o anumită bucată de text conține un număr urmat de unul dintre cuvintele _pig_, _cow_ sau _chicken_ sau forma de plural a acestor cuvinte.

Am putea scrie trei expresii diferite și apoi să le testăm peste string, dar există o modalitate mai simplă. Caracterul `|` denotă o alegere între două variante de potrivire. Deci, am putea scrie:

```
let animalCount = /\b\d+ (pig|cow|chicken)s?\b/;
console.log(animalCount.test("15 pigs"));
// → true
console.log(animalCount.test("15 pigchickens"));
// → false
```

{{index [parentheses, "in regular expressions"]}}

Putem utiliza paranteze pentru a delimita partea din șsablon pe care se aplică operatorul `|` și putem folosi mai mulți asemenea operatori pentru a exprima o alegere între mai multe alternative.

## Mecanica potrivirii

{{index ["regular expression", matching], [matching, algorithm], "search problem"}}

Conceptual, când utilizați `exec` sau `test`, motorul de expresii regulate caută o potrivire începând cu primul caracter din string, apoi cu al doilea caracter, și așa mai departe, până când găsește o potrivire sau ajunge la sfârșitul stringului. Fie va returna prima potrivire pe care o găsește, fie va eșua în a găsi o potrivire.

{{index ["regular expression", matching], [matching, algorithm]}}

Pentru a realiza potrivirea propriu-zisă, motorul tratează expresia regulată ca pe o diagramă de flux. În figura de mai jos vă prezint diagrama asociată expresiei anterioare:

{{figure {url: "img/re_pigchickens.svg", alt: "Visualization of /\\b\\d+ (pig|cow|chicken)s?\\b/"}}}

{{index traversal}}

Expresia noastră se potrivește dacă putem găsi o cale din partea stângă a diagramei până în partea dreaptă. Păstrăm poziția curentă în string și de câte ori ne deplasăm la o cutie, verificăm că partea din string de după poziția curentă se potrivește cu cutia.

De exemplu, dacă încercăm să potrivim `"the 3 pigs"` din poziția 4, progresul nostru prin diagrama de flux ar arăta cam așa:

 - În poziția 4, există o limită de cuvânt, deci putem trece de prima cutie.

 - Tot în poziția 4, găsim o cifră, deci putem trece de a doua cutie.

 - În poziția 5, o cale se întoarce înapoi înainte de a doua cutie iar cealaltă merge mai departe spre cutia care reprezintă un singur spațiu. Există un spațiu în string, așa că mergem pe a doua cale.

 - Acum suntem în poziția 6 (începutul _pigs_) și avem trei căi. Nu vedem _cow_ sau _chicken_, dar vedem _pig_, așa că alegem această cale

 - În poiziția 9, o cale sare peste cutia _s_ și merge la limita de final de cuvânt, iar cealaltă potrivește un _s_. Există un caracter _s_ așa că vom alege calea prin cutia _s_.

 - Suntem acum în poziția 10 (sfârșitul stringului) și putem potrivi doar o limită de cuvânt. Sfârșitul stringului este o limită, deci trecem și de ultima cutie și am reușit să potrivim cu succes acest string.

{{id backtracking}}

## Backtracking

{{index ["regular expression", backtracking], "binary number", "decimal number", "hexadecimal number", "flow diagram", [matching, algorithm], backtracking}}

Expresia regulată `/\b([01]+b|[\da-f]+h|\d+)\b/` potrivește fie un număr binar, urmat de un _b_, fie un număr hexadecimal (adică în baza 16, cu literele _a_ până la _f_ în locul cifrelor de la 10 la 15) urmat de un _h_, sau un număr zecimal normal, fără caracter de sufix. Diagrama corespunzătoare o puteți consulta în imaginea de mai jos:

{{figure {url: "img/re_number.svg", alt: "Visualization of /\\b([01]+b|\\d+|[\\da-f]+h)\\b/"}}}

{{index branching}}

Când potrivim această expresie, prima cale (binar) este parcursă chiar dacă nu avem un număr binar. Când încercăm potrivirea stringului `"103"`, de exemplu, doar când ajungem la 3 ne dăm seama că suntem pe calea greșită. Stringul _potrivește_ expresie, dar nu pe calea pe care suntem.

{{index backtracking, "search problem"}}

Motorul de potrivire va reveni. Când intră pe o anumită cale, reține poziția curentă (care în cazul nostru este începutul stringului, imediat după după prima cutie de limită de cuvânt din diagramă) astfel încât se va putea întoarce și să încerce o altă cale dacă cea curentă nu funcționează. Pentru stringul `"103"`, după detectarea caracterului 3, va trece la începutul căii pentru numere hexadecimale, care va eșua pentru că nu există _h_ după număr. Astfel va ajunge să exploreze calea pentru numere zecimale. Aceasta se va potrivi și se va raporta o potrivire.

{{index [matching, algorithm]}}

Motorul de potrivire se va opri imediat ce determină o potrivire completă. Aceasta înseamnă că, dacă există mai multe căi care ar putea potrivi un string, doar prima (determinată de ordinea în care căile apar în expresia regulată) va fi utilizată.

Backtrackingul are loc și pentru operatorii de repetiție `+` și `*`. Dacă potriviți `/^.*x/` peste `"abcxe"`, partea `.*` va încerca mai întâi să consume întregul string. Apoi motorul își va da seama că trebuie să găsească un _x_ pentru a potrivi șablonul. Deoarece nu există nici un _x_ după sfârșitul stringului, operatorul `*` încearcă să potrivească cu un caracter mai puțin. Dar motorul nu găsește nici un `x` după `abcx`, așa că revine cu un caracter mai în față, potrivind `*` cu stringul `abc`. Acum găsește un _x_ unde are nevoie și raportează o potrivire pentru pozițiile de la 0 la 4.

{{index performance, complexity}}

Putem scrie expresii regulate care vor efectua mult backtracking. Această problemă apare atunci când un șablon poate potrivi o parte din input în multe moduri. De exemplu, dacă suntem confuzi când scriem expresia regulată pentru un număr binar, am putea scrie accidental ceva de genul `/([01]+)+b/`.

{{figure {url: "img/re_slow.svg", alt: "Visualization of /([01]+)+b/",width: "6cm"}}}

{{index "inner loop", [nesting, "in regexps"]}}

Dacă încercăm să potrivim o serie lungă de zero și unu fără caracterul _b_, motorul mai întâi va executa bucla interioară până când nu mai găsește cifre. Apoi constată că nu există un _b_ și revine pe o poziție anterioară, trece prin bucla anterioară din nou și apoi renunță, încercând să revină din bucla interioară cu încă un caracter. Va continua să încerce fiecare rută posibilă pe aceste două bucle. Prin urmare, efortul se va dubla pentru fiecare caracter adăugat. Chiar și pentru un numar de câteva zeci de caractere, potrivirea va dura extrem de mult.

## Metoda replace

{{index "replace method", "regular expression"}}

Valorile de tip string au o metodă `replace` care permite înlocuirea unei părți din string cu un alt string.

```
console.log("papa".replace("p", "m"));
// → mapa
```

{{index ["regular expression", flags], ["regular expression", global]}}

Primul argument poate fi și o expresie regulată, în care caz se înlocuiește prima potrivire a expresiei regulate. Când opțiunea `g` (_global_) este adăugată expresiei regulate, vor fi înlocuite toate potrivirile, nu doar prima.

```
console.log("Borobudur".replace(/[ou]/, "a"));
// → Barobudur
console.log("Borobudur".replace(/[ou]/g, "a"));
// → Barabadar
```

{{index [interface, design], argument}}

Ar fi fost inteligent dacă alegerea între înlocuirea primei apariții și înlocuirea tuturor aparițiilor ar fi fost controlată de către un argument opțional al metodei `replace` sau prin implementarea unei alte metode, `replaceAll`. Dintr-un motiv nefericit, alegerea se bazează însă pe o proprietate a expresiei regulate.

{{index grouping, "capture group", "dollar sign", "replace method", ["regular expression", grouping]}}

Adevărata putere a utilizării expresiilor regulate cu `replace` derivă din faptul că putem referi grupurile potrivite în stringul de înlocuire. De exemplu, avem un string foarte lung care conține numele unor persoane, câte unul pe linie, în formatul `Lastname, Firstname`. Dacă vrem să schimbăm formatul în `Firstname Lastname`, putem utiliza următorul cod:

```
console.log(
  "Liskov, Barbara\nMcCarthy, John\nWadler, Philip"
    .replace(/(\w+), (\w+)/g, "$2 $1"));
// → Barbara Liskov
//   John McCarthy
//   Philip Wadler
```

`$1` și `$2`  din stringul de înlocuire se referă la grupurile dintre paranteze din șablon. `$1` se înlocuiește cu textul potrivit pentru primul grup, iar `$2` reprezintă cel de-al doilea grup, și așa mai departe până la `$9`. Potrivirea completă poate fi referită prin `$&`.

{{index [function, "higher-order"], grouping, "capture group"}}

Putem transmite o funcție în loc de un string - ca și cel de-al doilea argument al `replace`. Pentru fiecare înlocuire, funcția va fi apelată cu grupurile potrivite (precum și potrivirea completă) ca și argumente și valoarea returnată va fi inserată în noul string.

Iată un scurt exemplu:

```
let s = "the cia and fbi";
console.log(s.replace(/\b(fbi|cia)\b/g,
            str => str.toUpperCase()));
// → the CIA and FBI
```

Și unul mai interesant:

```
let stock = "1 lemon, 2 cabbages, and 101 eggs";
function minusOne(match, amount, unit) {
  amount = Number(amount) - 1;
  if (amount == 1) { // only one left, remove the 's'
    unit = unit.slice(0, unit.length - 1);
  } else if (amount == 0) {
    amount = "no";
  }
  return amount + " " + unit;
}
console.log(stock.replace(/(\d+) (\w+)/g, minusOne));
// → no lemon, 1 cabbage, and 100 eggs
```

Acesta preia un string, găsește toate aparițiile unor numere urmate de un cuvânt și returnează un string în care fiecare asemenea apariție este decrementată cu 1.

Grupul `(\d+)` devine argumentul `amount` al funcției, iar grupul `(\w+)` va fi transmis în parametrul `unit`. Funcția convertește `amount` la un number - ceea ce va funcționa întotdeauna deoarece a potrivit `\d+` - și apoi ajustează cuvântul în cazul în care numărul este 0 sau 1.

## Lăcomie

{{index greed, "regular expression"}}

Putem folosi `replace` pentru a scrie o funcție care elimină toate comentariile dintr-o bucată de cod JavaScript. Iată o primă încercare:

```{test: wrap}
function stripComments(code) {
  return code.replace(/\/\/.*|\/\*[^]*\*\//g, "");
}
console.log(stripComments("1 + /* 2 */3"));
// → 1 + 3
console.log(stripComments("x = 10;// ten!"));
// → x = 10;
console.log(stripComments("1 /* a */+/* b */ 1"));
// → 1  1
```

{{index "period character", "slash character", "newline character", "empty set", "block comment", "line comment"}}

Partea din stânga operatorului _or_ potrivește două caractere `/` urmate de oricâte caractere altele decât linie nouă. Cea de a doua parte, pentru comentarii pe mai multe linii, este mai elaborată. Utilizăm `[^]` (orice caracter care nu face parte din setul gol de caractere) ca și o modalitate de a potrivi orice caracter. Nu putem să utilizăm `.` deoarece comentariile de tip block pot continua pe mai multe linii și caracterul `.` nu potrivește caractere linie nouă.

Însă iețirea de pe ultima linie pare greșită. De ce?

{{index backtracking, greed, "regular expression"}}

Partea `[^]*` a expresiei, așa cum am descris în secțiunea despre backtracking, va potrivi mai întâi cât de mult poate. Dacă aceasta cauzează nepotrivirea următoarei părți a șablonului, motorul de potrivire se mută un caracter înapoi și încearcă din nou. În exemplu, motorul încearcă mai întâi să potrivească tot stringul și apoi execută backtracking-ul. Va determina o apariție a `*/` după ce s-a întors 4 caractere și o va potrivi. Dar nu aceasta a fost intenția noastră, ceea ce doream era să identificăm un singur comentariu, nu să mergem până la sfârșitul stringului și să identificăm marcajul de final al ultimului comentariu de tip bloc.

Din cauza acestui comportament, spunem că operatorii de repetiție (`+`, `*`, `?`, și `{}`) sunt _lacomi_, adică potrivesc cât de mult pot și utilizează backtracking. Dacă plasați un semn de întrebare după operator (`+?`, `*?`, `??`, `{}?`), atunci modificați comportamentul lot și vor încerca să potrivească un string cât mai mic. Vor potrivi un string mai lung doar dacă partea rămasă din șablon nu potrivește un string mai scurt.

Exact asta ne doream în acest caz. Să potrivim `*` cu cea mai scurtă secvență de caractere care ne conduce la `*/` și astfel să identificăm un singur bloc de comentarii.

```{test: wrap}
function stripComments(code) {
  return code.replace(/\/\/.*|\/\*[^]*?\*\//g, "");
}
console.log(stripComments("1 /* a */+/* b */ 1"));
// → 1 + 1
```

Multe buguri în programele care utilizează expresii regulate pot fi identificate ca utilizare neintenționată a unui operator _lacom (greedy)_. Atunci când utilizați un operator de repetiție, luați în calcul varianta non-greedy mai întâi.

## Crearea dinamică a obiectelor RegExp

{{index ["regular expression", creation], "underscore character", "RegExp class"}}

Există cazuri în care nu cunoașteți șablonul exact pentru potrivire atunci când scrieți codul. Să presupunem că vreți să identificați numele utilizatorului într-o bucată de text și să îl plasați între caractere `_` pentru a-l evidenția. Deoarece veți cunoaște numele doar atunci când programul va fi executat, nu puteți utiliza notația cu `/`.

Dar puteți construi un string și să utilizați constructorul `RegExp` asupra lui. Iată un exemplu:

```
let name = "harry";
let text = "Harry is a suspicious character.";
let regexp = new RegExp("\\b(" + name + ")\\b", "gi");
console.log(text.replace(regexp, "_$1_"));
// → _Harry_ is a suspicious character.
```

{{index ["regular expression", flags], ["backslash character", "in regular expressions"]}}

Când creem markerii `\b` trebuie să îi precizăm `\\b` deoarece îi utilizăm într-un string. Cel de-al doilea argument al constructorului `RegExp` reprezintă opțiunile pentru expresia regulată - în acest caz `gi` pentru global și fără diferență între literele mari și mici.

Dar dacă numele este `"dea+hl[]rd"` pentru că utilizatorul nostru are un "nume de scenă" ciudat? Am obține o expresie regulată fără sens care de fapt nu va detecta potrivirile pentru numele utilizatorului.

{{index ["backslash character", "in regular expressions"], [escaping, "in regexps"], ["regular expression", escaping]}}

O soluție ar fi să adăugăm `\` înaintea oricărui caracter cu semnificație specială.

```
let name = "dea+hl[]rd";
let text = "This dea+hl[]rd guy is super annoying.";
let escaped = name.replace(/[\\[.+*?(){|^$]/g, "\\$&");
let regexp = new RegExp("\\b" + escaped + "\\b", "gi");
console.log(text.replace(regexp, "_$&_"));
// → This _dea+hl[]rd_ guy is super annoying.
```

## Metoda `search`

{{index ["regular expression", methods], "indexOf method", "search method"}}

Metoda `indexOf` pentru stringuri nu poate fi apelată și pentru expresiile regulate. Dar există o altă metodă, `search`, care așteaptă o expresie regulată. Ca și `indexOf`, returnează primul index la care expresia a fost găsită și -1 dacă nu a găsit expresia.

```
console.log("  word".search(/\S/));
// → 2
console.log("    ".search(/\S/));
// → -1
```

Din păcate, nu putem preciza un index la care să înceapă căutarea (așa cum utilizăm cel de-al doilea argument al `indexOf`), care ar fi adesea util.

## Proprietatea `lastIndex`

{{index "exec method", "regular expression"}}

Similar, metoda `exec` nu ne oferă un mod convenabil de a începe căutarea dintr-o anumită poziție a stringului. Dar ne oferă o modalitate *ne*convenabilă de a specifica aceasta.

{{index ["regular expression", matching], matching, "source property", "lastIndex property"}}

Obiectele de tip expresii regulate au proprietăți. O asemenea proprietate este `source`, care conține stringul din care a fost creată expresia. O altă proprietate este `lastIndex`, care controlează, în anumite circumstanțe limitate, poziția din care se va începe următoarea potrivire.

{{index [interface, design], "exec method", ["regular expression", global]}}

Acele circumstanțe se referă la faptul că expresia regulată trebuie să aibă opțiunile global (`g`) sau sticky (`y`) activate, iar potrivirea trebuie să se execute prin metoda `exec`. Din nou, o soluție mai puțin creatoare de confuzii ar fi fost adăugarea unui extra-argument care să fie transmis funcției `exec`, dar confuzia este o caracteristică esențială a interfeței JavaScript pentru expresii regulate.

```
let pattern = /y/g;
pattern.lastIndex = 3;
let match = pattern.exec("xyzzy");
console.log(match.index);
// → 4
console.log(pattern.lastIndex);
// → 5
```

{{index "side effect", "lastIndex property"}}

Dacă potrivirea s-a realizat cu succes, apelul către `exec` actualizează automat proprietatea `lastIndexOf` ca să indice după potrivire. Dacă nu s-a găsit nici o potrivire, atunci `lastIndex` se va seta înapoi la zero, care este și valoarea implicită atunci când se construiește un nou obiect de tip expresie regulată.

Diferența dintre opțiunile "global" și "sticky" este că, atunci când "sticky" este permis, potrivirea se va face cu succes numai dacă începe exact după `lastIndex`, în timp ce "global" va căuta în avans o poziție unde potrivirea poate să înceapă.

```
let global = /abc/g;
console.log(global.exec("xyz abc"));
// → ["abc"]
let sticky = /abc/y;
console.log(sticky.exec("xyz abc"));
// → null
```

{{index bug}}

Când se utilizează o expresie regulată pentru mai multe apeluri ale `exec`, aceste actualizări automate ale proprietății `lastIndex` pot cauza probleme. Expresia regulată ar putea începe accidental la un index setat de către un apel anterior.

```
let digit = /\d/g;
console.log(digit.exec("here it is: 1"));
// → ["1"]
console.log(digit.exec("and now: 1"));
// → null
```

{{index ["regular expression", global], "match method"}}

Un alt efect interesant al opțiunii "global" este modificarea modului în care metoda `match` funcționează pe stringuri. Când va fi apelată cu o expresie globală, în loc să returneze un array similar cu cel returnat de către `exec`, `match` va găsi _toate_ potrivirile șablonului în string și va returna un array ce conține toate stringurile potrivite.

```
console.log("Banana".match(/an/g));
// → ["an", "an"]
```

Deci, aveți grijă cu expresiile regulate globale. Cazurile in care sunt necesare - apeluri ale `replace` și locuri în care intenționați să folosiți explicit `lastIndex` - sunt de regulă singurele locuri în care ați vrea să le utilizați.

### Iterarea peste potriviri

{{index "lastIndex property", "exec method", loop}}

O acțiune frecventă este scanarea tuturor aparițiilor unui șablon într-un string, într-un mod care să ne permită accesul la obiectul de potrivire în corpul buclei. Putem realiza aceasta prin utilizarea `lastIndex` și `exec`.

```
let input = "A string with 3 numbers in it... 42 and 88.";
let number = /\b\d+\b/g;
let match;
while (match = number.exec(input)) {
  console.log("Found", match[0], "at", match.index);
}
// → Found 3 at 14
//   Found 42 at 33
//   Found 88 at 40
```

{{index "while loop", ["= operator", "as expression"], [binding, "as state"]}}

Aceasta utilizează faptul că valoarea unei expresii de atribuire (`=`) este valoarea atribuită. Deci, utilizând `match = number.exec(input)` ca și condișie în instrucțiunea `while`, realizăm potrivirea la începutul fiecărei iterații, salvăm rezultatul într-un binding și ne oprim din repetiție atunci când nu mai găsim potriviri.

{{id ini}}
## Parsarea unui fișier INI

{{index comment, "file format", "enemies example", "INI file"}}

Pentru a finaliza acest capitol, vom analiza o problemă care necesită utilizarea expresiilor regulate. Imaginați-vă că scriem un program pentru colectarea automată a informațiilor despre inamicii noștri de pe Internet. (Nu vom scrie de fapt acest program, ci doar partea care citește fișierul de configurare). Fișierul de configurare ar arăta cam așa:

```{lang: "text/plain"}
searchengine=https://duckduckgo.com/?q=$1
spitefulness=9.7

; comments are preceded by a semicolon...
; each section concerns an individual enemy
[larry]
fullname=Larry Doe
type=kindergarten bully
website=http://www.geocities.com/CapeCanaveral/11451

[davaeorn]
fullname=Davaeorn
type=evil wizard
outputdir=/home/marijn/enemies/davaeorn
```

{{index grammar}}

Regulile exacte pentru acest format (care este utilizat pe larg, de regulă denumit fișier_INI_) sunt următoarele:

- Liniile goale și liniile care încep cu `;` sunt ignorate.
- Liniile între `[` și `]` încep o nouă secțiune.
- Liniile ce conțin un identificator alfanumeric urmat de `=` adaugă o setare la secțiunea curentă.
- Orice altceva este invalid.

Sarcina noastră este de a converti un string de acest fel într-un obiect ale cărui proprietăți rețin stringuri pentru setările scrise înainte de prima secțiune și subobiecte pentru secțiuni, fiecare subobiect reținând setările secțiunii.

{{index "carriage return", "line break", "newline character"}}

Deoarece acest format trebuie procesat linie cu linie, împărțirea fișierului în linii separate este un start bun. Am studiat deja metoda `split` în [capitolul ?](data#split). Unele sisteme de operare utilizează o combinație de caractere pentru sfârșitul de linie (`"\r\n"`). 
Fiindcă metoda `split` acceptă și expresii regulate ca și argumente, am putea utiliza expresia regulată `/\r?\n/` pentru a permite atât `"\n"` cât și `"\r\n"` între linii.

```{startCode: true}
function parseINI(string) {
  // Start with an object to hold the top-level fields
  let result = {};
  let section = result;
  string.split(/\r?\n/).forEach(line => {
    let match;
    if (match = line.match(/^(\w+)=(.*)$/)) {
      section[match[1]] = match[2];
    } else if (match = line.match(/^\[(.*)\]$/)) {
      section = result[match[1]] = {};
    } else if (!/^\s*(;.*)?$/.test(line)) {
      throw new Error("Line '" + line + "' is not valid.");
    }
  });
  return result;
}

console.log(parseINI(`
name=Vasilis
[address]
city=Tessaloniki`));
// → {name: "Vasilis", address: {city: "Tessaloniki"}}
```

{{index "parseINI function", parsing}}

Codul parcurge liniile fișierului și construiește un obiect. Proprietățile de la început sunt memorate direct în obiect, iar proprietățile identificate în secțiuni sunt stocate în obiecte separate pentru fiecare secțiune. Bindingul `section` se referă la obiectul pentru secțiunea curentă.

Există două feluri de linii semnificative - headere de secțiuni și linii care setează proprietăți. Când o linie se referă la o prorietate, este memorată în secțiunea curentă. Când linia reprezintă un header de secțiune, se crează un nou obiect pentru secțiune iar `section` este setat să se refere la acesta.

{{index "caret character", "dollar sign", boundary}}

Observați utilizarea recurentă a `^` și `$` pentru a ne asigura că expresia este potrivită pentru toată linia, nu doar pentru o parte a ei. Dacă le eliminăm, vom obține un cod care de cele mai multe ori funcționează dar care se comportă ciudat pentru anumite date de intrare, ceea ce ar fi un bug dificil de identificat.

{{index "if keyword", assignment, ["= operator", "as expression"]}}

Șablonul `if (match = string.match(...))` este similar cu trick-ul de a utiliza o atribuire ca și condiție pentru `while`. De multe ori nu sunteți siguri că apelul metodei `match` va avea succes, astfel încât veți putea folosi obiectul rezultat doar în interiorul unei instrucțiuni `if` care testează acest lucru. Pentru a nu distruge plăcutul lanț de forme `else if`, atribuim rezultatul potrivirii unui binding și apoi utilizăm imediat acea atribuire ca și test pentru instrucțiunea `if`.

{{index [parentheses, "in regular expressions"]}}

Dacă o linie nu reprezintă un header de ssecțiune sau o proprietate, funcția verifică dacă linia reprezintă un comentariu sau o linie goală cu ajutorul expresiei `/^\s*(;.*)?$/`. Înțegeți cum funcționează? Partea dintre paranteze va potrivi comentariile iar simbolul `?` asigură că va potrivi și liniile care conțin doar spații albe. Când o linie nu potrivește nici una dintre formele așteptate, funcția aruncă o excepție.

## Caractere internaționale

{{index internationalization, Unicode, ["regular expression", internationalization]}}

Urmare a implementării inițiale simpliste din JavaScript și a faptului că aceasta a fost ulterior standardizată, expresiile regulate din JavaScript sunt mai degrabă proaste atunci când au de a face cu caractere care nu fac parte din setul limbii engleze. De exemplu, în contextul expresiilor regulate, un "caracter al unui cuvânt" este doar unul dintre cele 26 de simboluri ale alfabetului latin (litere mari și mici), o cifră zecimală și, din oarecare motiv, simbolul underscore `_`. Caractere cum ar fi _é_ sau _β_, care cu siguranță pot face parte dintr-un cuvânt, nu vor potrivi `\w` (dar vor potrivi `\W`, categoria simbolurilor care nu fac parte dintr-un cuvânt).

{{index [whitespace, matching]}}

Printr-un accident istoric ciudat, `\s` (spații albe) nu are această problemă și potrivește toate caracterele considerate spații albe în standardul Unicode, inclusiv "non-breaking space" și separatorul pentru vocale în scrierea mongolă.

O altă problemă este cauzată de faptul că, implicit, expresiile regulate funcționează pe unități de cod, așa cum am discutat în [capitolul ?](higher_order#code_units), și nu pe caracterele propriu-zise. Aceasta înseamnă că acele caractere compuse din două unități de cod se vor comporta ciudat.

```
console.log(/🍎{3}/.test("🍎🍎🍎"));
// → false
console.log(/<.>/.test("<🌹>"));
// → false
console.log(/<.>/u.test("<🌹>"));
// → true
```

Problema este cauzată de faptul că 🍎 din prima linie este tratat ca un caracter format din două unități iar `{3}` se aplică doar celei de-a doua unități de cod. Similar, `.` potrivește o singură unitate de cod, nu ambelor unități ce formează codul pentru trandafir.

Pentru a trata corect asemenea caractere, trebuie să adăugați opțiunea `u` la expresia regulată (pentru Unicode). Comportamentul greșit rămâne însă cel implicit, deoarece modificarea acestuia ar cauza probleme de compatibilitate cu codul deja existent.

{{index "character category", [Unicode, property]}}

Deși încă nu este suportată pe larg și a fost standardizată de curând, puteți folosi opțiunea `\p` într-o expresie regulată (care trebuie să aibă activată opțiunea pentru Unicode) pentru a potrivi toate caracterele cărora standardul Unicode le asociază o anumită proprietate.

```{test: never}
console.log(/\p{Script=Greek}/u.test("α"));
// → true
console.log(/\p{Script=Arabic}/u.test("α"));
// → false
console.log(/\p{Alphabetic}/u.test("α"));
// → true
console.log(/\p{Alphabetic}/u.test("!"));
// → false
```

Unicode definește câteva proprietăți utile, dar găsirea celei de care aveți nevoie nu este întotdeauna o sarcină trivială, Puteți utiliza notația `\p{Property=Value}` pentru a potrivi orice caracter care are valoarea dată pentru respectiva proprietate. Dacă omiteți numele proprietății, ca și în `\p{Name}`, numele se presupune a fi fie o proprietate binară, cum ar fi `Alphabetic` fie o categorie, cum ar fi `Number`.

{{id summary_regexp}}

## Rezumat

Expresiile regulate sunt obiecte care reprezinttă șabloane în stringuri. Ele utilizează propriul limbaj pentru a defini asemenea șabloane.

{{table {cols: [1, 5]}}}

| `/abc/`     | O secvență de caractere
| `/[abc]/`   | Orice caracter dintr-un set de caractere
| `/[^abc]/`  | Orice caracter din afara unui set de caractere
| `/[0-9]/`   | Orice caracter dintr-un interval de caractere
| `/x+/`      | Una sau mai multe apariții ale șablonului `x`
| `/x+?/`     | Una sau mai multe apariții, non-greedy
| `/x*/`      | Zero sau mai multe apariții
| `/x?/`      | Zero sau o singură apariție
| `/x{2,4}/`  | Două până la patru apariții
| `/(abc)/`   | Un grup
| `/a|b|c/`   | Oricare dintre mai multe șabloane
| `/\d/`      | Orice caracter cifră
| `/\w/`      | Orice caracter alfanumeric ("word character")
| `/\s/`      | Orice caracter spațiu
| `/./`       | Orice caracter cu excepția caracterului linie-nouă
| `/\b/`      | Limita unui cuvânt
| `/^/`       | Începutul stringului
| `/$/`       | Sfârșitul stringului

O expresie regulată are o metodă `test` pentru a testa dacă un string dat o potrivește. Mai are și o metodă `exec` care, atunci când găsește o potrivire, returnează un array cu toate grupurile potrivite. Un asemenea array are o proprietate `index` care arată unde a început potrivirea.

Stringurile au metoda `match` pentru a le potrivi relativ la o expresie regulată precum și o metodă `search` pentru a căuta o potrivire, care returnează doar poziția de început a potrivirii. Cu ajutorul metodei `replace` putem înlocui potrivirile șablonului cu un string de înlocuire sau cu ajutorul unei funcții.

Expresiile regulate pot avea opțiuni, care sunt scrise după `/` de final. Opțiunea `i` permite potrivirea fără a face diferența între literele mari și cele mici, opțiunea `g` face ca expresia să fie _globală_, ceea ce , printre altele, cauzează înlocuirea tuturor potrivirilor în metoda `replace`, nu doar a primeia. Opțiunea `y` face ca expresia să devină "sticky", ceea ce înseamnă că nu va căuta în avans sau sări peste părți ale stringului atunci când caută o potrivire. Opțiunea `u` activează modul Unicode, ceea ce fixează mai multe probleme legate de tratarea caracterelor reprezentate pe două unități de cod.

Expresiile regulate sunt o unealtă bine ascuțită cu un mâner destul de ciudat. Ele simplifică extrem demult anumite sarcini dar pot deveni repede greu de gestionat când sunt aplicate unor probleme complexe. Parte a cunoașterii lor este și cum să le evitați atunci când ele nu pot exprima corect ceea ce doriți.

## Exerciții

{{index debugging, bug}}

Este aproape inevitabil ca, în timp ce lucrați la aceste exerciții, sa deveniți confuzi și frustrați de comportamentul inexplicabil al unora dintre expresiile regulate. Uneori este util să introduceți expresia regulată într-un instrument online cum ar fi [_https://debuggex.com_](https://www.debuggex.com/) pentru a verifica dacă vizualizarea sa corespunde cu ceea ce intenționați și pentru a experimenta modul în care aceasta răspunde la diferitele stringuri de intrare.

### Regexp golf

{{index "program size", "code golf", "regexp golf (exercise)"}}

_Code golf_ este un termen utilizat pentru jocul de a încerca să exprimați un anume program cu cât mai puține caractere. Similar, _regexp golf_ este practica de a scrie o expresie regulată cât mai scurtă pentru a potrivi un anume șsablon și _doar_ acel șablon.

{{index boundary, matching}}

Pentru fiecare dintre itemii de mai jos scrieți o expresie regulată pentru a testa dacă oricare dintre substringurile date apare în string. Expresia regulată trebuie să potrivească doar stringurile ce conțin unul dintre substringurile date. Nu vă preocupați despre limitele cuvintelor decât dacă se menționează explicit. După ce reușiți să faceți expresia funcțională, încercați să o scurtați.

 1. _car_ și _cat_
 2. _pop_ și _prop_
 3. _ferret_, _ferry_, și _ferrari_
 4. Orice cuvânt ce se termină în _ious_
 5. Orice spațiu urmat de punct, virgulă, două puncte sau punct și virgulă.
 6. Un cuvânt mai lung de 6 liutere
 7. Un cuvânt fără litera _e_ (sau _E_)

Consultați tabelul din [rezumatul capitolului](regexp#summary_regexp) ca ajutor. Testați fiecare soluție cu câteva stringuri.

{{if interactive
```
// Fill in the regular expressions

verify(/.../,
       ["my car", "bad cats"],
       ["camper", "high art"]);

verify(/.../,
       ["pop culture", "mad props"],
       ["plop", "prrrop"]);

verify(/.../,
       ["ferret", "ferry", "ferrari"],
       ["ferrum", "transfer A"]);

verify(/.../,
       ["how delicious", "spacious room"],
       ["ruinous", "consciousness"]);

verify(/.../,
       ["bad punctuation ."],
       ["escape the period"]);

verify(/.../,
       ["Siebentausenddreihundertzweiundzwanzig"],
       ["no", "three small words"]);

verify(/.../,
       ["red platypus", "wobbling nest"],
       ["earth bed", "learning ape", "BEET"]);


function verify(regexp, yes, no) {
  // Ignore unfinished exercises
  if (regexp.source == "...") return;
  for (let str of yes) if (!regexp.test(str)) {
    console.log(`Failure to match '${str}'`);
  }
  for (let str of no) if (regexp.test(str)) {
    console.log(`Unexpected match for '${str}'`);
  }
}
```

if}}

### Stilul de citare

{{index "quoting style (exercise)", "single-quote character", "double-quote character"}}

Imaginați-vă că ați scris o poveste și ați folosit apostroafe peste tot pentru a marca o bucată de dialog. Acum vreți să utilizați ghilimele pentru dialoguri dar să păstrați apostroafele în contrageri cum ar fi _aren't_.

{{index "replace method"}}

Gândiți-vă la un șablon care distinge între cele două modalități de utilizare a apostroafelor și gândiți un apel către metoda `replace` care realizează înlocuirea.

{{if interactive
```{test: no}
let text = "'I'm the cook,' he said, 'it's my job.'";
// Change this call.
console.log(text.replace(/A/g, "B"));
// → "I'm the cook," he said, "it's my job."
```
if}}

{{hint

{{index "quoting style (exercise)", boundary}}

Cea mai evidentă soluție este să înlocuiți doar apostroafele care au un caracter non-alfanumeric în cel puțin o parte, ceva în genul `/\W'|'\W/`. Dar ar trebui să luați în calcul și începutul sau sfârșitul liniei.

{{index grouping, "replace method", [parentheses, "in regular expressions"]}}

În plus, trebuie să vă asigurați că înlocuirea include și caracterele care au fost potrivite de către șablonul `W` astfel încât acestea să nu fie eliminate. Putem realiza aceasta prin utilizarea parantezelor și includerea grupurilor în stringul de înlocuire (`$1`, `$2`). Grupurile care nu sunt potrivite nu vor fi înlocuite.

hint}}

### Din nou despre numere

{{index sign, "fractional number", [syntax, number], minus, "plus character", exponent, "scientific notation", "period character"}}

Scrieți o expresie care să potrivească doar numerele scrise în stilul acceptat de către JavaScript. Trebuie să permită un simbol opțional plus sau minus în fața numărului, punctul zecimal, precum și notația cu exponent - `5e-3` sau `1E10` - din nou cu un semn opțional în fața exponentului. De asemenea, de menționat că prezența cifrelor înainte sau după punctul zecimal nu este obligatorie. Adică, `.5` și `5.` sunt numere valide în JavaScript, dar punctul zecimal în sine nu este.

{{if interactive
```{test: no}
// Fill in this regular expression.
let number = /^...$/;

// Tests:
for (let str of ["1", "-1", "+15", "1.55", ".5", "5.",
                 "1.3e2", "1E-4", "1e+12"]) {
  if (!number.test(str)) {
    console.log(`Failed to match '${str}'`);
  }
}
for (let str of ["1a", "+-1", "1.2.3", "1+1", "1e4.5",
                 ".5.", "1f5", "."]) {
  if (number.test(str)) {
    console.log(`Incorrectly accepted '${str}'`);
  }
}
```

if}}

{{hint

{{index ["regular expression", escaping], ["backslash character", "in regular expressions"]}}

Mai întâi, nu uitați de `\` în fața punctului.

Potrivirea semnului opțional în fața numărului, precum și în fața exponentului, se poate face cu expresia `[+\-]?` sau `(\+|-|)`
(plus, minus, sau nici un caracter).

{{index "pipe character"}}

Partea mai complicată a exercițiului este potrivirea `"5."` și `".5"` fără a potrivi și `"."`. Pentru aceasta, o soluție bună ar fi utilizarea operatorului `|` pentru a separa cele două cazuri - fie una sau mai multe cifre urmate de un punct opțional și apoi de zero sau mai multe cifre, fie un punct urmat de una sau mai multe cifre.

{{index exponent, "case sensitivity", ["regular expression", flags]}}

În final, pentru a potrivi _e_ fără a face diferența față de _E_ folosiți fie opțiunea `i` pentru expresia regulată, fie `[eE]`.

hint}}