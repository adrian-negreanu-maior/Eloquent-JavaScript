{{meta {load_files: ["code/chapter/08_error.js"]}}}

# Buguri și erori

{{quote {author: "Brian Kernighan and P.J. Plauger", title: "The Elements of Programming Style", chapter: true}

Debugging-ul este de două ori mai dificil decât scrierea codului inițial. Prin urmare, dacă ați scris codul cât de inteligent ați putut, sunteți, prin definiție, insuficient de deștept pentru a-l repara.

quote}}

{{figure {url: "img/chapter_picture_8.jpg", alt: "Picture of a collection of bugs", chapter: framed}}}

{{index "Kernighan, Brian", "Plauger, P.J.", debugging, "error handling"}}

Defectele din programele pentru computer se numesc de obicei _bug-uri_. Programatorii se simt bine să și le imagineze ca și mici lucruri care reușesc să pătrundă în munca lor. Însă, în realitate, noi suntem cei care le plasăm în cod.

Dacă un program este suficient de cristalizat, putem categoriza bug-urile în cele care sunt cauzate de gândirea confuză și cele cauzate de erori atunci când gândirea a fost transformată în cod. În general, cele din prima categorie sunt mai greu de diagnosticat și fixat decât cele din a doua categorie.

## Limbajul

{{index parsing, analysis}}

Multe greșeli pot fi identificate automat de către computer, dacă înțelege suficient ceea ce încercăm să facem. Dar libertatea din JavaScript este un dezavantaj. Conceptul său de bindinguri și proprietăți este suficient de vag încât rareori va prinde erorile de tastare înainte de a rula programul. Și chiar și atunci, vă va permite să faceți lucruri care evident nu au sens fără să se plângă, cum ar fi să calculați `true * "monkey"`.

{{index [syntax, error], [property, access]}}

Există însă și lucruri despre care JavaScript se plânge. Scrierea unui program care nu respectă gramatica limbajului va determina computerul să reclame imediat. Alte lucruri, cum ar fi apelarea unui binding care nu este o funcție sau încercarea de a accesa o proprietate a unei valori `undefined`, vor determina raportarea unor erori atunci când programul încearcă să execute acea acțiune.

{{index NaN, error}}

Dar adesea, calculele noastre fără sens vor produce `NaN` (not a number) sau o valoare `undefined`, în timp ce programul își va continua execuția, convins că face ceva important. Greșeala se va manifesta doar mai târziu, după ce valoarea eronată a călătorit prin câteva funcții. Ar putea să nu declanșeze o eroare ci doar să cauzeze o ieșire eronată a programului. Identificarea sursei unor asemenea probleme poate fi dificilă.

Procesul de identificare a erorilor în programe se numește _debugging_.

## Modul strict

{{index "strict mode", [syntax, error], function}}

{{indexsee "use strict", "strict mode"}}

JavaScript poate funcționa puțin mai _strict_ prin activarea modului _strict_. Pentru aceasta, trebuie să plasăm stringul `"use strict"` la începutul programului sau la începutul corpului funcției. Iată un exemplu:

```{test: "error \"ReferenceError: counter is not defined\""}
function canYouSpotTheProblem() {
  "use strict";
  for (counter = 0; counter < 10; counter++) {
    console.log("Happy happy");
  }
}

canYouSpotTheProblem();
// → ReferenceError: counter is not defined
```

{{index "let keyword", [binding, global]}}

În mod normal, când uitați să folosiți `let` pentru a vă declara bindingul, cum se întâmplă cu `counter` în exemplul de mai sus, JavaScript crează silențios un binding global pentru aceasta. Însă în mod strict va fi raportată o eroare. Ceea ce este foarte util. Însă trebuie să menționăm că eroarea nu este detectată în cazul în care există deja un binding global cu acel nume. În acest caz, bucla va suprascrie valoarea acelui binding.

{{index "this binding", "global object", undefined, "strict mode"}}

O altă modificare în modul strict este aceea că valoarea bindingului `this` este nedefinită în funcțiile care nu sunt apelate ca metode. Un asemenea apel în afara modului strict, va avea bindingul `this` referitor la obiectul din domeniul de vizibilitate global, care este un obiect ale cărui proprietăți sunt tocmai bindingurile globale. Astfel, dacă apelați accidental incorect o metodă sau un constructor, în mod strict, JavaScript va produce o eroare imediat ce încearcă să acceseze `this`, în loc să se mulțumească să scrie în domeniul de vizibilitate global.

De exemplu, să considerăm următorul cod, care apelează funcția constructor fără cuvântul-cheie `new` astfel încât `this` nu se va referi la obiectul nou construit:

```
function Person(name) { this.name = name; }
let ferdinand = Person("Ferdinand"); // oops
console.log(name);
// → Ferdinand
```

{{index error}}

Deci, apelul eronat către `Person` a reușit, dar a returnat o valoare `undefined` și a creat bindingul global `name`. În mod strict, rezultatul este diferit:

```{test: "error \"TypeError: Cannot set property 'name' of undefined\""}
"use strict";
function Person(name) { this.name = name; }
let ferdinand = Person("Ferdinand"); // forgot new
// → TypeError: Cannot set property 'name' of undefined
```

Ni se spune imediat că ceva este greșit, ceea ce este util.

Din fericire, constructorii creați cu notația `class` vor reclama întotdeauna faptul că nu sunt apelați cu `new`, ceea ce reduce dimensiunea problemei chiar și în mod non-strict.

{{index parameter, [binding, naming], "with statement"}}

Modul strict mai realizează alte câteva lucruri. Nu permite să transmitem unei funcții mai mulți parametri cu aceleași nume și elimină anumite funcționalități problematice din limbaj (cum ar fi instrucțiunea `with`, atât de greșită încât nici măcar nu vom discuta despre ea în cele ce urmează).

{{index debugging}}

Pe scurt, plasarea `"use strict"` la începutul programului rareori dăunează, dar v-ar putea ajuta să identificați o problemă.

## Tipuri

Unele limbaje doresc să cunoască tipurile tuturor bindingurilor și expresiilor înainte de rularea programului. Ele vă vor spune imediat dacă un tip este utilizat în mod inconsistent. JavaScript ia în considerare tipurilor doar atunci când rulează programul și, chiar și atunci, adesea încearcă să convertească implicit valorile la tipurile pe care le așteaptă, așa că nu este de mare ajutor.

Totuși, tipurile oferă un cadru util pentru a discuta despre programe. Multe greșeli apar din confuzia relativ la tipul de valoare care intră sau iese dintr-o funcție. Dacă aveți această informație, confuzia este mai puțin probabilă.

Ați putea adăuga un comentariu ca și cel de mai jos înainte de funcția `goalOrientedRobot` din capitolul anterior, pentru a-i descrie tipul:

```
// (VillageState, Array) → {direction: string, memory: Array}
function goalOrientedRobot(state, memory) {
  // ...
}
```

Există mai multe convenții pentru adnotarea programelor JavaScript cu tipuri.

Un lucru important despre tipurile de date este că acestea trebuie să introducă propria lor complexitate pentru a putea descrie suficient codul și a fi utile. Care credeți că va fi tipul funcției `randomPick` care returnează la întâmplare un element al unui array? Trebuie să introducem o _variabilă pentru tip_, _T_, care va reprezenta orice tip, astfel încât `randomPick` să aibă un tip `([T]) → T` (o funcție care reduce un array de *T*-uri la un *T*).

{{index "type checking", TypeScript}}

{{id typing}}

Când tipurile unui program sunt cunoscute, computerul poate să ne ajute prin verificarea lor, identificarea erorilor înainte de execuția programului. Există mai multe dialecte ale JavaScript care adaugă tipuri la limbaj și le verifică. Cel mai popular este [TypeScript](https://www.typescriptlang.org/). Dacă sunteți interesați în a adăuga mai multă rigoare programelor voastre, vă recomand să îl încercați.

În această carte vom continua să utilizăm cod JavaScript brut, periculos, fără tipuri.

## Testarea

{{index "test suite", "run-time error", automation, testing}}

Dacă limbajul nu ne ajută prea mult în identificarea greșelilor, va trebui să le găsim pe calea dificilă: să rulăm programul și să vedem dacă face ceea ce trebuie.

Să facem asta manual, din nou și din nou, ar fi o idee în general proastă. Nu doar că este plictisitor, dar tinde să fie și ineficient deoarece necesită prea mult timp să testăm exhaustiv de fiecare dată când facem o modificare.

Computerele excelează în sarcinile repetitive iar testarea este sarcina repetitivă perfectă. Testarea automată este procesul de scriere a unui program care testează alt program. Scrierea testelor necesită puțin mai multă muncă decât testarea manuală dar, o dată ce au fost scrise, aveți o super putere: în doar câteva secunde veți putea verifica dacă programul vostru se comportă adecvat pentru toate situațiile pentru care ați scris teste. Când stricați ceva, veți observa imediat, în loc să vă aflați întâmplător mai târziu în situația de a observa eroarea.

{{index "toUpperCase method"}}

Testele, de regulă, sunt mici programe etichetate care verifică anumite aspecte ale codului. De exemplu, un set de teste pentru metoda `toUpperCase` (metodă implicită și probabil deja testată de altcineva) ar putea arăta astfel:

```
function test(label, body) {
  if (!body()) console.log(`Failed: ${label}`);
}

test("convert Latin text to uppercase", () => {
  return "hello".toUpperCase() == "HELLO";
});
test("convert Greek text to uppercase", () => {
  return "Χαίρετε".toUpperCase() == "ΧΑΊΡΕΤΕ";
});
test("don't convert case-less characters", () => {
  return "مرحبا".toUpperCase() == "مرحبا";
});
```

{{index "domain-specific language"}}

Scrierea acestui gen de teste tinde să producă un cod ciudat, repetitiv. Din fericire, există aplicații care ne ajută să construim și să rulăm colecții de teste (sau _suite de teste_) și care ne pun la dispoziție un limbaj (sub forma unor funcții și metode) adecvat pentru exprimarea testelor și care afișează rapoarte informative despre testele care eșuează. Acestea se numesc _executoare de teste_ (_test runners_).

{{index "persistent data structure"}}

Unele bucăți de cod sunt mai ușor de testat decât altele. În general, cu cât codul interacționează cu mai multe obiecte externe, cu atât este mai greu de setat contextul în care acesta să fie testat. Stilul de programare prezentat în [capitolul anterior](robot), care utilizează valori persistente în loc de obiecte ce se modifică, tinde să fie mai ușor de testat.

## Debugging

{{index debugging}}

Când observați că ceva este greșit în programul vostru deoarece se comportă eronat și produce erori, următorul pas este să încercați să vă dați seama _care_ este problema.

Uneori este evident. Mesajul de eroare indică o anumită linie din program și, dacă citiți descrierea erorii și acea linie de program, reușiți să vă dați seama adesea care este problema.

{{index "run-time error"}}

Dar nu întotdeauna. Uneori, linia care a declanșat problema este doar primul loc în care o valoare eronată, produsă în altă parte, a fost utilizată într-un mod invalid. Dacă ați încercat să rezolvați exercițiile din capitolele anterioare, probabil că deja ați ajuns în asemenea situații.

{{index "decimal number", "binary number"}}

Exemplul următor încearcă să convertească un număr întreg într-un string într-o bază dată (zecimal, binar etc) prin considerarea repetată a ultimei cifre și apoi împărțirea numărului pentru a elimina cifra prelucrată. Dar ieșirea ciudată pe care o produce sugerează că există un bug.

```
function numberToString(n, base = 10) {
  let result = "", sign = "";
  if (n < 0) {
    sign = "-";
    n = -n;
  }
  do {
    result = String(n % base) + result;
    n /= base;
  } while (n > 0);
  return sign + result;
}
console.log(numberToString(13, 10));
// → 1.5e-3231.3e-3221.3e-3211.3e-3201.3e-3191.3e-3181.3…
```

{{index analysis}}

Chiar dacă ați identificat deja problema, simulați că nu ați reușit încă. Știm că programul nostru funcționează greșit și vrem să identificăm de ce.

{{index "trial and error"}}

Acesta este momentul în care trebuie să rezistați tentației de a face modificări la întâmplare în codul vostru, cu speranța că problema va dispărea. În loc de asta, _gândiți_. Analizați ce se întâmplă și elaborați o teorie cu privire la motivul pentru care apare această eroare. Apoi continuați cu observații suplimentare pentru a vă testa teoria sau, dacă nu aveți încă o teorie, face observații suplimentare pentru a încerca să elaborați una.

{{index "console.log", output, debugging, logging}}

Plasarea strategică a câtorva apeluri `console.log` este o modalitate bună de a obține informații suplimentare despre ceea ce face programul. În cazul nostru, vrem ca `n` să ia pe rând valorile `13`, `1` și `0`. Haideți să inspectăm acest lucru.

```{lang: null}
13
1.3
0.13
0.013
…
1.5e-323
```

{{index rounding}}

_Corect_. Împărțirea lui 13 la 10 nu a produs un număr întreg. În loc de `n /= base`, ceea ce voiam de fapt era `n = Math.floor(n / base)` astfel încât eliminarea ultimei cifre să fie corectă.

{{index "JavaScript console", "debugger statement"}}

O alternativă la utilizarea `console.log` pentru a investiga comportamentul programului este să utilizați capabilitățile de _debugging_ ale browserului. Browserele au abilitatea de a seta un _breakpoint_ pe o anumită linie a codului vostru. Când execuția programului ajunge la o linie cu un asemenea breakpoint, programul ia o pauză și puteți inspecta valorile diferitelor bindinguri în acel moment. Nu vom intra în detalii deoarece debugger-ele diferă de la un browser la altul, dar căutați "Developer Tools" pentru browserul vostru sau căutați mai multe informații pe Web.

Un alt mod de a seta un breakpoint este de a include o instrucțiune `debugger` în program (pur și simplu utilizați acest cuvânt cheie). Dacă instrumentele de dezvoltare din browser sunt active, programul va lua o pauză de fiecare dată când ajunge la o asemenea instrucțiune.

## Propagarea erorilor

{{index input, output, "run-time error", error, validation}}

Programatorii nu pot preveni toate problemele, din păcate. Dacă programul vostru interacționează cu lumea exterioară în orice mod, se poate întâmpla să primească date de intrare formatate eronat, sau să se suprasolicite sau pur și simplu să apară o eroare a rețelei.

{{index "error recovery"}}

Dacă programați doar pentru voi, puteți ignora aceste erori până în momentul în care vor apărea. Dar dacă construiți un program ce va fi utilizat de către alții, de regulă veți dori ca programul să se comporte mai bine decât doar să "crape". Uneori, măsura corectă este ca programul să continue, ignorând datele de intrare eronate. În alte cazuri, mai bine raportați utilizatorului ce eroare ați întâlnit și doar apoi renunțați. Dar, în orice situație, programul trebuie să reacționeze activ ca și răspuns la problemă.

{{index "promptNumber function", validation}}

Să presupunem că aveți o funcție `promptNumber` care cere utilizatorului un număr și apoi îl returnează. Ce ar trebui să returneze dacă utilizatorul introduce "orange"?

{{index null, undefined, "return value", "special return value"}}

O opțiune ar fi să returnați o valoare specială. Alegeri frecvente sunt `null`, `undefined` sau -1.

```{test: no}
function promptNumber(question) {
  let result = Number(prompt(question));
  if (Number.isNaN(result)) return null;
  else return result;
}

console.log(promptNumber("How many trees do you see?"));
```

Acum, orice cod care apelează `promptNumber` ar trebui să verifice dacă a fost introdus un număr iar dacă nu, să recupereze cumva - fie prin solicitarea unui alt număr, fie prin setarea unei valori implicite. Sau ar putea să returneze o valoare specială către _apelantul_ său, prin care să informeze că nu a reușit să execute ceea ce s-a solicitat.

{{index "error handling"}}

În multe situații, mai ales atunci când erorile sunt obișnuite și apelantul trebuie să le ia explicit în considerare, returnarea unei valori speciale este o variantă bună pentru a semnala o eroare. Dar are și dezavantaje. Mai întâi, ce se întâmplă dacă funcția poate returna orice tip posibil de valori? Într-o asemenea funcție, ar trebui să împachetați răspunsul într-un obiect pentru a putea distinge între succes și eșec.

```
function lastElement(array) {
  if (array.length == 0) {
    return {failed: true};
  } else {
    return {element: array[array.length - 1]};
  }
}
```

{{index "special return value", readability}}

A doua problemă este că returnarea valorilor speciale poate conduce la cod ciudat. Dacă o bucată de cod apelează `promptNumber` de 10 ori, va trebui să verifice de 10 ori dacă s-a returnat `null`. Iar dacă răspunsul pentru valoarea `null` constă în returnarea acestei valori, apelanții acestei funcții va trebui să facă alte verificări asupra acestei valori, și așa mai departe.

## Excepții

{{index "error handling"}}

Când o funcție nu se poate executa normal, ceea ce ne-ar _place_ ar fi să își oprească execuția și să cedăm imediat controlul unei bucăți de cod care știe cum să gestioneze problema. Acesta este rolul _gestionării excepțiilor_.

{{index ["control flow", exceptions], "raising (exception)", "throw keyword", "call stack"}}

Excepțiile sunt un mecanism care permite codului care ajunge într-o situație de eroare să _ridice_ (sau să _arunce_) o excepție. O excepție poate fi orice valoare. Ridicarea unei excepții seamănă oarecum cu un return "pe steroizi" într-o funcție: se va ieși nu doar din contextul funcției ci și din contextele tuturor apelanților, până la codul inițial care a inițiat execuția curentă. Acest comportament se numește _desfacerea stivei_. Vă reamintiți probabil despre stiva de apel al funcțiilor menționată în [capitolul ?](functions#stack). O excepție micșorează stiva prin eliminarea tuturor contextelor de apel pe care le găsește.

{{index "error handling", [syntax, statement], "catch keyword"}}

Dacă excepțiile ajung întotdeauna până la partea de jos a stivei, atunci nu au prea mare utilitate. Vor fi doar o nouă modalitate de a închide programul. Puterea lor constă în faptul că putem plasa "obstacole" pe stivă, care vor _prinde_ excepția care se deplasează în jos peste aceasta. Imediat ce ați prins o excepție, puteți executa o acțiune pentru a adresa problema și apoi să continuați execuția programului.

Iată un exemplu:

{{id look}}
```
function promptDirection(question) {
  let result = prompt(question);
  if (result.toLowerCase() == "left") return "L";
  if (result.toLowerCase() == "right") return "R";
  throw new Error("Invalid direction: " + result);
}

function look() {
  if (promptDirection("Which way?") == "L") {
    return "a house";
  } else {
    return "two angry bears";
  }
}

try {
  console.log("You see", look());
} catch (error) {
  console.log("Something went wrong: " + error);
}
```

{{index "exception handling", block, "throw keyword", "try keyword", "catch keyword"}}

Cuvântul cheie `throw` este utilizat pentru ridicarea unei excepții. Prinderea uneia se realizează prin plasarea codului generator de excepții într-un bloc `try`, urmat de un bloc `catch`. Când blocul `try` va cauza ridicarea unei excepții, se va evalua blocul `catch` cu numele din paranteze legat de valoarea excepției. După ce s-a executat blocul `catch`, sau dacă execuția blocului `try` s-a finalizat fără probleme, programul continuă cu instrucțiunea de după instrucțiunea `try/catch`.

{{index debugging, "call stack", "Error type"}}

Am utilizat constructorul `Error` pentru a crea valoarea excepției. Acesta este un constructor JavaScript standard care crează un obiect cu proprietatea `message`. În majoritatea mediilor JavaScript, instanțele acestui constructor adună și informații despre stiva de apel atunci când excepția fost creată, așa-numita _stack trace_. Această informație este memorată în proprietatea _stack_ și poate fi utilă pentru identificarea și rezolvarea problemei: ne informează cu privire la funcția în care a apărut problema și ce funcții au creat apelul care a eșuat.

{{index "exception handling"}}

Observați că funcția `look` ignoră complet posibilitatea ca funcția `promptDirection` să funcționeze greșit. Acesta este marele avantaj al excepțiilor: gestionarea erorilor este necesară doar în punctul în care apare eroarea și în cel în care eroarea este prelucrată. Funcțiile apelate între aceste două puncte pot să o ignore complet.

Ei bine, aproape...

## Curățenie după excepții

{{index "exception handling", "cleaning up", ["control flow", exceptions]}}

Efectul unei excepții este un alt tip de fluxul al controlului. Fiecare acțiune care ar putea cauza o excepție, ceea ce înseamnă de fapt fiecare apel de funcție și fiecare accesare a unei proprietăți, poate conduce la transferul brusc al controlului în afara codului vostru.

Aceasta înseamnă că, de fapt, atunci când codul are mai multe efecte secundare, chiar dacă fluxul "obișnuit" al controlului pare că le va cauza pe toate, o excepție ar putea să prevină apariția unora dintre aceste efecte.

{{index "banking example"}}

Iată un cod foarte rău:

```{includeCode: true}
const accounts = {
  a: 100,
  b: 0,
  c: 20
};

function getAccount() {
  let accountName = prompt("Enter an account name");
  if (!accounts.hasOwnProperty(accountName)) {
    throw new Error(`No such account: ${accountName}`);
  }
  return accountName;
}

function transfer(from, amount) {
  if (accounts[from] < amount) return;
  accounts[from] -= amount;
  accounts[getAccount()] += amount;
}
```

Funcția `transfer` mută o sumă de bani dintr-un cont în altul, solicitând la un moment dat numele celuilalt cont. Dacă se introduce un nume invalid de cont, `getAccount` aruncă o excepție.

Dar `transfer` _mai întâi_ scoate banii din cont și apoi apelează `getAccount`, înainte de a-i adăuga în celălalt cont. Dacă este întreruptă de către o excepție, banii vor dispărea pur și simplu.

Acest cod ar fi putut fi scris puțin mai inteligent, de exemplu prin apelarea `getAccount` înainte de a începe să mute banii între conturi. Însă, adesea, problemele de acest gen apar în moduri mai subtile. Chiar și funcții care nu par a arunca vreo excepție, ar putea proceda astfel în situații excepționale sau atunci când programatorul comite o greșeală.

O variantă de a adresa această problemă este de utiliza mai puține efecte secundare. Din nou, un stil de programare care calculează valori noi în loc să le modifice pe cele existente ar putea fi de ajutor. Dacă o bucată de cod se oprește din execuție în mijlocul procesului de creare a noii valori, nimeni nu va observa valoarea incomplet generată și nu va fi nici o problemă.

{{index block, "try keyword", "finally keyword"}}

Dar nu este întotdeauna practic să procedăm astfel. Astfel că mai există o caracteristică pe care o au instrucțiunile `try`. Ele pot fi urmate de un bloc `finally`, fie în locul fie după un bloc `catch`. Blocul `finally` spune: "indiferent de rezultat, rulează acest cod după ce ai încercat să rulezi codul din blocul `try`".

```{includeCode: true}
function transfer(from, amount) {
  if (accounts[from] < amount) return;
  let progress = 0;
  try {
    accounts[from] -= amount;
    progress = 1;
    accounts[getAccount()] += amount;
    progress = 2;
  } finally {
    if (progress == 1) {
      accounts[from] += amount;
    }
  }
}
```

Această versiune a funcției, își urmărește progresul și, dacă la ieșire constată că a fost terminat blocul `try` într-un punct în care a creat o stare inconsistentă, repară ceea ce a deteriorat.

De menționat că blocul `finally` se va executa și atunci când se aruncă o excepție din blocul `try`, dar nu interferă cu excepția. După executarea blocului `finally`, excepția va continua să "desfacă" stiva.

{{index "exception safety"}}

Scrierea unor programe care operează fiabil, chiar și atunci când apar excepțiile în locuri neașteptate, este dificilă. Cei mai mulți programatori nici nu își bat capul și, deoarece excepțiile sunt de regulă rezervate pentru circumstanțe excepționale, problema ar putea apărea suficient de rar încât să nu fie observată vreodată. Dacă e bine sau rău, asta depinde de câte stricăciuni ar putea provoca software-ul atunci când eșuează.

## Tratarea selectivă a erorilor

{{index "uncaught exception", "exception handling", "JavaScript console", "developer tools", "call stack", error}}

Atunci când o excepție ajunge până la baza stivei fără a fi tratată, va fi gestionată de către mediul de execuție. Medii diferite reacționează în mod diferit. În browsere, de regulă se afișează o descriere a erorii în consola JavaScript. NodeJS, mediul JavaScript diferit de browsere, despre care vom discuta în [capitolul ?](node), este mai atent la coruperea datelor. Va renunța la întregul proces în cazul în care există o excepție negestionată.

{{index crash, "error handling"}}

Pentru greșelile de programare, să lăsăm eroarea să își urmeze cursul este de multe ori cel mai bun lucru pe care îl putem face. O excepție negestionată este un mod rezonabil de semnaliza un program defect, iar consola JavaScript din browserele moderne vă va furniza mai multe informații despre apelurile de funcții care existau în stivă la apariția problemei.

{{index "user interface"}}

Pentru problemele care este de așteptat să apară în utilizarea obișnuită, lipsa gestionării erorilor este o strategie dezastruoasă.

{{index [function, application], "exception handling", "Error type", [binding, undefined]}}

Utilizarea invalidă a limbajului, cum ar fi referirea unui binding inexistent, referirea unei proprietăți asupra `null` sau apelarea unei construcții care nu este o funcție, va rezulta de asemenea în ridicarea unei excepții. Asemenea excepții pot fi prinse la rândul lor.

{{index "catch keyword"}}

La intrarea în blocul `catch`, tot ceea ce știm este că _ceva_ din corpul blocului `try` a cauzat o excepție. Dar nu știm _ce_ a cauzat-o și nici _ce fel_ de excepție avem.

{{index "exception handling"}}

JavaScript nu oferă suport direct pentru tratarea selectivă a excepțiilor: ori le prindem pe toate ori pe nici una. Este tentant să presupunem că excepția pe care o primim este cea la care ne gândeam când am scris blocul `catch`.

{{index "promptDirection function"}}

Dar s-ar putea să nu fie adevărat. Poate că alte presupuneri pe care le-am făcut au fost violate sau poate că am introdus un bug care a cauzat excepția. Exemplul de mai jos _încearcă_ să apeleze `promptDirection` până când primește un răspuns valid:

```{test: no}
for (;;) {
  try {
    let dir = promtDirection("Where?"); // ← typo!
    console.log("You chose ", dir);
    break;
  } catch (e) {
    console.log("Not a valid direction. Try again.");
  }
}
```

{{index "infinite loop", "for loop", "catch keyword", debugging}}

Construcția `for(;;)` este o modalitate de a crea intenționat o buclă infinită. Ieșim din buclă doar atunci când se introduce o direcție validă. _Dar_ am scris greșit `promptDirection`, ceea ce va conduce la o eroare "undefined variable". Deoarece blocul `catch` ignoră complet valoarea pentru excepție (`e`), presupunând că știe care este eroarea, dar tratează greșit eroarea de binding ca fiind un indiciu pentru date de intrare greșite. Nu doar că este provocată o buclă infinită, dar este "îngropat" mesajul de eroare util despre bindingul scris greșit.

Ca și regulă generală, nu este o idee bună să prindem cu plasa toate erorile, decât în cazul în care scopul este de a le direcționa în altă parte - de exemplu, transmitem prin rețea unui alt sistem că programul nostru s-a întrerupt. Chiar și atunci, gândiți cu atenție despre modul în care s-ar putea să ascundeți informație.

{{index "exception handling"}}

Prin urmare, vrem să prindem un tip specific de excepție. Am putea face asta prin a verifica în blocul `catch` dacă excepția pe care am primit-o este cea care ne interesează și să o re-aruncăm dacă nu. Dar cum recunoaștem o excepție?

Am putea compara proprietatea sa `message` cu mesajul de eroare pe care îl așteptăm. Dar acesta este un mod problematic de a identifica excepțiile - am utiliza informație adresată consumului uman (mesajul de eroare) pentru a lua decizii programatice. Imediat ce mesajul va fi modificat (de exemplu, prin translatare), codul nu va mai funcționa.

{{index "Error type", "instanceof operator", "promptDirection function"}}

Ar fi o idee mult mai bună să definim un nou tip de eroare și să utilizăm `instanceof` pentru a realiza identificarea.

```{includeCode: true}
class InputError extends Error {}

function promptDirection(question) {
  let result = prompt(question);
  if (result.toLowerCase() == "left") return "L";
  if (result.toLowerCase() == "right") return "R";
  throw new InputError("Invalid direction: " + result);
}
```

{{index "throw keyword", inheritance}}

Noua clasă de eroare extinde `Error`. Nu definește propriul său constructor, ceea ce înseamnă că moștenește constructorul `Error` care așteaptă un mesaj de tip string ca și argument. De fapt, nu definim absolut nimic - clasa este goală. Obiectele de tip `InputError` se comportă ca și cele de tip `Error`, cu excepția faptului că au o clasă diferită, ceea ce ne permite să le identificăm.

{{index "exception handling"}}

Acum, în codul buclei putem trata excepțiile cu mai multă grijă.

```{test: no}
for (;;) {
  try {
    let dir = promptDirection("Where?");
    console.log("You chose ", dir);
    break;
  } catch (e) {
    if (e instanceof InputError) {
      console.log("Not a valid direction. Try again.");
    } else {
      throw e;
    }
  }
}
```

{{index debugging}}

Astfel vom prinde doar excepțiile de tip `InputError` și celelalte excepții vor trece mai departe. Dacă reintroduceți greșeala, eroarea de definire a bindingului va fi raportată corespunzător.

## Aserțiuni

{{index "assert function", assertion, debugging}}

_Aserțiunile_ sunt verificări într-un program cu privire la faptul că anumite lucruri sunt așa cum ar trebui să fie. Ele nu sunt utilizate pentru a gestiona situații care pot apărea în operarea normală, ci pentru a găsi eventualele greșeli ale programatorului.

Dacă, de exemplu, `firstElement` este descrisă ca o funcție care trebuie să nu fie niciodată apelată pentru array-uri goale, am putea să o scriem astfel:

```
function firstElement(array) {
  if (array.length == 0) {
    throw new Error("firstElement called with []");
  }
  return array[0];
}
```

{{index validation, "run-time error", crash, assumption}}

Acum, în loc să returnăm silențios `undefined` (ceea ce obținem atunci când accesăm o proprietate a unui array care nu există), acum programul va exploda cu mare zgomot dacă utilizați greșit funcția. Astfel, șansa ca o asemenea greșeală să treacă neobservată se reduce și este mai ușor de determinat sursa problemelor atunci cînd ele apar.

Nu vă recomand să scrieți aserțiuni pentru fiecare tip posibil de date de intrare greșite. Ar fi foarte mult de lucru și codul ar fi tare gălăgios. Mai bine vă păstrați energia pentru greșelile care e ușor să apară (sau pe care constatați că le faceți frecvent).

## Rezumat

Greșelile, ca și datele de intrare eronate, fac parte din viață. O parte importantă în programare constă în identificarea, diagosticarea și repararea erorilor. Problemele sunt mai ușor de observat dacă aveți o suită de testare automată sau adăugați aserțiuni în programele voastre.

Problemele cauzate de factori externi programului ar trebui să fie tratate cu gentilețe. Uneori, atunci când problema poate fi tratată local, valorile speciale de retur sunt o strategie bună. Altfel, ar fi de preferat să utilizați excepțiile.

Aruncarea unei excepții va determina desfășurarea stivei de apel până la întâlnirea unui bloc `try/catch` sau până la baza stivei. Valoarea excepției va fi dată blocului `catch` care o prinde și care ar trebui să verifice dacă are de a face cu tipul de excepție așteptat și apoi să execute ceva ca urmare a excepției. Pentru a ne ajuta să adresăm impredictibilitatea fluxului de execuție cauzată de excepții, putem utiliza blocurile `finally` care ne asigură că o bucată de cod se execută _întotdeauna_ la finalul blocului `try/catch`.

## Exerciții

### Retry

{{index "primitiveMultiply (exercise)", "exception handling", "throw keyword"}}

Să presupunem că avem o funcție `primitiveMultiply` care în 20% din cazuri înmulțește două numere iar în restul de 80% din cazuri aruncă o excepție `MultiplicatorUnitFailure`. Scrieți o funcție care împachetează această funcție ciudată și o apelează repetat până când un apel se termină cu succes, după care returnează rezultatul.

{{index "catch keyword"}}

Asigurați-vă că tratați doar excepțiile pe care ar trebui să le tratați.

{{if interactive

```{test: no}
class MultiplicatorUnitFailure extends Error {}

function primitiveMultiply(a, b) {
  if (Math.random() < 0.2) {
    return a * b;
  } else {
    throw new MultiplicatorUnitFailure("Klunk");
  }
}

function reliableMultiply(a, b) {
  // Your code here.
}

console.log(reliableMultiply(8, 8));
// → 64
```
if}}

{{hint

{{index "primitiveMultiply (exercise)", "try keyword", "catch keyword", "throw keyword"}}

Cu siguranță, apelul funcției `primitiveMultiply` trebuie să aibă loc în interiorul unui bloc `try`. Blocul `catch` corespunzător ar trebui să arunce mai departe toate excepțiile care nu sunt instanțe ale `MultiplicatorUnitFailure` iar atunci când detectează o excepție de tipul menționat să reîncerce apelul.

Pentru a reîncerca, puteți scrie fie o buclă care se oprește doar dacă apelul reușește, ca în [exemplul `look`](error#look) anterior - sau să utilizați recursivitatea și să sperați că nu veți obține o secvență de erori atât de lungă încât stiva să fie umplută (ceea ce este destul de improbabil în acest caz).

hint}}

### Cutia încuiată

{{index "locked box (exercise)"}}

Să considerăm următorul obiect (controversat):

```
const box = {
  locked: true,
  unlock() { this.locked = false; },
  lock() { this.locked = true;  },
  _content: [],
  get content() {
    if (this.locked) throw new Error("Locked!");
    return this._content;
  }
};
```

{{index "private property", "access control"}}

Este o cutie cu o încuietoare. Există un array în interiorul cutiei, dar nu putem avea acces la el decât după deblocarea cutiei. Accesul direct la proprietatea privată `_content` este interzis.

{{index "finally keyword", "exception handling"}}

Scrieți o funcție numită `withBoxUnlocked` care primește ca argument o valoare de tip funcție, deblochează cutia, rulează funcția și apoi închide din nou cutia înainte de returnare, indiferent dacă funcția argument a returnat normal sau a aruncat o excepție.

{{if interactive

```
const box = {
  locked: true,
  unlock() { this.locked = false; },
  lock() { this.locked = true;  },
  _content: [],
  get content() {
    if (this.locked) throw new Error("Locked!");
    return this._content;
  }
};

function withBoxUnlocked(body) {
  // Your code here.
}

withBoxUnlocked(function() {
  box.content.push("gold piece");
});

try {
  withBoxUnlocked(function() {
    throw new Error("Pirates on the horizon! Abort!");
  });
} catch (e) {
  console.log("Error raised: " + e);
}
console.log(box.locked);
// → true
```

if}}

Pentru puncte-bonus, asigurați-vă că dacă apelați `withBoxUnlocked` iar cutia era deja deblocată, aceasta rămâne deblocată.

{{hint

{{index "locked box (exercise)", "finally keyword", "try keyword"}}

În acest exercițiu trebuie să utilizăm un block `finally`. Funcția trebuie să deblocheze mai întâi cutia și apoi să apeleze funcția argument din interiorul unui bloc `try`. În blocul `finally` cutia va fi blocată din nou.

Pentru a ne asigura că nu blocăm cutia dacă ea nu era blocată inițial, trebuie să verificăm starea acesteia la început și să utilizăm deblocarea și blocarea doar atunci când cutia era inițial blocată.

hint}}
