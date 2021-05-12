# Funcții

{{quote {author: "Donald Knuth", chapter: true}

Oamenii cred că știința computerelor este arta geniilor dar realitatea demonstrează contrarul, doar mai mulți oameni făcând lucruri care se construiesc unele pe baza altora, ca un perete de mici pietre.

quote}}

{{index "Knuth, Donald"}}

{{figure {url: "img/chapter_picture_3.jpg", alt: "Picture of fern leaves with a fractal shape", chapter: framed}}}

{{index function, [code, "structure of"]}}

Funcțiile sunt "sarea și piperul" programării JavaScript. Ideea de a împacheta o bucată de program într-o valoare are multe utilizări. Ne oferă o modalitate de a structura programe mai mari, de a reduce repetiția, de a asocia nume subprogramelor și de a izola subprogramele.

Cea mai evidentă aplicație a funcțiilor este pentru definirea unui nou vocabular. Crearea unor noi cuvinte în proză este de regulă un stil nerecomandat. Însă în programare, este indispensabilă.

{{index abstraction, vocabulary}}

Un vorbitor adult de limba engleză are un vocabular de aproximativ 20000 de cuvinte. Puține limbaje de programare au 20000 de comenzi predefinite. Vocabularul _disponibil_ tinde să fie definit mai precis și mai puțin flexibil decât cel din limbile vorbite de oameni. Prin urmare, de regulă avem nevoie să introducem noi concepte pentru a evita repetiția excesivă.

## Definirea unei funcții

{{index "square example", [function, definition], [binding, definition]}}

Definirea unei funcții este un binding obișnuit în care valoarea bindingului este o funcție. De exemplu, codul de ,ao jos definește `square` pentru a defini o funcție care produce pătratul unui număr dat:

```
const square = function(x) {
  return x * x;
};

console.log(square(12));
// → 144
```

{{indexsee "curly braces", braces}}
{{index [braces, "function body"], block, [syntax, function], "function keyword", [function, body], [function, "as value"], [parentheses, arguments]}}

O funcție este creată cu ajutorul unei expresii care începe cu cuvântul cheie `function`. Funcțiile au o listă de parametri (în cazul nostru, doar `x`) și un _corp_ care conține instrucțiunile ce trebuie executate atunci când se apelează funcția. Corpul funcție astfel create trebuie inclus întotdeauna între acolade, chiar dacă funcția execută o singură instrucțiune. 

{{index "power example"}}

O funcție poate avea mai mulți parametri sau nici unul. În exemplul de mai jos, funcția `makeNoise` nu enumeră nici un parametru iar funcția `power` enumeră doi parametri:

```
const makeNoise = function() {
  console.log("Pling!");
};

makeNoise();
// → Pling!

const power = function(base, exponent) {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
};

console.log(power(2, 10));
// → 1024
```

{{index "return value", "return keyword", undefined}}

Unele funcții produc o valoare, cum ar fi `power` sau `square` iar altele nu produc nici o valoare, cum ar fi `makeNoise` al cărei singur rezultat este un efect secundar. O instrucțiune `return` determină valoarea returnată de către o funcție. Atuncci când controlul ajunge la o asemenea isntrucțiune, execuția funcției se încheie imediat și valoarea este returnată codului care a apelat funcția. Utilizarea cuvântului cheie `return` fără a preciza expresia de evaluat va determina returnarea valorii `undefined`. Funcțiile care nu conțin deloc o instrucțiune `return`, cum este cazul funcției `makeNoise` vor returna tot valoarea `undefined`.

{{index parameter, [function, application], [binding, "from parameter"]}}

Parametrii unei funcții se comportă ca orice alt binding, dar valorile lor inițiale sunt furnizate de către codul _apelant_, nu de către codul funcției. 

## Bindinguri și domenii de vizibilitate (scopes)

{{indexsee "top-level scope", "global scope"}}
{{index "var keyword", "global scope", [binding, global], [binding, "scope of"]}}

Fiecare binding are un _domeniu de vizibilitate (scope)_ care este partea programului în care bindingul este vizibil (accesibil). Pentru bindingurile definite în afara oricărei funcții sau a oricărui bloc, domeniul de vizibilitate este întregul program - vă puteți referi la un asemenea binding din orice parte a programului. Aceste bindiguri au un domeniu de vizibilitate _global_. 

{{index "local scope", [binding, local]}}

Biindigurile crete pentru parametrii unei funcții sau cele declarate în interiorul unei funcții pot fi referite doar în interiorul acelei funcții. Din acest motiv, sunt denumite bindinguri _locale_. La fiecare apel al funcției, se crează noi instanțe ale acestor bindinguri. Astfel este posibilă o izolare a funcțiilor - fiecare apel al unei funcții acționează in micul univers al funcției (mediul său local) și de regulă poate fi înțeles fără a cunoaște multe detalii despre ceea ce se întâmplă în contextul global.

{{index "let keyword", "const keyword", "var keyword"}}

Bindigurile declarate cu `let` sau `const` sunt de fapt locale _blocului_ de cod în care au fost declarate. Prin urmare, dacă declarați un astfel de binding în interiorul unei bucle, codul ce precede sau cel ce urmează bucla nu va "vedea" bindingul. În versiunile anterioare JavaScript-2015, doar funcțiile creau noi domenii de vizibilitate astfel încât bindigurile create folosind `var` erau vizibile în tot contextul funcției în care erau definite sau în tot contextul global, dacă nu erau definite într-o funcție.

```
let x = 10;
if (true) {
  let y = 20;
  var z = 30;
  console.log(x + y + z);
  // → 60
}
// y is not visible here
console.log(x + z);
// → 40
```

{{index [binding, visibility]}}

Fiecare domeniu de vizibilitate poate să "vadă" contextul care îl include, astfel încât `x` este vizibil în interiorul blocului din exemplu. Excepția este cazul în care mai multe bindinguri au același nume (fiind definite în contexte diferite) - în acest caz, codul poate referi doar bindingul din contextul cel mai interior. De exemplu, atunci când codul din funcția `halve` se referă la `n` se va referi la propriul său binding creat de parametrul `n` și nu la `n` definit în contextul global.

```
const halve = function(n) {
  return n / 2;
};

let n = 10;
console.log(halve(100));
// → 50
console.log(n);
// → 10
```

{{id scoping}}

### Domenii de vizibilitate imbricate

{{index [nesting, "of functions"], [nesting, "of scope"], scope, "inner function", "lexical scoping"}}

JavaScript nu face diferența doar între bingurile _globale_ și cele _locale_. Blocurile și funcțiile se pot crea în interiorul altor blocuri sau funcții, producând mai multe nivele de _localizare_.

{{index "landscape example"}}

De exemplu, această funcție, care afișează ingredientele necesare pentru a găti o porție de humus - are o altă funcție în interiorul ei:

```
const hummus = function(factor) {
  const ingredient = function(amount, unit, name) {
    let ingredientAmount = amount * factor;
    if (ingredientAmount > 1) {
      unit += "s";
    }
    console.log(`${ingredientAmount} ${unit} ${name}`);
  };
  ingredient(1, "can", "chickpeas");
  ingredient(0.25, "cup", "tahini");
  ingredient(0.25, "cup", "lemon juice");
  ingredient(1, "clove", "garlic");
  ingredient(2, "tablespoon", "olive oil");
  ingredient(0.5, "teaspoon", "cumin");
};
```

{{index [function, scope], scope}}

Codul din interiorul funcției `ingredient` poate referi bindingul `factor` definit în funcția exterioară. Dar bindingurile sale locale, cum ar fi `unit` sau `ingredientAmount` nu sunt vizibile în funcția exterioară.

Mulțimea bindigurilor vizibile în interiorul unui bloc este determinată de locul în care este plasat blocul în cadrul programului. Fiecare domeniu de vizibilitate local poate observa toate domeniile de vizibilitate care îl conțin și orice domeniu de vizibilitate poate observa contextul global. Aceste reguli cu privire la vizibilitatea biningurilor definesc _sfera lexicală_.

## Funcțiile ca și valori

{{index [function, "as value"], [binding, definition]}}

Un binding de tip funcție este de regulă un nume pentru o bucată din program. Un asemenea binding este definit o singură dată și nu se modifică. Aceasta poate provoca o confuzie între funcție și numele ei.

{{index [binding, assignment]}}

Dar cele două noțiuni sunt diferite. O valoare de tip funcție poate fi utilizată la fel ca și orice altă valoare - o puteți utiliza în orice expresii, nu doar să o apelați. Puteți să memorați o valoare de tip funcție într-un nou binding, să o transmiteți ca și argument al unei funcții și altele. Similar, un binding care se refră la o funcție este doar un binding normal și, dacă nu este definit ca și constant, i se poate atribui o nouă valoare, ca și în exemplul de mai jos:

```{test: no}
let launchMissiles = function() {
  missileSystem.launch("now");
};
if (safeMode) {
  launchMissiles = function() {/* do nothing */};
}
```

{{index [function, "higher-order"]}}

În [Capitolul ?](higher_order), vom discuta despre lucrurile interesante pe care le puteți realiza prin transmiterea valorilor de tip funcșie ca și parametri ai altor funcții.

## Notația declarativă

{{index [syntax, function], "function keyword", "square example", [function, definition], [function, declaration]}}

Există și o modalitate mai scurtă puțin de a crea un binding de tip funcție. Când utilizăm cuvântul cheie `function` la începutul unei instrucțiuni, avem o funcționare diferită.

```{test: wrap}
function square(x) {
  return x * x;
}
```

{{index future, "execution order"}}

Aceasta este o _declarație_ a unei funcții. Instrucțiunea definește bindingul `square` care va referi funcția dată. Este puțin mai ușor de scris și nu este necesar să folosim `;` după acolada de final a funcției.

Există o subtilitate relativ la această modalitate de a defini o funcție.

```
console.log("The future says:", future());

function future() {
  return "You'll never have flying cars";
}
```

Codul de mai sus funcționează chiar dacă funcția este definită _după_ codul care o utilizează. Declarațiile funcțiilor nu fac parte din fluxul obișnuit al programului. Ele sunt mutate conceptual la începutul domeniului lor de vizibilitate și pot fi utilizate din orice parte a codului din domeniul de vizibilitate respectiv. Uneori această caracteristică este utilă deoarce oferă libertatea de a ordona codul într-un mod inteligibil fără a fi necesar să vă preocupe definirea tuturor funcțiilor înainte de a fi utilizate.

## Funcții Arrow

{{index function, "arrow function"}}

Există și o a treia notație pentru funcții, care arată complet diferit de celelalte. În locul cuvântului cheie `function` se utilizează o "săgeată (arrow)" (`=>`) (a nu se confunda cu operatorul mai mare sau egal scris `>=`).

```{test: wrap}
const power = (base, exponent) => {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
};
```

{{index [function, body]}}

Săgeata se plasează _după_ lista parametrilor și este urmată de corpul funcției. Această notație exprimă conceptul "această intrare (lista parametrilor) produce acest rezultat (corpul funcției)".

{{index [braces, "function body"], "square example", [parentheses, arguments]}}

Atunci când se utilizează un singur parametru, puteți omite parantezele în jurul listei de parametri. Dacă în corpul unei funcții există o singură expresie, nu un bloc plasat între acolade, acea expresie va fi returnată de către funcție. De exemplu, cele două definiții de mai jos ale funcției `square` produc același rezultat:

```
const square1 = (x) => { return x * x; };
const square2 = x => x * x;
```

{{index [parentheses, arguments]}}

Atunci când o funcție arrow nu are deloc parametri, lista sa de parametri este definită ca o pereche de paranteze fără a preciza nimic în interiorul parantezelor:

```
const horn = () => {
  console.log("Toot");
};
```

{{index verbosity}}

Nu există nici un motiv profund pentru a avea atât funcții arrow cât și expresii de tip funcție în limbaj. Cu excepția unui detaliu minor pe care îl vom discuta în capitolul [?](object), ele operează la fel. Funcțiile arrow au fost adăugate în 2015, în principal cu scopul de a scrie funcții scurte într=un mod mai puțin detaliat. Le vom utiliza pe larg în [capitolul ?](higher_order).

{{id stack}}

## Stiva de apel

{{indexsee stack, "call stack"}}
{{index "call stack", [function, application]}}

Modul în care se controlează execuția funcțiilor este oarecum elaborat. Haideți să facem câteva clarificări. Iată un program simplu care execută câteva apeluri de funcții:

```
function greet(who) {
  console.log("Hello " + who);
}
greet("Harry");
console.log("Bye");
```

{{index ["control flow", functions], "execution order", "console.log"}}

O execuție a acetui program funcționează în linii mari astfel: apelul la funcția `greet` va determina trecerea controlului la începutul acestei funcții (linia 2). Funcția apelează `console.log`, care preia controlul, își execută rolul și apoi returnează controlul în linia 2. Aici se ajunge la sfârșitul funcției deci se va reveni în locul de unde funcția a fost apelată, care este linia 4. Apoi se apelează din nou `console.log`. După ce se revine din acel apel, programul ajunge la sfârșit.

Am putea reprezenta schematic fluxul controlului astfel:

```{lang: null}
not in function
   in greet
        in console.log
   in greet
not in function
   in console.log
not in function
```

{{index "return keyword", [memory, call stack]}}

Deoarece din funcție se va sări înapoi în locul de unde a fost apelată, computerul trebuie să memoreze contextul din care a avut loc apelul. În unul dintre apeluri `console.log` a trebuit să returneze in funcția `greet` la sfârșitul execuției. În celalalt caz, a trebuit să returneze în contextul programului principal.

Locul în care computerul memorează acest context este _stiva de apel_. De fiecare dată când se apelează o funcție, contextul curent este memorat în vârful acestei stive. Când funcția returnează, contextul salvat este eliminat din stivă și utilizat pentru a continua execuția.

{{index "infinite loop", "stack overflow", recursion}}

Memorarea acestei stive necesită spațiu în memoria computerului. Atunci când stiva crește prea mult, computerul va eșua cu un mesaj de eroare în genul "out of stack space" sau "too much recursion". Codul din exemplul care urmează ilustrează acest comportament prin adresarea unei întrebări cu adevărat dificile care cauzează o infinitate de apeluri între două funcții. Ar fi într-adevăr o infinitate de apeluri dacă am avea în computer o cantitate infinită de memorie. În realitate însă, o să rămânem fără spațiu de memorie și "stiva va exploda".

```{test: no}
function chicken() {
  return egg();
}
function egg() {
  return chicken();
}
console.log(chicken() + " came first.");
// → ??
```

## Argumente opționale

{{index argument, [function, application]}}

Codul din exemplul de mai jos este permis și se execută fără nici o problemă:

```
function square(x) { return x * x; }
console.log(square(4, true, "hedgehog"));
// → 16
```

Am definit funncția `square` cu un singur parametru. Totuși am folosit trei parametri la apelul acestei funcții, fără ca interpretorul JavaScript să se plângă. Doar va ignora parametrii extra și va utiliza doar primul parametru transmis la apel.

{{index undefined}}

JavaScript este extrem de flexibil relativ la numărul de parametri pe care îi transmitem unei funcții. Dacă transmitem prea mulți parametri, parametrii extra sunt ignorați. Dacă transmitem prea puțini parametri, parametrii care lipsesc vor fi inițializați cu valoarea `undefined`.

Dezavantajul acestei abordări este că puteți ușor să transmiteți accidental un număr greșit de argumente spre funcții și nu veți primi nici un avertisment despre aceasta.

Avantajul este că veți putea să utilizați acest comportament pentru a permite apelul unei funcții cu un număr variabil de argument.De exemplu, funcția `minus` din exemplul de mai jos încearcă să imite operatorul `-` acționând asupra unuia sau a două argumente:

```
function minus(a, b) {
  if (b === undefined) return -a;
  else return a - b;
}

console.log(minus(10));
// → -10
console.log(minus(10, 5));
// → 5
```

{{id power}}
{{index "optional argument", "default value", parameter, ["= operator", "for default value"]}}

Dacă scrieți simbolul `=` după numele unui parametru și apoi precizați o expresie validă, valorea acelei expresii va înlocui parametrul în cazul în care acesta lipsește.

{{index "power example"}}

De exemplu, versiunea de mai jos a funcției `power` are al doilea argument opțional. Dacă nu transmiteți o valoare sau transmiteți valoarea `undefined`, va utiliza valoarea implicită 2 și funcția se va comporta ca și funcția `square` definită anterior.

```{test: wrap}
function power(base, exponent = 2) {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
}

console.log(power(4));
// → 16
console.log(power(2, 6));
// → 64
```

{{index "console.log"}}

În [capitolul următor](data#rest_parameters) vom vedea cum putem face ca în corpul funcției să accesăm întreaga listă a argumentelor care au fost transmise. Este util să putem face asta deoarece devine posibil să scriem funcții care acceptă oricâți parametri. De exemplu, `console.log` afișează toate valorile primite ca și parametri:

```
console.log("C", "O", 2);
// → C O 2
```

## Închideri (closures)

{{index "call stack", "local binding", [function, "as value"], scope}}

Abilitatea de a trata funcțiile ca și valori, combinată cu faptul că bindigurile locale sunt recreate de fiecare dată când se apelează o funcție, ridică o întrebare interesantă. Ce se întâmplă cu bindingurile locale când apelul funcției care le-a creat nu mai este activ?

Codul următor ilustrează această situație. Se definește o funcție `wrapValue` care crează un binding local. Apoi returnează o funcție care accesează și returnează bindingul local.

```
function wrapValue(n) {
  let local = n;
  return () => local;
}

let wrap1 = wrapValue(1);
let wrap2 = wrapValue(2);
console.log(wrap1());
// → 1
console.log(wrap2());
// → 2
```

Acest comportament este permis și funcționează așa cum speram - ambele instanțe ale bindingului pot fi accesate. Aceasta este o demonstrație bună a faptului că bindingurile locale sunt recreate pentru fiecare apel și că apelurile diferite nu își suprapun bindingurile locale.

Această facilitate - abilitatea de a referi o instanță specifică a unui binding local dintr-un domeniu de vizibilitate inclus se numește _închidere (closure)_. Acest comportament nu doar vă eliberează de îngrijorări cu privire la ciclul de viață al bindingurilor, dar vă și dă posibilitatea de a utiliza valori de tip funcție în moduri creative.

{{index "multiplier function"}}

Cu o modificare relativ simplă, putem schimba exemplul următor pentru a crea o funcție care multiplică cu orice factor arbitrar.

```
function multiplier(factor) {
  return number => number * factor;
}

let twice = multiplier(2);
console.log(twice(5));
// → 10
```

{{index [binding, "from parameter"]}}

Bindingul explicit cu numele `local` din exemplul cu funcția `wrapValue` nu este necesar, deoarece parametrul în sine este un binding local.

{{index [function, "model of"]}}

Aveți nevoie de puțină practică pentru a gândi astfel programele. Un model mental bun este să gândiți valorile de tip funcție ca având atât codul în corpul lor cât și mediul în care sunt create. Când sunt apelate, corpul funcției are acces la mediul în care a fost creat, nu la cel din care a fost apelat.

În exemplu, funcția `multiplier` este apelată și crează un mediu în care parametrul său `factor` este legat de valoarea 2. Valoarea de tip funcție pe care o returnează, care este memorată în `twice`, își amintește despre acest mediu astfel încât, atunci când este apelată, își multiplică argumentul cu 2.

## Recursivitatea

{{index "power example", "stack overflow", recursion, [function, application]}}

Este perfect în regulă pentru o funcție să se autoapeleze, cât timp nu o face de prea multe ori și cauzează umplerea stivei (stack overflow). O funcție care se autoapelează este o funncție _recursivă_. Recursivitatea permite scrierea anumitor funcții într-un stil diferit. Iată de exemplu o implementare alternativă a funcției `power`:

```{test: wrap}
function power(base, exponent) {
  if (exponent == 0) {
    return 1;
  } else {
    return base * power(base, exponent - 1);
  }
}

console.log(power(2, 3));
// → 8
```

{{index loop, readability, mathematics}}

Aceasta este mai aproape de modul în care matematicienii definesc exponențierea și descrie conceptul mai clar decât varianta care folosea o buclă. Funcția se autoapelează de mai multe ori cu un exponent din ce în ce mai redus, pentru a realiza înmulțirea repetată.

{{index [function, application], efficiency}}

Totuși, această implementare are o problemă: în implementările tipice ale JavaScript, este cam de trei ori mai lentă decât varianta ce folosește bucla. Execuția unei bucle este în general mai puțin costisitoare decât apelul unei funcții de mai multe ori.

{{index optimization}}

Dilema între vitezază și eleganță este interesantă. Există un continuum între "human-friendly" și "machine-friendly". Aproape orice program poate fi rapidizat prin mărirea dimensiunii lui și ridicarea complexității. Programatorul trebuie să decidă asupra unui echilibru adecvat.

În cazul funcției `power`, soluția mai puțin elegantă, bazată pe buclă este destul de simplă și ușor de citit. Nu face prea mult sens să o înlocuim cu versiunea recursivă. Însă adesea un program manevrează concepte atât de complexe încât diminuarea eficienței, cu scopul de a îmbunătății claritatea codului, este utilă.

{{index profiling}}

Preocuparea despre eficiență poate fi o distracție. Este un alt factor care complică conceperea programelor iar atunci când construim ceva care deja este dificil, acel motiv extra de preocupare poate fi paralizant.

{{index "premature optimization"}}

Prin urmare, începeți întotdeauna prin a scrie un cod corect și ușor de înțeles. Dacă vă preocupă că aveți un cod prea lent - ceea ce, de regulă, nu e cazul pentru că majoritatea codului nu se execută atât de des încât să consume un timp semnificativ - puteți măsura ulterior performanța și să o îmbunătățiți dacă e nevoie.

{{index "branching recursion"}}

Recursivitatea nu este întotdeauna doar o alternativă ineficientă la utilizarea buclelor. Unele probleme sunt cu mult mai ușor de rezolvat prin utilizarea recursivității. Cel mai adesea, acestea sunt probleme care necesită explorarea și procesarea mai multor "ramuri", fiecare dintre ele putând să se ramifice în și mai multe "ramuri".

{{id recursive_puzzle}}
{{index recursion, "number puzzle example"}}

Iată un exemplu: pornind de la numărul 1 și adăugând 5 sau multiplicând cu 3, putem genera o infinitate de numere. Cum veți scrie o funcție care, pentru un număr dat, încearcă să determine o secvență de asemenea adunări și inmulțiri încât să genereze acel număr?

De exemplu, numărul 13 poate fi obținut prin înmulțirea cu 3 și adunarea numărului 5 de două ori (1*3+5+5), iar numărul 15 nu poate fi generat.

Soluția recursivă este prezentată în exemplul de mai jos:

```
function findSolution(target) {
  function find(current, history) {
    if (current == target) {
      return history;
    } else if (current > target) {
      return null;
    } else {
      return find(current + 5, `(${history} + 5)`) ||
             find(current * 3, `(${history} * 3)`);
    }
  }
  return find(1, "1");
}

console.log(findSolution(24));
// → (((1 * 3) + 5) * 3)
```

Programul acesta nu determină neapărat _cea mai scurtă_ secvență de operații ci orice secvență care satisface enunțul problemei.

E în regulă dacă nu îl înțegeți cum funcționează. Haideți să îl parcurgem pentru că este un exercițiu foarte bun pentru gândirea recursivă.

Funcția interioară `find` execută de fapt recurența. Ea are două argumente: numărul curent și stringul care reține cum am ajuns la acel număr. Dacă găsește o soluție, returnează un string care arată cum ajungem la valoarea dată. Dacă nu găsește nici o soluție, va returna `null`.

{{index null, "|| operator", "short-circuit evaluation"}}

Pentru a realiza acest lucru, funcția execută una din trei acțiuni. Dacă numărul curent este numărul țintă, valoarea curentă a `history` este o modalitate de a obține acel număr și va fi returnată. Dacă numărul curent este mai mare decât valoarea-țintă, se va returna `null` (nu are sens să continuăm explorarea pentru ca putem obține doar valori și mai mari). În sfârșit, dacă suntem sub numărul-țintă, funcția va explora ambele căi posibile pornind de la numărul curent, adică se va apela de două ori, pentru multiplicarea cu 3 și adunarea cu 5. Dacă primul apel va returna o valoare nenulă, acesta va reprezenta soluția. În caz contrar, valoarea produsă de către al doilea apel va fi cea returnată.

{{index "call stack"}}

Pentru a înțelege și mai bine cum se execută funcția, haideți să analizăm apeluri la funcția `find` care se efectuează pentru găsirea unei soluții pentru numărul 13.

```{lang: null}
find(1, "1")
  find(6, "(1 + 5)")
    find(11, "((1 + 5) + 5)")
      find(16, "(((1 + 5) + 5) + 5)")
        too big
      find(33, "(((1 + 5) + 5) * 3)")
        too big
    find(18, "((1 + 5) * 3)")
      too big
  find(3, "(1 * 3)")
    find(8, "((1 * 3) + 5)")
      find(13, "(((1 * 3) + 5) + 5)")
        found!
```

Indentarea indică adâncimea stivei. Prima dată când este apelată funcția `find`, va începe prin autoapelare pentru a explora soluții care încep cu `(1 + 5)`. Apelul va continua pentru a explora _fiecare_ soluție care conduce la un număr mai mic sau egal cu numărul țintă. Deoarece nu va fi găsită o soluție, se va returna în final `null`. Operatorul `||` va determina continuarea explorării cu alternativa `(1 * 3)`. Această variantă va identifica la un moment dat o soluție.

## Crearea funcțiilor

{{index [function, definition]}}

Există două motive mai mult sau mai puțin naturale de a introduce funcții în programe.

{{index repetition}}

Primul este acela că scrieți cod similar de mai multe ori. Ați prefera să nu faceți asta. O cantitate mai mare de cod înseamnă mai multe șanse pentru greșeli și mai mult material pentru lectură pentru cineva care încearcă să înțeleagă programul. Astfel încât veți lua funcționalitatea pe care o repetați, îi veți da un nume adecvat și o veți plasa într-o funcție.

Al doilea motiv este că trebuie să implementați o anumită funcționalitate pe care nu ați implementat-o deja și care pare să merite propria funcție. Veți începe prin a denumi funcția și apoi veți scrie corpul ei. Puteți chiar să începeți să scrieți cod care utilizează funcția respectivă înainte de a o defini.

{{index [function, naming], [binding, naming]}}

Dificultatea de a găsi un nume bun pentru funcție este un indicator bun cu privire la cât de clar este conceptul pe care încercăm să îl implementăm. Haideți să mai analizăm un exemplu:

{{index "farm example"}}

Vrem să scriem un program care afișează două numere: numărul de vaci și găini dintr-o fermă, cu cuvintele `Cows` și `Chickens` după ele, si cu zerouri nesemnificative în față astfel încât numerele se vor afișa întotdeauna cu trei cifre.

```{lang: null}
007 Cows
011 Chickens
```

Avem nevoie de o funcție cu două argumente - numărul de vaci și numărul de găini.

```
function printFarmInventory(cows, chickens) {
  let cowString = String(cows);
  while (cowString.length < 3) {
    cowString = "0" + cowString;
  }
  console.log(`${cowString} Cows`);
  let chickenString = String(chickens);
  while (chickenString.length < 3) {
    chickenString = "0" + chickenString;
  }
  console.log(`${chickenString} Chickens`);
}
printFarmInventory(7, 11);
```

{{index ["length property", "for string"], "while loop"}}

Scriind `.length` după o expresie de tip string vom obține lungimea stringului respectiv. Prin urmare, bucla `while` va continua să adauge zerouri la începutul stringului ce reprezintă numărul până când acesta va avea lungimea de 3 caractere.

Misiune îndeplinită! Dar chiar când ne pregătim să trimitem fermierului codul (și o factură serioasă :) ), primim un apel în care suntem informați că ferma a început să crească și porci și au rugămintea să includem și această informație în afișaj.

{{index "copy-paste programming"}}

Desigur că putem. Dar chiar în timp ce ne pregăteam să copiem încă odată cele patru linii, ne oprim și reanalizăm. Trebuie să existe o variantă mai bună. Iată o primă încercare:

```
function printZeroPaddedWithLabel(number, label) {
  let numberString = String(number);
  while (numberString.length < 3) {
    numberString = "0" + numberString;
  }
  console.log(`${numberString} ${label}`);
}

function printFarmInventory(cows, chickens, pigs) {
  printZeroPaddedWithLabel(cows, "Cows");
  printZeroPaddedWithLabel(chickens, "Chickens");
  printZeroPaddedWithLabel(pigs, "Pigs");
}

printFarmInventory(7, 11, 3);
```

{{index [function, naming]}}

Funcționează! Dar acel nume, `printZeroPaddedWithLabel`, este cam ciudat. De fapt am cuplat trei operații într-o singură funcție: afișarea, adăugarea zerourilor și adăugarea unei etichete.

{{index "zeroPad function"}}

În loc să extragem toată partea care se repetă, haideți să extragem un singur _concept_.

```
function zeroPad(number, width) {
  let string = String(number);
  while (string.length < width) {
    string = "0" + string;
  }
  return string;
}

function printFarmInventory(cows, chickens, pigs) {
  console.log(`${zeroPad(cows, 3)} Cows`);
  console.log(`${zeroPad(chickens, 3)} Chickens`);
  console.log(`${zeroPad(pigs, 3)} Pigs`);
}

printFarmInventory(7, 16, 3);
```

{{index readability, "pure function"}}

O funcție cu un nume simplu și evident, cum ar fi `zeroPad` ușurează sarcina de a identifica rolul funcției pentru cineva care citește codul. Pe de altă parte, o asemenea funcție este utilă în mai multe situații decât acest program specific. Ați putea să o utilizați pentru a afișa tabele aliniate frumos.

{{index [interface, design]}}

Cât de inteligentă și de versatilă trebuie să fie funcția noastră? Putem scrie orice, de la o funcție extraordinar de simplă care poate doar să afișeze un număr cu trei caractere, până la un sistem foarte complicat de formatare a numerelor care gestionează numere fracționare, numere negative, alinierea punctului zecimal, utilizarea unor caractere diferite de `0` pentru aliniere și așa mai departe.

Un principiu util este de a nu adăuga "istețime" până nu sunteți absolut convinși că veți avea nevoie de ea. Poate fi tentant să scrieți cod general pentru fiecare bucată de funcționalitate pe care o implementați. Rezistați acestei tentații. Nu veți finaliza nimic real - doar veți scrie cod pe care nu îl veți utiliza niciodată.

{{id pure}}
## Funcțiile și efectele secundare

{{index "side effect", "pure function", [function, purity]}}

Funcțiile pot fi împărțite în principiu în două mari categorii: cele care sunt apelate pentru efectele lor secundare și cele care sunt apelate pentru valoarea returnată (deși este posibil ca o funcție să aibă atât efecte secundare cât și să returneze o valoare).

{{index reuse}}

Prima funcție helper din exemplul de mai sus cu ferma, `printZeroPaddedWithLabel`, este apelată pentru efectele sale secundare: afișează o linie. Cea de a doua versiune, `zeroPad`, este apelată pentru valoarea returnată. Nu este o întâmplare că cea de a doua funcție este utilă în mai multe situații decât prima. Funcțiile care crează valori sunt mai ușor de combinat în moduri noi decât cele care provoacă direct efecte secundare.

{{index substitution}}

O funcție _pură_ este un tip specific de funcție care produce valori, care nu doar că nu are efecte secundare dar nici nu se bazează pe efectele secundare din alt cod - de exemplu, nu se referă la bindingurile globale a căror valori s-ar putea modifica. O funcție pură are proprietatea plăcută de a produce întotdeauna aceeași valoare atunci când este apelată cu același set de valori pentru argumente, fără a executa nimic altceva. Un apel la o asemenea funcție poate fi înlocuit cu valoarea returnată fără a modifica semnificația codului. Atunci când nu sunteți siguri dacă o funcție pură funcționează corect, o puteți testa prin simpla apelare a ei cu garanția că, dacă funcționează în acel context, va funcționa în orice alt context. Funcțiile impure for necesita de regulă mai multe pregătiri pentru a fi testate.

{{index optimization, "console.log"}}

Totuși, nu trebuie să vă simțiși rău când scrieți funcții impure sau să declanșați un război sfânt pentru îndepărtarea lor din cod. Efectele secundare sunt adesea utile. Nu există nici o modalitate de a scrie o versiune pură a `console.log`, de exemplu, dar este util să aveți la dispoziție o asemenea funcție. Unele operații sunt mai ușor de exprimat eficient atunci când utilizăm efectele secundare, astfel încât viteza de calcul poate fi un motiv bun de a evita puritatea.

## Rezumat

În acest capitol ați învățat cum să scrieți propriile voastre funcții. Cuvântul cheie `function`, utilizat ca o expresie, poate crea o valoare de tip funcție. Atunci când este utilizat ca o instrucțiune, poate fi utilizat pentru a declara un binding și ai atribui ca valoare o funcție. Funcțiile arrow sunt un alt mod de a crea funcții.

```
// Define f to hold a function value
const f = function(a) {
  console.log(a + 2);
};

// Declare g to be a function
function g(a, b) {
  return a * b * 3.5;
}

// A less verbose function value
let h = a => a % 3;
```

Un aspect cheie pentru înțelegerea funcțiilor este înțelegerea domeniilor de vizibilitate. Fiecare bloc crează un nou domeniu de vizibilitate. Parametrii și bindingurile create într-un domeniu de vizibilitate sunt locale și nu sunt vizibile din exterior. Bindingurile declarate cu `var` au comportament diferit - ele devin vizibile în cel mai apropiat domeniu de vizibilitate al unei funcții sau în domeniul de vizibilitate global.

Separarea prelucrărilor efectuate de programul vostru în funcții este utilă. Nu va trebui să vă repetați,  iar funcțiile vă ajută să organizați programul prin gruparea codului în bucăți care execută sarcini specifice.

## Exerciții

### Minimum

{{index "Math object", "minimum (exercise)", "Math.min function", minimum}}

[Capitolul anterior](program_structure#return_values) a introdus funcția `Math.min` care returna cea mai mică dintre valorile primite ca argumente. Scrieți o funcție `min` care primește două argumente și returnează minimul dintre ele.

{{if interactive

```{test: no}
// Your code here.

console.log(min(0, 10));
// → 0
console.log(min(0, -10));
// → -10
```
if}}

{{hint

{{index "minimum (exercise)"}}

Dacă aveți probleme în plasarea corectă a parantezelor și a acoladelor pentru a obține o definiție validă a funcției, începeți prin a copia unul dintre exemplele anterioare și apoi modificați exemplul respectiv.

{{index "return keyword"}}

O funcție poate conține mai multe instrucțiuni `return`.

hint}}

### Recursivitate

{{index recursion, "isEven (exercise)", "even number"}}

Am văzut că `%` (operatorul pentru calculul restului împărțirii întregi) poat fi utilizat pentru a testa dacă un număr este par sau impar prin a utiliza `% 2` ca să verificăm dacă numărul este divizibil cu 2. Există însă un alt mod de a defini dacă un număr pozitiv este par sau impar:

- Zero este par.

- Unu este impar.

- Pentru orice alt număr _N_, paritatea sa este aceeași cu a numărului _N_ - 2.

Definiți o funcție recursivă `isEven` corespunzător acestei descrieri. Funcția trebuie să accepte un singur parametru (un număr natural) și să returneze o valoare booleană.

{{index "stack overflow"}}

Testați pentru 50 sau 75. Verificați comportamentul pentru -1. De ce? Puteți imagina un mod de a fixa problema?

{{if interactive

```{test: no}
// Your code here.

console.log(isEven(50));
// → true
console.log(isEven(75));
// → false
console.log(isEven(-1));
// → ??
```

if}}

{{hint

{{index "isEven (exercise)", ["if keyword", chaining], recursion}}

Funcția voastră va fi asemănătoare cu funcția interioară `find` în exemplul recursiv [`findSolution`](functions#recursive_puzzle) din acest capitol, cu o structură `if`/`else if`/`else` care testează care dintre cele trei cazuri trebuie aplicat. În `else` de la final, corespunzător celui de-al treilea caz, realizați apelul recursiv. Fiecare dintre cele trei cazuri trebuie să conțină o instrucțiune `return` sau să folosească un alt mod de a pregăti o valoare specifică pentru returnare.

{{index "stack overflow"}}

Atunci când i se transmite un parametru negativ, funcția va continua să se autoapeleze, cu valori negative din ce în ce mai mici și se va îndepărta de rezultat. La un moment dat, va epuiza tot spațiul din stiva de apel și va renunța.

hint}}

### Numărarea B-urilor

{{index "bean counting (exercise)", [string, indexing], "zero-based counting", ["length property", "for string"]}}

Puteți determina cel de-al N-lea caracter al unui string scriind `"string"[N]`. Valoarea returnată va fi un string format dintr-un singur caracter (de exemplu, `"b"`). Primul caracter se află pe poziția 0, ceea ce înseamnă că ultimul caracter se va afla pe poziția  `string.length - 1`. Cu alte cuvinte, un string cu două caractere are lungimea 2 iar caracterele sale se află pe pozițiile 0 și 1.

Scrieți o funcție `countBs` care primește un string ca și singurul său argument și returnează un număr care arată câte caractere `"B"` conține stringul respectiv.

Apoi scrieți o funcție `countChar` care are același comportament cu `countBs` cu excepția că primește un al doilea argument care precizează caracterul ale cărui apariții vor fi numărate. Apoi rescrieți `countBs` pentru a utiliza această nouă funcție.

{{if interactive

```{test: no}
// Your code here.

console.log(countBs("BBC"));
// → 2
console.log(countChar("kakkerlak", "k"));
// → 4
```

if}}

{{hint

{{index "bean counting (exercise)", ["length property", "for string"], "counter variable"}}

Funcția voastră va avea o buclă care va testa fiecare caracter al stringului. Va folosi un index care merge de la 0 la o valoare cu unu mai mică decât lungimea stringului (`< string.length`). Dacă în poziția curentă avem același caracter ca și cel pe care îl căutăm, adăugam 1 la o variabilă contor. După terminarea buclei, putem returna contorul.

{{index "local binding"}}

Aveți grijă să creați toate bindingurile din funcție _locale_ funcției, prin declararea corespunzătoare a lor cu ajutorul cuvintelor cheie `let` sau `const`.

hint}}
