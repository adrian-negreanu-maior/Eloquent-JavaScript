# Structura programelor

{{figure {url: "img/chapter_picture_2.jpg", alt: "Picture of tentacles holding objects", chapter: framed}}}

În acest capitol vom începe să realizăm lucruri care se pot numi _programare_. Ne vom extinde cunoștințele despre limbajul JavaScript dincolo de denumirile si fragmentele de propoziții cu care am venit în contact până acum, până în punctul în care vom putea scrie proză cu semnificație.

## Expresii și instrucțiuni

{{index grammar, [syntax, expression], [code, "structure of"], grammar, [JavaScript, syntax]}}

În capitolul [?](values), am creat valori și am aplicat operatori asupra lor pentru a crea noi valori. Crearea valorilor în acest mod este principala activitate în orice program JavaScript. Dar această activitate trebuie să fie plasată într-un context mai larg pentru a fi cu adevărat utilă. Aceste aspecte le vom acoperi în cele ce urmează.

{{index "literal expression", [parentheses, expression]}}

Un fragment de cod care produce o valoare se numește _expresie_. Orice valoare scrisă (cum ar fi `22` sau `"psihoanaliză"`) este o expresie. O expresie plasată între paranteze este tot o expresie, așa cum tot expresie este și o construcție în care un operator binar se aplică asupra a două expresii sau un operator unar se aplică asupra unei singure expresii.

{{index [nesting, "of expressions"], "human language"}}

Aceasta ilustrează o parte din frumusețea unei interfețe bazate pe limbaj. Expresiile pot conține alte expresii într-un mod similar cu modul în care în limbile de comunicare umane frazele conțin propoziții. Aceasta ne permite să construim calcule oricât de complexe.

{{index statement, semicolon, program}}

Dacă o expresie corespunde unui fragment dintr-o propoziție, o _instrucțiune_ JavaScript corespunde unei propoziții complete. Un program este o listă de instrucțiuni.

{{index [syntax, statement]}}

Cea mai simplă instrucțiune este o expresie urmată de `;`. Acesta este un program:

```
1;
!false;
```

Programul de mai sus este inutil. O expresie poate fi responsabilă de producerea unei valori, care apoi urmează să fie folosită de către codul care o include. O instrucțiune este o construcție de sine stătătoare și contează doar în măsura în care produce un efect. Aceasta ar putea afișa o informație pe ecran sau ar putea modifica starea internă a mașinii într-un mod care să influențeze execuția instrucțiunilor care îi urmează. Aceste modificări sunt denumite _efecte secundare (side effects)_. Instrucțiunile din programul de mai sus doar produc valorile `1` și `true` pe care nu le folosesc mai departe pentru nimic altceva, ceea ce nu produce nici un fel de modificări. Atunci când rulați acest program, nu se întâmplă nimic observabil.

{{index "programming style", "automatic semicolon insertion", semicolon}}

În unele situații, JavaScript vă permite să omiteți caracterul `;` la sfârșitul unei instrucțiuni. În alte cazuri, este obligatoriu să îl utilizați pentru a evita ca următoarea instrucțiune să fie tratată ca și cum ar face parte din instrucțiunea curentă. Regulile cu privire la omiterea caracterului `;` sunt relativ complexe și favorizează erorile. Astfel încât, în această carte, la sfârșitul fiecărei instrucțiuni vom utiliza `;`, cel puțin până veți învăța suficient de multe despre subtilitățile omiterii caracterului `;`.

## Bindings

{{indexsee variable, binding}}
{{index [syntax, statement], [binding, definition], "side effect", [memory, organization], [state, in binding]}}

Cum își păstrează un program starea internă? Cum își amintește lucruri? Am văzut cum putem produce valori noi pe baza unor valori existente dar astfel nu vom schimba valorile existente iar noile valori trebuie utilizate imediat, altfel vor dispărea. Pentru a păstra valorile, JavaScript utilizează un concept numit _binding_ sau _variabilă_:

```
let caught = 5 * 5;
```

{{index "let keyword"}}

Acesta este un alt tip de instrucțiune. Cuvântul special `let` arată că această propoziție va defini o variabilă. Este urmat de numele variabilei și, dacă vrem să oferim imediat o valoare, vom folosi operatorul `=` și apoi o expresie.

Instrucțiunea anterioară crează o variabilă numită `caught` și o utilizează pentru a prelua și stoca numărul produs de înmulțirea `5*5`.

După ce o variabilă a fost definită, numele ei poate fi folosit ca și o expresie. Valoarea produsă de o astfel de expresie este tocmai valoarea stocată în variabilă. Iată un exemplu:

```
let ten = 10;
console.log(ten * ten);
// → 100
```

{{index "= operator", assignment, [binding, assignment]}}

Când o variabilă indică o anumită valoare, aceasta nu înseamnă că variabila este legată de acea valoare pentru totdeauna. Operatorul `=` poate fi utilizat pentru a deconecta o variabilă de la valoarea curentă și a o face să refere o altă valoare.

```
let mood = "light";
console.log(mood);
// → light
mood = "dark";
console.log(mood);
// → dark
```

{{index [binding, "model of"], "tentacle (analogy)"}}

V-ați putea imagina variabilele ca și niște tentacule, în loc de cutii. Ele nu conțin valori, ci le referă. Două variabile pot referi aceeași valoare. Un program poate referi doar acele valori pentru care încă mai are cel puțin o referință. Atunci când trebuie să memorați un lucru, vă creați un tentacul care să îl refere sau reatașați un tentacul existent, care referă o valoare de care nu mai aveți nevoie, către o noua valoare care prezintă interes.

Să analizăm un alt exemplu. Pentru a memora suma în dolari pe care Luigi v-o datorează, creați o variabilă. Apoi, când Luigi vă returnează $35, îi atribuiți acestei variabile o noua valoare.

```
let luigisDebt = 140;
luigisDebt = luigisDebt - 35;
console.log(luigisDebt);
// → 105
```

{{index undefined}}

Când definiți o variabilă fără să îi atribuiți o valoare, tentaculul nu referă nimic. Dacă cereți valoarea unei variabile "empty", veți obține valoarea `undefined`.

{{index "let keyword"}}

O singură instrucțiune `let` poate defini mai multe variabile. Definițiile trebuie separate prin virgulă:

```
let one = 1, two = 2;
console.log(one + two);
// → 3
```

Și cuvintele `var` și `const` pot fi folosite pentru crearea variabilelor, similar cu modul în care am folosit `let`.

```
var name = "Ayda";
const greeting = "Hello ";
console.log(greeting + name);
// → Hello Ayda
```

{{index "var keyword"}}

`var` (prescurtarea de la _variabilă_) este modul în care se defineau variabilele în versiunile JavaScript anterioare versiunii 2015. Voi reveni asupra diferenței față de `let` în [capitolul următor](functions). Pentru moment, rețineți că în cea mai mare parte se întâmplă același lucru, dar rareori vom folosi `var` în această carte pentru că are câteva proprietăți care crează confuzie.

{{index "const keyword", naming}}

Cuvântul `const` este prescurtarea de la _constantă_. El definește un binding constant, care se referă la aceeași valoare pe toată durata existenței sale. Acesta este util pentru crearea de binding-uri care dau un nume unei valori astfel încât aceasta poate fi ușor referită în continuare.

## Numele binding-urilor

{{index "underscore character", "dollar sign", [binding, naming]}}

Numele bindingurilor pot fi orice cuvinte. Cifrele pot face parte din nume - `catch22` este un nume valid - dar nu este permis ca numele să înceapă cu o cifră. Numele unui binding poate include simbolul `$` sau `_` dar nu și alte semne de punctuație sau caractere speciale.

{{index [syntax, identifier], "implements (reserved word)", "interface (reserved word)", "package (reserved word)", "private (reserved word)", "protected (reserved word)", "public (reserved word)", "static (reserved word)", "void operator", "yield (reserved word)", "enum (reserved word)", "reserved word", [binding, naming]}}

Cuvintele care au o semnificație specială, cum ar fi `let`, sunt _cuvinte-cheie_ și nu este permisă utilizarea lor pentru a denumi binding-urile. Există de asemenea o serie de cuvinte care sunt _rezervate_ pentru utilizarea lor în versiunile ulterioare ale JavaScript și care de asemenea nu pot fi folosite ca și nume pentru binding-uri. Lista completă a cuvintelor cheie și rezervate este destul de lungă.

```{lang: "text/plain"}
break case catch class const continue debugger default
delete do else enum export extends false finally for
function if implements import interface in instanceof let
new package private protected public return static super
switch this throw true try typeof var void while with yield
```

{{index [syntax, error]}}

Nu vă îngrijorați că ar trebui să memorați această listă. Atunci când crearea unui binding produce o eroare de sintaxă neașteptată, verificați dacă nu cumva utilizați un cuvânt rezervat pentru definire.

## Mediul

{{index "standard environment", [browser, environment]}}

Colecția de bindinguri precum și valorile lor care există la orice moment dat reprezintă _mediul_. Când un program își începe execuția, mediul nu este gol. El conține întotdeauna bindinguri care fac parte din standardul limbajului și, de cele mai multe ori, are și bindinguri care oferă modalități de interacțiune cu sistemul înconjurător. De exemplu, într-un browser, există funcții pentru interacțiunea cu websiteul încărcat momentan și pentru a citi intrarea de la mouse sau tastatură.

## Funcții

{{indexsee "application (of functions)", [function, application]}}
{{indexsee "invoking (of functions)", [function, application]}}
{{indexsee "calling (of functions)", [function, application]}}
{{index output, function, [function, application], [browser, environment]}}

Mare parte dintre valorile oferite în mediul implicit au tipul _function_. O funcție este o bucată de program împachetată într-o valoare. Asemenea valori pot fi _aplicate_ pentru a executa mini-programul împachetat în interiorul lor. De exemplu, în mediul unui browser, bindingul `prompt` reține o funcție care afișează o casetă de dialog în care user-ului i se cere să introducă o valoare. Se utilizează astfel:

```
prompt("Enter passcode");
```

{{figure {url: "img/prompt.png", alt: "A prompt dialog", width: "8cm"}}}

{{index parameter, [function, application], [parentheses, arguments]}}

Execuția unei funcții se numește _invocare (invoking)_, _apelare (calling)_ sau _aplicare (applying)_. Puteți apela o funcție prin plasarea parantezelor după o expresie care produce valoarea de tip funcție. De regulă, veți utiliza direct numele bindingului care memorează funcția. Valorile dintre paranteze sunt furnizate ca și parametri pentru programul din interiorul funcției. În exemplu, funcția `prompt` utilizează stringul pe care l-am dat ca și text explicativ care se afișează în caseta de dialog. Valorile furnizate funcțiilor se numesc _argumente_. Funcții diferite pot avea un număr diferit de argumente sau tipuri diferite ale argumentelor.

Funcția `prompt` nu este foarte utilizată în programarea web modernă, în principal pentru că nu avem control asupra aspectului casetei de dialog, dar poate fi utilă pentru programe de antrenament și experimente.

## Funcția `console.log`

{{index "JavaScript console", "developer tools", "Node.js", "console.log", output, [browser, environment]}}

În exemple, am utilizat funcția `console.log` pentru a afișa valori. Majoritatea sistemelor JavaScript (inclusiv browserele web moderne si NodeJS) pun la dispoziție o funcție `console.log` care afișează argumentele primite pe un device de ieșire de tip text. În browsere, afișarea se produce în consola JavaScript. Această parte a interfeței browserului este ascunsă în mod implicit, dar va fi afișată prin apăsarea tastei F12. Dacă nu se întâmplă nimic, căutați în meniu un item numit Developer Tools sau similar.

{{if interactive

Când rulați exemplele din această carte (sau propriul vostru cod) în paginile acestei cărți, ieșirea pentru `console.log` este prezentată la sfârșitul exemplului, nu în consola JavaScript.

```
let x = 30;
console.log("the value of x is", x);
// → the value of x is 30
```

if}}

{{index [object, property], [property, access]}}

Deși numele bindingurilor nu por conține caracterul `.`, `console.log` îl conține. Aceasta, deoarece `console.log` nu este doar un simplu binding. Este de fapt o expresie care returnează proprietatea `log` a bindingului `console`. Vom afla mai exact ce înseamnă asta în [Capitolul ?](data#properties).

{{id return_values}}
## Valori returnate

{{index [comparison, "of numbers"], "return value", "Math.max function", maximum}}

Afișarea unei casete de dialog pe ecran este un efect secundar. Multe funcții sunt utile ca urmare a efectului secundar pe care îl produc. Funcțiile pot produce și valori, caz în care nu trebuie să producă un efect secundar pentru a fi utile. De exemplu, funcția `Math.max` primește oricâte argumente și returnează pe cel mai mare dintre ele.

```
console.log(Math.max(2, 4));
// → 4
```

{{index [function, application], minimum, "Math.min function"}}

Când o funcție produce o valoare, spunem că _returnează_ acea valoare. Orice produce o valoare este o expresie în JavaScript. Ceea ce înseamnă că apelurile funcțiilor pot fi utilizate în expresii mai complicate. Iată un exemplu în care `Math.min` este utilizat ca și parte dintr-o expresie:

```
console.log(Math.min(2, 4) + 100);
// → 102
```

[Capitolul următor](functions) vă explică modul în care puteți scrie propriile funcții.

## Controlul execuției

{{index "execution order", program, "control flow"}}

Când programul vostru conține mai mult de o instrucțiune, instrucțiunile se execută ca și cum ar fi o poveste, de sus în jos. Exemplul de mai jos conține două instrucțiuni. Prima solicită utilizatorului să introducă un număr, iar ceea de a doua, executată ulterior primeia, afișează pătratul numărului respectiv.

```
let theNumber = Number(prompt("Pick a number"));
console.log("Your number is the square root of " + theNumber * theNumber);
```

{{index [number, "conversion to"], "type coercion", "Number function", "String function", "Boolean function", [Boolean, "conversion to"]}}

Funcția `Number` convertește o valoare într-un număr. Avem nevoie de această conversie deoarece rezultatul funcției `prompt` este un string dar noi ridicăm la pătrat un număr. Există funcții similare `String` și `Boolean` care convertesc valorile la tipurile respective.

Iată o reprezentare schematică trivială a controlului liniar al execuției:

{{figure {url: "img/controlflow-straight.svg", alt: "Controlul fluxului de execuție", width: "4cm"}}}

## Execuția condițională

{{index Boolean, ["control flow", conditional]}}

Nu toate programele sunt liniare. Am putea, de exemplu, să creem o ramificație iar programul să continue execuția pe ramura corespunzătoare, în funcție de situația concretă. Aceasta este o _execuție condițională_.

{{figure {url: "img/controlflow-if.svg", alt: "Conditional control flow",width: "4cm"}}}

{{index [syntax, statement], "Number function", "if keyword"}}

Execuția condițională se definește cu ajutorul cuvântului-cheie `if`. În cea mai simplă variantă, dorim ca o anume bucată de cod să se execute dacă și numai dacă o anumită condiție este adevărată. De exemplu, am putea afișa pătratul valorii introduse doar dacă valoarea respectivă este un număr.

```{test: wrap}
let theNumber = Number(prompt("Pick a number"));
if (!Number.isNaN(theNumber)) {
  console.log("Your number is the square root of " + theNumber * theNumber);
}
```
Cu această modificare, daca introduceți "parrot", nu se va afișa nimic.

{{index [parentheses, statement]}}

Cuvântul cheie `if` execută sau nu o instrucțiune funcție de valoarea unei expresii booleene. Expresia pe baza căreia se face decizia este scrisă imediat după `if`, între paranteze. Apoi urmează instrucțiunea care se execută condiționat.

{{index "Number.isNaN function"}}

Funcția `Number.isNaN` este o funcție standard a JavaScript care returnează `true` dacă și numai dacă argumentul pe care îl primește este `NaN`. Funcția `Number` returnează `NaN` dacă primește ca argument un string care nu reprezintă un număr valid. Prin urmare, am putea citi în limbaj natural "cu excepția situației în care `theNumber` nu este un număr, execută instrucțiunea".

{{index grouping, "{} (block)", [braces, "block"]}}

Instrucțiunea de după `if` este plasată între acolade (`{` și `}`). Acoladele se pot utiliza pentru a grupa oricâte instrucțiuni într-o singură instrucțiune, numită _bloc_. În exemplul de mai sus le-am fi putut omite, dar pentru a evita analiza dacă sunt sau nu necesare acoladele, majoritatea programatorilor JavaScript le utilizează în situații de genul celei de mai sus. De regulă vom respecta această convenție în carte, cu excepția unor situații ocazionale:

```
if (1 + 1 == 2) console.log("It's true");
// → It's true
```

{{index "else keyword"}}

Adesea, nu veți scrie doar cod care se execută dacă o anumită condiție este adevărată dar și cod care tratează cazul celălalt (condiția este falsă). Acestă cale alternativă este reprezentată de către cealaltă săgeată din diagramă. Puteți utiliza cuvântul cheie `else` împreună cu `if` pentru a crea două căi alternative separate de execuție

```{test: wrap}
let theNumber = Number(prompt("Pick a number"));
if (!Number.isNaN(theNumber)) {
  console.log("Your number is the square root of " + theNumber * theNumber);
} else {
  console.log("Hey. Why didn't you give me a number?");
}
```

{{index ["if keyword", chaining]}}

Dacă aveți mai mult de două căi între care trebuie să alegeți, puteți înlănțui mai multe perechi `if/else`, ca în exemplul de mai jos:

```
let num = Number(prompt("Pick a number"));

if (num < 10) {
  console.log("Small");
} else if (num < 100) {
  console.log("Medium");
} else {
  console.log("Large");
}
```

Programul va verifica mai întâi daca `num` este mai mic decât 10. Dacă da, va afișa `"Small"` și execuția se încheie. Dacă nu, va selecta ramura `else`, care conține un al doilea `if`. Dacă cea de a doua condiție este adevărată (`< 100`), înseamnă că numărul are valoarea cel puțin 10 dar sub 100 și se va afișa `"Medium"`. Dacă nici a doua condiție nu este adevărată, cea de a doua ramura `else` va fi afișată.

Schema pentru acest program ar arăta cam așa:

{{figure {url: "img/controlflow-nested-if.svg", alt: "Înlănțuirea deciziilor în fluxul de execuție", width: "4cm"}}}

{{id loops}}
## Bucle `while` și `do` (repetiții)

Să considerăm un program care afișează toate numerele pare de la 0 la 12. O modalitate de a scrie acest program este:

```
console.log(0);
console.log(2);
console.log(4);
console.log(6);
console.log(8);
console.log(10);
console.log(12);
```

{{index ["control flow", loop]}}

Aceasta funcționează, dar ideea de a scrie un program este de a munci _mai puțin_. Dacă vrem să afișăm toate numerele pare mai mici decât 1000, acestă abordare nu este potrivită. Ceea ce ne trebuie este o modalitate de a repeta de mai multe ori execuția unei bucăți de cod. Această modalitate de control al execuției reprezintă o _buclă (repetiție)_.

{{figure {url: "img/controlflow-loop.svg", alt: "Bucla în fluxul de execuție", width: "4cm"}}}

{{index [syntax, statement], "counter variable"}}

Buclele ne permit să revenim la un punct din program unde am fost anterior și să repetăm blocul respectiv de instrucțiuni în noul context al programului. Dacă mai utilizăm și un binding pentru contorizare, putem proceda astfel:

```
let number = 0;
while (number <= 12) {
  console.log(number);
  number = number + 2;
}
// → 0
// → 2
//   … etcetera
```

{{index "while loop", Boolean, [parentheses, statement]}}

O instrucțiune care începe cu cuvântul cheie `while` crează o buclă. Cuvântul `while` este urmat de o expresie între paranteze și apoi o instrucțiune, similar cu `if`. Bucla execută instrucțiunea cât timp expresia produce o valoare care se convertește la valoarea booleană `true`.

{{index [state, in binding], [binding, as state]}}

Binding-ul `number` demonstrează modul în care putem urmări progresul unui program. La fiecare repetiție a buclei, valoarea referită de `number` crește cu 2. La începutul fiecărei repetiții, valoarea este comparată cu 12 pentru a determina dacă programul trebuie să se încheie sau nu.

{{index exponentiation}}

Ca și exemplu care calculează o valoare utilă, putem scrie un program care calculează valoarea expresiei 2^10^. Utilizăm două bindinguri: unul pentru a memora rezultatul și altul pentru a număra de câte ori am multiplicat rezultatul cu 2. Bucla testează dacă cel de-al doilea binding a ajuns la valoarea 10 sau nu. Dacă nu, actualizează ambele bindinguri.

```
let result = 1;
let counter = 0;
while (counter < 10) {
  result = result * 2;
  counter = counter + 1;
}
console.log(result);
// → 1024
```

Am fi putut inițializa contorul cu 1 și să testăm dacă este `<=10` dar, pentru motive care vor deveni evidente în capitolul [?](data#array_indexing), este o idee bună să ne familiarizăm cu indexarea de la 0.

{{index "loop body", "do loop", ["control flow", loop]}}

O buclă `do` este o structură de control similară cu o buclă `while`. Există o singură diferență: o buclă `do` va executa întotdeauna cel puțin odată instrucțiunea definită în corpul său și va testa dacă trebuie să își continue execuția dupa prima execuție. Pentru a reflecta acest aspect, testul se definește după corpul buclei.

```
let yourName;
do {
  yourName = prompt("Who are you?");
} while (!yourName);
console.log(yourName);
```

{{index [Boolean, "conversion to"], "! operator"}}

Acest program vă va obliga să introduceți un nume. Va continua să vă solicite acest lucru până când veti introduce o valoare diferită de stringul gol. Aplicarea operatorului `!` va converti valoarea introdusă la o valoare booleană înainte de a o nega. Orice string diferit de `""` va fi convertit la valoarea `true`. Aceasta înseamnă ca bucla se va executa până când veți introduce un nume diferit de stringul gol.

## Indentarea codului

{{index [code, "structure of"], [whitespace, indentation], "programming style"}}

În exemple, am adăugat spații în fața instrucțiunilor care fac parte din alte instrucțiuni. Aceste spații nu sunt obligatorii - computerul va accepta programul și fără ele. De fapt, chiar și scrierea programelor pe mai multe linii este opțională. Puteți scrie un program sub forma unei singure linii foarte lungi, dacă doriți.

Rolul indentării în interiorul unui bloc este de a evidenția structura codului. În codul în care noile blocuri sunt introduse în interiorul altor blocuri, poate fi dificil să vă dați seama unde se termină un bloc și începe un altul. Prin indentarea corespunzătoare, forma vizuală a programului reprezintă modul de subordonare a blocurilor din interiorul său. Prefer să utilizez două spații pentru indentare, dar gusturile diferă - unii programatori utilizează patru spații, alții preferă să utilizeze `tab`-uri. Ceea ce este cu adevărat important este să utilizați o indentare consistentă.

```
if (false != true) {
  console.log("That makes sense.");
  if (1 < 2) {
    console.log("No surprise there.");
  }
}
```

Majoritatea programelor pentru editarea codului vă asistă prin indentarea automată a codului în mod corespunzător.

## Bucle `for`

{{index [syntax, statement], "while loop", "counter variable"}}

Multe bucle urmează un șablon similar cu cel prezentat în exemplele pentru bucla `while`. Se crează mai întâi un binding cu rol de "contor" pentru a urmări progresul buclei `while`. Apoi urmează bucla `while`, de regulă cu o expresie de test care verifică dacă s-a atins sau nu valoarea finală a contorului. La sfârșitul corpului buclei se actualizează valoarea contorului pentru a urmări progresul.

{{index "for loop", loop}}

Deoarece acest șablon este atât de frecvent, JavaScript, ca și alte limbaje de programare, pune la dispoziție o instrucțiune mai compactă și mai inteligibilă, bucla `for`.

```
for (let number = 0; number <= 12; number = number + 2) {
  console.log(number);
}
// → 0
// → 2
//   … etcetera
```

{{index ["control flow", loop], state}}

Acest program este echivalent cu cel [anterior](program_structure#loops) care afișa numerele pare. Singura modificare este că toate instrucțiunile relativ la modificarea "stării" buclei sunt grupate după `for`.

{{index [binding, as state], [parentheses, statement]}}

Parantezele de după cuvântul cheie `for` trebuie să conțină două caractere `;`. Partea din fața primului caracter `;` _inițializează_ bucla, de regulă prin definirea unui binding. Între cele două caractere `;` se definește expresia care _verifică_ dacă bucla trebuie să continue. Iar după cel de-al doilea caracter `;` se precizează setul de expresii care _actualizează_ starea buclei după fiecare iterație. De cele mai multe ori, o construcție `for` este mai scurtă și mai clară decât o construcție `while`.

{{index exponentiation}}

În exemplul de mai jos calculăm 2^10^ utilizând o buclă `for` în locul unei bucle `while`:

```{test: wrap}
let result = 1;
for (let counter = 0; counter < 10; counter = counter + 1) {
  result = result * 2;
}
console.log(result);
// → 1024
```

## Ieșirea forțată dintr-o buclă

{{index [loop, "termination of"], "break keyword"}}

Evaluarea la valoarea `false` a condiției buclei nu este singura modalitate de a ieși dintr-o buclă. Avem la dispoziție o instrucțiune specială `break` care are ca efect ieșirea imediată din bucla în interiorul căreia este plasată.

Programul care urmează exemplifică utilizarea instrucțiunii `break`. Acest program determină primul număr mai mare decât 20 și divizibil cu 7:

```
for (let current = 20; ; current = current + 1) {
  if (current % 7 == 0) {
    console.log(current);
    break;
  }
}
// → 21
```

{{index "remainder operator", "% operator"}}

Operatorul pentru calculul restului împărțirii întregi (`%`) este o modalitate simplă de a determina dacă un număr este divizibil cu un alt număr. Dacă da, restul împărțirii întregi este zero.

{{index "for loop"}}

Construcția `for` din exemplul de mai sus nu are definită partea care verifică dacă bucla trebuie să se încheie. Aceasta înseamnă că bucla nu se va încheia decât în momentul în care se va executa instrucțiunea `break` din interiorul ei. 

Dacă eliminați instrucțiunea `break` sau scrieți accidental o condiție care se evaluează întotdeauna la valoarea `true`, programul va intra într-o _buclă infinită_ și nu se va încheia niciodată, ceea ce, de regulă, este de nedorit.

{{if interactive

Dacă reușiți să creați o buclă infinită într-unul din exemplele din această carte, de regulă veți fi întrebat după câteva secunde dacă doriți să încheiați execuția scriptului. Dacă nu, va trebui să inchideți tabul respectiv sau, în unele browsere, să închideți complet browserul pentru a recupera din situația de eroare.

if}}

{{index "continue keyword"}}

Cuvântul cheie `continue` este similar cuvântului cheie `break` în sensul că influențează progresul buclei. Când se va excuta instrucțiunea `continue` în interiorul buclei, se trece imediat la o nouă iterație a buclei.

## Actualizarea compactă a bindingurilor

{{index assignment, "+= operator", "-= operator", "/= operator", "*= operator", [state, in binding], "side effect"}}

Mai ales în cadrul buclelor, un program are nevoie să "actualizeze" un binding care memorează o valoare pe baza valorii anterioare a bindingului.

```{test: no}
counter = counter + 1;
```

JavaScript vă pune la dispoziție o variantă prescurtată:

```{test: no}
counter += 1;
```

Aveți la dispoziție prescurtări similare pentru mulți alți operatori, cum ar fi `result *= 2` pentru a dubla `result` sau `counter -= 1` pentru a contoriza prin scădere.

Această facilitate ne permită să mai scurtăm puțin exemplul nostru de contorizare:

```
for (let number = 0; number <= 12; number += 2) {
  console.log(number);
}
```

{{index "++ operator", "-- operator"}}

Pentru `counter += 1` și `counter -= 1` avem la dispoziție forme și mai compacte: `counter++` și `counter--`.

## Selectarea pe baza unei valori folosind `switch`

{{index [syntax, statement], "conditional execution", dispatch, ["if keyword", chaining]}}

Frecvent, avem cod în programele noastre care arată astfel:

```{test: no}
if (x == "value1") action1();
else if (x == "value2") action2();
else if (x == "value3") action3();
else defaultAction();
```

{{index "colon character", "switch keyword"}}

Există o construcție numită `switch` care are rolul de a exprima o asemenea "selecție" mai direct. Din păcate, sintaxa utilizată de JavaScript (care este moștenită din limbajele de programare descendente din C/Java) este oarecum incomodă - o succesiune de instrucțiuni `if` ar putea arăta mai bine. Iată un exemplu:

```
switch (prompt("What is the weather like?")) {
  case "rainy":
    console.log("Remember to bring an umbrella.");
    break;
  case "sunny":
    console.log("Dress lightly.");
  case "cloudy":
    console.log("Go outside.");
    break;
  default:
    console.log("Unknown weather type!");
    break;
}
```

{{index fallthrough, "break keyword", "case keyword", "default keyword"}}

Puteți plasa oricâte etichete `case` în interiorul blocului deschis prin `switch`. Programul va continua execuția cu eticheta care corespunde valorii evaluate în expresia de început a instrucțiunii `switch` sau cu cazul `default` dacă nu a fost găsită nici o potrivire. Execuția va continua până la întâlnirea unei instrucțiuni `break`. În unele cazuri, cum ar fi cazul `"sunny"` din exemplu, execuția continuă cu instrucțiunea asociată cazului următor (lipsește `break`). Dar trebuie să fiți atenți - dacă ați omis accidental instrucțiunea `break`, programul ar putea executa cod pe care nu ați dorit să îl execute.

## Utilizarea literelor mari și mici 

{{index capitalization, [binding, naming], [whitespace, syntax]}}

Numele bindingurilor nu pot conține spații însă este de multe ori util să folosim mai multe cuvinte pentru a descrie ce reprezintă bindingul respectiv. În mare, aveți următoarele opțiuni lizibile de a scrie numele unui binding cu mai multe cuvinte:

```{lang: null}
fuzzylittleturtle
fuzzy_little_turtle
FuzzyLittleTurtle
fuzzyLittleTurtle
```

{{index "camel case", "programming style", "underscore character"}}

Primul stil este greu de citit. Mai bine aș utiliza al doilea stil, însă acesta este mai greu de tastat. Funcțiile standard din JavaScript utilizează ultimul stil de scriere - fiecare cuvânt se scrie cu literă mare, cu excepția primului. Nu e greu să vă acomodați cu asemenea aspecte, iar codul scris cu multiple stiluri de denumire este deranjant la citire, astfel că vom utiliza și noi ultima convenție de mai sus.

{{index "Number function", constructor}}

În câteva cazuri, cum ar fi funcția `Number`, prima literă a numelui este literă mare. Aceasta marchează faptul că această funcție este un constructor (este o convenție a programatorilor JavaScript). O să clarificăm ce înseamnă un constructor în capitolul [?](object#constructor). Deocamdată, important este să nu vă lăsați deranjați de această aparentă lipsă de consistență.

## Comentarii

{{index readability}}

Adesea, codul în sine nu transmite toată informația pe care am dori să o transmită programul unui cititor uman sau o transmite într-un mod suficient de criptic pentru a o face greu de înțeles pentru unii cititori. Alteori, probabil că doriți să adăugați câteva idei ca și parte a programului. Pentru toate acestea, puteți folosi _comentarii_.

{{index "slash character", "line comment"}}

Un comentariu este o bucată de text care face parte dintr-un program dar este complet ignorată de către computer. JavaScript permite două moduri de a scrie comentarii. Pentru a scrie un comentariu pe o singură linie, veți putea folosi `//` și apoi să scrieți textul comentariului.

```{test: no}
let accountBalance = calculateBalance(account);
// It's a green hollow where a river sings
accountBalance.adjust();
// Madly catching white tatters in the grass.
let report = new Report();
// Where the sun on the proud mountain rings:
addToReport(accountBalance, report);
// It's a little valley, foaming like light in a glass.
```

{{index "block comment"}}

Un comentariu `//` se încheie la sfârșitul liniei. O secțiune de text între `/*` și `*/` va fi ignorată complet, chiar dacă este scrisă pe mai multe linii. Aceasta este utilă pentru a adăuga un comentariu de tip bloc despre un fișier sau o anumită parte a unui program.

```
/*
  I first found this number scrawled on the back of an old notebook.
  Since then, it has often dropped by, showing up in phone numbers
  and the serial numbers of products that I've bought. It obviously
  likes me, so I've decided to keep it.
*/
const myNumber = 11213;
```

## Rezumat

Acum știți că un program este alcătuit din instrucțiuni, care pot conține la rândul lor alte instrucțiuni. Instrucțiunile tind să conțină expresii, care la rândul lor pot conține alte expresii.

Scriind instrucțiunile una după alta obțineți un program care este executat de sus în jos. Puteți introduce perturbații în execuția programului prin utilizarea instrucțiunilor condiționale (`if`, `else` și `switch`) sau a celor de repetiție (`while`, `do` și `for`).

Bindingurile pot fi utilizate pentru a asocia nume unor date și sunt utile pentru a urmări starea programului. Mediul reprezintă setul de bindinguri care sunt definite. Sistemul JavaScript introduce întotdeauna un număr de bindinguri utile în mediul programului.

Funcțiile sunt valori speciale care încapsulează o bucată dintr-un program. Le puteți apela astfel: `functionName(argument1, argument2)`. Un asemenea apel de funcție este o expresie și poate produce o valoare.

## Exerciții

{{index exercises}}

Dacă nu sunteți siguri cum să vă testați soluțiile, referiți-vă la [Introducere](intro).

Fiecare exercițiu începe cu o descriere a problemei. Citiți această descriere cu atenție și încercați să rezolvați exercițiul. Dacă vă încurcați, citiți indiciile de ajutor de la sfârșitul exercițiului. Cartea nu conține soluțiile complete, dar le puteți găsi la adresa  [_https://eloquentjavascript.net/code_](https://eloquentjavascript.net/code#2). Dacă doriți cu adevărat să învățați din exerciții, vă recomand să nu citiți soluțiile înainte de a rezolva exercițiul sau măcar amânați până ce ați depus suficient efort pentru rezolvarea exercițiului.

### Construcția unui triunghi

{{index "triangle (exercise)"}}

Scrieți o buclă care efectuează apeluri utile ale `console.log` pentru a afișa un triunghi conform exemplului de mai jos:

```{lang: null}
#
##
###
####
#####
######
#######
```

{{index [string, length]}}

Ar putea fi util să știți că puteți determina lungimea unui string scriind `.length` după el.

```
let abc = "abc";
console.log(abc.length);
// → 3
```

{{if interactive

Majoritatea exercițiilor conțin o bucată de cod pe care o puteți edita pentru a rezolva exercițiul. Vă reamintesc că puteți executa click pe blocul de cod pentru a-l edita.

```
// Your code here.
```
if}}

{{hint

{{index "triangle (exercise)"}}

Puteți începe prin a scrie un program care să afișeze numerele de la 1 la 7, pe care îl puteți obține prin modificarea [exemplului cu afișarea numerelor pare](program_structure#loops) anterior, folosit ca exemplu când am introdus bucla `for`.

Acum analizați echivalența dintre numere și numărul de caractere `#`. Puteți trece de la 1 la 2 adăugând 1 (`+=1`). Puteți trece de la `#` la `##` adăugând un caracter (`+="#"`). Prin urmare, puteți folosi aceeași structură ca și în programul care afișează numerele.

hint}}

### FizzBuzz

{{index "FizzBuzz (exercise)", loop, "conditional execution"}}

Scrieți un program care utilizează `console.log` pentru a afișa toate numerele de la 1 la 100, cu două excepții. Pentru numerele divizibile cu 3, va afișa `"Fizz"` în locul numărului iar pentru cele divizibile cu 5 (dar nu și cu 3), va afișa `"Buzz"`.

După ce reușiti să rezolvați exercițiul, modificați codul astfel încât să se afișeze `"FizzBuzz"` pentru numerele care sunt divizibile și cu 3 și cu 5 (și va continua să afișeze `"Fizz"` sau `"Buzz"` pentru cele divizibile doar cu una dintre aceste valori).

(Aceasta este de fapt o întrebare din interviuri despre care se crede că filtrează un procent semnificativ de candidați. Dacă ați reușit să o rezolvați, valoarea voastră pe piața muncii tocmai a crescut.)

{{if interactive
```
// Your code here.
```
if}}

{{hint

{{index "FizzBuzz (exercise)", "remainder operator", "% operator"}}

Evident, parcurgerea intervalului de valori de la 1 la 100 presupune utilizarea unei bucle iar selectarea valorii care urmează să se afișeze este o execuție condiționată. Vă reamintesc că putem utiliza operatorul modulo (`%`) pentru a verifica dacă un număr este divizibil cu un alt număr (restul împărțirii este zero).

În prima versiune, există trei rezultate posibile pentru fiecare număr deci, va trebui să construiți o structură `if`/`else if`/`else`.

{{index "|| operator", ["if keyword", chaining]}}

Cea de a două versiune are o soluție evidentă și una mai inteligentă. Soluția simplă presupune adăugarea unei noi ramuri pentru a testa condiția suplimentară. Pentru soluția mai inteligentă, construiți un string ce conține cuvântul sau cuvintele ce trebuie afișate și apoi afișați fie cuvântul fie numărul dacă nu există cuvântul, prin utilizarea operatorului `||`.

hint}}

### Tabla de șah

{{index "chessboard (exercise)", loop, [nesting, "of loops"], "newline character"}}

Scrieți un program care crează un string ce reprezintă o tablă 8 x 8 utilizând caractere `\n` pentru a separa liniile. Pe fiecare poziție a gridului există fie un spațiu, fie un caracter `#`. Caracterele trebuie să alterneze pe linii consecutive, ca pe o tablă de șah.

Transmițând acest string metodei `console.log` ar trebui să afișați ceva similar cu:

```{lang: null}
 # # # #
# # # # 
 # # # #
# # # # 
 # # # #
# # # # 
 # # # #
# # # # 
```

După ce reușiți să generați tabla 8 x 8, creați un binding `size = 8` și modificați programul astfel încât să funcționeze pentru orice dimensiune a grilei, afișând o tablă de dimensiune `size`.

{{if interactive
```
// Your code here.
```
if}}

{{hint

{{index "chess board (exercise)"}}

Puteți construi stringul începând cu un string gol `""` și să adaugați pe rând caractere. Caracterul `"\n"` poate fi utilizat pentru a afișa o linie nouă.

{{index [nesting, "of loops"], [braces, "block"]}}

Pentru a itera în două dimensiuni, va trebui să utilizați o buclă în interiorul altei bucle. Plasați acolade în jurul corpurilor buclelor pentru a identifica ușor începutul și sfârșitul fiecărei bucle. Indentați corespunzător corpurile celor două bucle. Ordinea celor două bucle trebuie să urmărească ordinea în care construim stringul (linie cu linie, de la stânga la dreapta și de sus în jos). Prin urmare, bucla exterioară gestionează liniile iar bucla interioară gestionează caracterele de pe fiecare linie.

{{index "counter variable", "remainder operator", "% operator"}}

Pentru a urmări progresul, aveți nevoie de două bindinguri. Pentru a determina care dintre caractere trebuie să îl folosiți pe o poziție dată puteți testa paritatea sumei celor două contoare (`%2`).

După fiecare linie (adică după fiecare finalizare a buclei interioare), adăugați caracterul linie nouă (`"\n"`).

hint}}
