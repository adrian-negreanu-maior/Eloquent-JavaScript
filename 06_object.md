{{meta {load_files: ["code/chapter/06_object.js"], zip: "node/html"}}}

# Viața secretă a obiectelor

{{quote {author: "Barbara Liskov", title: "Programming with Abstract Data Types", chapter: true}

Un tip de date abstract se construiește prin scrierea unui tip special de program […] care definește tipul sub forma operațiilor care se pot efectua asupra lui.

quote}}

{{index "Liskov, Barbara", "abstract data type"}}

{{figure {url: "img/chapter_picture_6.jpg", alt: "Imaginea unui iepure și a proto-iepurelui său", chapter: framed}}}

[Capitolul ?](data) introduce obiectele JavaScript. În cultura programării, avem o metodologie numită _programare orientată obiect_ care reprezintă un set de tehnici care utilizează obiecte (și concepte înrudite) ca și principiul central de organizare a programelor.

Deși nu există un consens cu privire la o definiție precisă, programarea orientată obiect a definit forma multor limbaje de programare, inclusiv JavaScript. Acest capitol va descrie modul în care aceste idei pot fi aplicate în JavaScript.

## Încapsularea

{{index encapsulation, isolation, modularity}}

Ideea de bază în programarea orientată obiect este de a divide programele în bucăți mai mici și a scrie fiecare bucată astfel încât să fie responsabilă de gestiunea stării sale.

Astfel, anumite cunoștințe despre modul în care funcționează o anumită parte din program pot fi păstrate la nivel _local_, în acea bucată de cod. Cineva care ar lucra în altă parte a programului nu va trebui să își amintească sau să conștientizeze aceste cunoștințe. Întotdeauna când aceste detalii locale vor fi actualizate, doar codul care le conține direct va trebui să fie actualizat.

{{id interface}}
{{index [interface, object]}}

Diferitele părți ale unui asemenea program interacționează între ele prin intermediul _interfețelor_, seturi limitate de funcții sau bindinguri care oferă funcționalitate utilă la un nivel mai abstract, ascuzând detaliile de implementare.

{{index "public properties", "private properties", "access control", [method, interface]}}

Asemenea părți de program se modelează folosind obiecte. Interfața lor constă într-un set specific de metode și proprietăți. Proprietățile care fac parte din interfață sunt numite _publice_. Celelalte, care nu trebuie să fie utilizate de codul din afara, sunt numite _private_.

{{index "underscore character"}}

Multe limbaje oferă un mod de a distinge între accesarea proprietăților publice și private și interzic codului exterior să le acceseze pe cele private. JavaScript, printr-o abordare minimalistă, nu o interzice - cel puțin nu încă. Există inițiative de a adăuga această facilitate limbajului.

Deși limbajul nu are această dinstincție implementată, programatorii JavaScript utilizează cu succes această idee. De regulă, interfața disponibilă este descrisă în  documentație sau în comentarii. De asemenea, o practică frecventă este plasarea caractererului `_` la începutul numelui unei proprietăți pentru a preciza că acea proprietate este privată.

Separarea interfeței de implementare este o idee extraordinară. De regulă, aceasta poartă numele de _încapsulare_.

{{id obj_methods}}

## Metode

{{index "rabbit example", method, [property, access]}}

Metodele nu sunt altceva decât proprietăți care rețin o valoare de tip funcție. Iată o metodă simplă:

```
let rabbit = {};
rabbit.speak = function(line) {
  console.log(`The rabbit says '${line}'`);
};

rabbit.speak("I'm alive.");
// → The rabbit says 'I'm alive.'
```

{{index "this binding", "method call"}}

De regulă o metodă trebuie să execute o operație cu obiectul asupra căruia a fost apelată. Atunci când o funcție este apelată ca și o metodă - folosind `object.method()` - bindingul numit `this` în corpul metodei indică automat obiectul asupra căruia a fost apelată.

```{includeCode: "top_lines:6", test: join}
function speak(line) {
  console.log(`The ${this.type} rabbit says '${line}'`);
}
let whiteRabbit = {type: "white", speak};
let hungryRabbit = {type: "hungry", speak};

whiteRabbit.speak("Oh my ears and whiskers, " +
                  "how late it's getting!");
// → The white rabbit says 'Oh my ears and whiskers, how
//   late it's getting!'
hungryRabbit.speak("I could use a carrot right now.");
// → The hungry rabbit says 'I could use a carrot right now.'
```

{{id call_method}}

{{index "call method"}}

Puteți interpreta `this` ca și un parametru suplimentar transmis într-un mod diferit. Dacă doriți să îl transmiteți explicit, puteți utiliza metoda `call` a unei funcții, care primește `this` ca și primul argument și tratează celelalte argumente ca și parametri normali.

```
speak.call(hungryRabbit, "Burp!");
// → The hungry rabbit says 'Burp!'
```

Deoarece fiecare funcție are propriul său binding `this`, a cărui valoare depinde de modul în care a fost apelată, nu vă puteți referi la `this` din domeniul de vizibilitate înconjurător într-o funcție definită cu ajutorul cuvântului cheie `function`.

{{index "this binding", "arrow function"}}

Funcțiile arrow sunt diferite - ele nu crează propriul binding `this` ci se referă la bindingul `this` din domeniul de vizibilitate care le înconjoară. Prin urmare, puteți scrie cod astfel, referindu-vă la `this` din interiorul unei funcții locale:

```
function normalize() {
  console.log(this.coords.map(n => n / this.length));
}
normalize.call({coords: [0, 2, 3], length: 5});
// → [0, 0.4, 0.6]
```

{{index "map method"}}

Dacă aș fi scris argumentul funcției `map` folosind cuvântul cheie `function`, codul de mai sus nu ar fi funcționat.

{{id prototypes}}

## Prototipuri

{{index "toString method"}}

Urmăriți atent:

```
let empty = {};
console.log(empty.toString);
// → function toString(){…}
console.log(empty.toString());
// → [object Object]
```

{{index magic}}

Am extras o proprietate dintr-un obiect gol. Magie!

{{index [property, inheritance], [object, property]}}

Bine, nu chiar. Doar că nu am dat încă detalii despre modul în care funcționează obiectele JavaScript. Pe lângă setul propriu de proprietăți, majoritatea obiectelor au și un _prototip_. Un prototip este un alt obiect care este folosit ca și sursă implicită de proprietăți. Dacă un obiect primește o cerere pentru o proprietate pe care nu o are, proprietatea respectivă va fi căutată în prototipul său, apoi în prototipul prototipului său, etc.

{{index "Object prototype"}}

Dar cine este prototipul unui obiect gol? Este marele strămoș, prototipul din spatele aproape tuturor obiectelor, `Object.prototype`.

```
console.log(Object.getPrototypeOf({}) ==
            Object.prototype);
// → true
console.log(Object.getPrototypeOf(Object.prototype));
// → null
```

{{index "getPrototypeOf function"}}

Cuum probabil ați ghicit, `Object.getPrototypeOf` returnează prototipul unui obiect.

{{index "toString method"}}

Relațiile dintre prototipurile obiectelor JavaScript formează o structură arborescentă iar rădăcina acestei structuri este `Object.prototype`. Acesta pune la dispoziție mai multe metode, cum ar fi `toString` care convertește un obiect în reprezentarea sa de tip string.

{{index inheritance, "Function prototype", "Array prototype", "Object prototype"}}

Multe obiecte nu au descendență directă din `Object.prototype` ca și prototip al lor ci sunt descendenți ai unui alt obiect care are un set diferit de proprietăți implicite. Orice funcție este derivată din `Function.prototype`, iar array-urile sunt derivate din `Array.prototype`.

```
console.log(Object.getPrototypeOf(Math.max) ==
            Function.prototype);
// → true
console.log(Object.getPrototypeOf([]) ==
            Array.prototype);
// → true
```

{{index "Object prototype"}}

Asemenea obiecte prototip au la rândul lor un prototip, adesea `Object.prototype`, astfel încât, în mod indirect, metodele cum ar fi `toString` sunt disponibile.

{{index "rabbit example", "Object.create function"}}

Puteți folosi `Object.create` pentru a construi un obiect cu un anumit prototip.

```
let protoRabbit = {
  speak(line) {
    console.log(`The ${this.type} rabbit says '${line}'`);
  }
};
let killerRabbit = Object.create(protoRabbit);
killerRabbit.type = "killer";
killerRabbit.speak("SKREEEE!");
// → The killer rabbit says 'SKREEEE!'
```

{{index "shared property"}}

O proprietate asemănătoare cu `speak(line)` într-o expresie de tip obiect este o modalitate mai scurtă de a defini o metodă. Ea crează o proprietate numită `speak` și îi atribuie ca și valoare o funcție.

Iepurele "proto" acționează ca și un container pentru proprietățile pe care le au toți iepurii. Un obiect individual de tip iepure, cum ar fi un iepure ucigaș, are toate proprietățile care se referă doar la el - tipul său - precum și pe cele derivate din prototipul său.

{{id classes}}

## Clase

{{index "object-oriented programming"}}

Sistemul de prototipuri din JavaScript poate fi interpretat ca un sistem informal de reprezentare a conceptului orientat obiect numit _clase_. O clasă definește forma unui anumit tip de obiect - ce metode și ce proprietăți are respectivul tip de obiecte. Un obiect particular va fi denumit _instanță_ a clasei respective.

{{index [property, inheritance]}}

Prototipurile sunt utile pentru a defini proprietăți pentru care toate instanțele unei clase partajează aceeași valoare, cum ar fi metodele. Proprietățile care diferă pentru fiecare instanță, cum ar fi proprietatea `type` a iepurelui, trebuie să fie memorate direct în obiect.

{{id constructors}}

Prin urmare, pentru a crea o instanță a unei clase date, trebuie să construiți un obiect care este derivat din prototipul corespunzător, dar trebui și să vă asigurați că are toate proprietățile pe care le au instanțele acestei clase. Pentru aceasta folosim funcții _constructor_.

```
function makeRabbit(type) {
  let rabbit = Object.create(protoRabbit);
  rabbit.type = type;
  return rabbit;
}
```

{{index "new operator", "this binding", "return keyword", [object, creation]}}

În JavaScript aveți posibilitatea să definiți mai ușor acest tip de funcții. Dacă plasați cuvântul `new` în fața unui apel de funcție, funcția este tratată ca un constructor. Aceasta înseamnă că se va crea automat un obiect cu prototipul adecvat, legat de `this` în funcție și returnat la sfârșitul funcției.

{{index "prototype property"}}

Obiectul prototip utilizat pentru construcția obiectelor este determinat prin consultarea proprietății `prototype` a funcției constructor.

{{index "rabbit example"}}

```
function Rabbit(type) {
  this.type = type;
}
Rabbit.prototype.speak = function(line) {
  console.log(`The ${this.type} rabbit says '${line}'`);
};

let weirdRabbit = new Rabbit("weird");
```

{{index constructor}}

Constructorii (de fapt toate funcțiile) automat au o proprietate numită `prototype`, care implicit stochează un obiect gol ce derivă din `Object.prototype`. O puteți suprascrie cu un alt obiect dacă doriți. Sau puteți adăuga proprietăți la obiectul existent, ca și în exemplu.

{{index capitalization}}

Prin convenție, numele constructorilor încep cu literă mare astfel încât să îi putem identifica rapid în multitudinea de funcții.

{{index "prototype property", "getPrototypeOf function"}}

Este important să înțelegeți distincția dintre modul în care un prototip se asociază cu un constructor (prin intermediul proprietății `prototype`) și felul în care obiectele au un prototip (care poate fi determinat cu ajutorul `Object.getPrototypeOf`). De fapt, prototipul unui constructor este `Function.prototype` deoarece constructorii sunt funcții. Proprietatea `prototype` reține prototipul utilizat pentru a crea instanțe cu ajutorul său.

```
console.log(Object.getPrototypeOf(Rabbit) ==
            Function.prototype);
// → true
console.log(Object.getPrototypeOf(weirdRabbit) ==
            Rabbit.prototype);
// → true
```

## Notația pentru clasă

Clasele JavaScript sunt funcții constructor ce au un prototip. Așa funcționează și așa trebuia să le scriem, până în 2015. Acum, avem la dispoziție o notație mai puțin ciudată.

```{includeCode: true}
class Rabbit {
  constructor(type) {
    this.type = type;
  }
  speak(line) {
    console.log(`The ${this.type} rabbit says '${line}'`);
  }
}

let killerRabbit = new Rabbit("killer");
let blackRabbit = new Rabbit("black");
```

{{index "rabbit example", [braces, class]}}

Cuvîntul `class` începe declarația unei clase, ceea ce ne permite să definim un constructor și un set de metode, toate într-un singur bloc. Putem scrie oricâte metode în blocul declarației. Cea numită `constructor` primește un tratament special. Ea este de fapt funcția constructor, care va fi legată de numele `Rabbit`. Toate celelalte sunt împachetate în prototipul acestui constructor. Prin urmare, declarația anterioară a unei clase este echivalentă cu definiția constructorului din secțiunea anterioară. Doar că arată mai bine.

{{index ["class declaration", properties]}}

Declarațiile claselor permit deocamdată doar adăugarea _metodelor_ - proprietăți ce memorează funcții - la prototip. Acesta ar putea fi oarecum un inconvenient când încercați să salvați o valoare care nu este funcție. Probabil versiunile următoare ale limbajului vor imbunătăți aceasta. Acum, puteți crea asemenea proprietăți prin manipularea directă a prototipului, după ce ați definit clasa.

Ca și `function`, `class` poate fi utilizat atât în instrucțiuni cât și în expresii. Când este utilizat într-o expresie, nu definește un binding ci doar produce constructorul clasei ca și valoare. Puteți să omiteți denumirea clasei într-o expresie de tip clasă.

```
let object = new class { getWord() { return "hello"; } };
console.log(object.getWord());
// → hello
```

## Suprascrierea proprietăților derivate

{{index "shared property", overriding, [property, inheritance]}}

Când adăugați o proprietate la un obiect, fie că este prezentă în prototip sau nu, proprietatea este adăugată obiectului în sine. Dacă exista deja o proprietate cu acest nume în prototipul său, acea proprietate nu va mai afecta obiectul deoarece acum va fi ascunsă de către proprietatea obiectului în sine.

```
Rabbit.prototype.teeth = "small";
console.log(killerRabbit.teeth);
// → small
killerRabbit.teeth = "long, sharp, and bloody";
console.log(killerRabbit.teeth);
// → long, sharp, and bloody
console.log(blackRabbit.teeth);
// → small
console.log(Rabbit.prototype.teeth);
// → small
```

{{index [prototype, diagram]}}

Următoarea diagramă schițează situația după execuția acestui cod. Prototipurile obiectelor `Rabbit` și `Object` sunt în spatele obiectului `killerRabbit`, ca un fel de fundal, iar proprietățile care nu sunt găsite în obiectul în sine vor fi căutate în ele.

{{figure {url: "img/rabbits.svg", alt: "Rabbit object prototype schema",width: "8cm"}}}

{{index "shared property"}}

Suprascrierea proprietăților care există deja într-un prototip poate fi utilă. Așa cum ne arată exemplul de mai sus, suprascrierea poate fi utilizată pentru a exprima proprietăți excepționale în instanțe ale unei clase mai generice, iar obiectele normale vor avea valorile standard ale proprietăților extrase din prototip.

{{index "toString method", "Array prototype", "Function prototype"}}

Am putea utiliza suprascrierea pentru a înlocui funcția `toString` standard pentru prototipurile pentru funcții și array-uri cu o metodă diferită de cea de bază.

```
console.log(Array.prototype.toString ==
            Object.prototype.toString);
// → false
console.log([1, 2].toString());
// → 1,2
```

{{index "toString method", "join method", "call method"}}

Apelul funcției `toString` pentru un array ne dă un rezultat similar cu apelul funcției `.join(",")` asupra sa - se afișează o listă de valori separate prin virgulă. Apelul direct al funcției `Object.prototype.toString` cu un parametru de tip array va produce un string diferit. Acea funcție nu știe ce este un array, astfel încât va afișa cuvântul _object_ și numele tipului variabilei între paranteze pătrate.

```
console.log(Object.prototype.toString.call([1, 2]));
// → [object Array]
```

## Map-uri

{{index "map method"}}

Am utilizat _map_ în [capitolul anterior](higher_order#map) pentru operația de transformare a unei structuri de date prin aplicarea unei funcții asupra elementelor sale. Cu toată confuzia, în programare se utilizează același cuvânt pentru a denumi un concept înrudit dar diferit.

{{index "map (data structure)", "ages example", ["data structure", map]}}

Un _map_ (substantiv) este o structură de date care asociază valori (chei) cu alte valori. De exemplu, am dori să mapăm numele cu vârstele. Pentru aceasta putem folosi obiecte.

```
let ages = {
  Boris: 39,
  Liang: 22,
  Júlia: 62
};

console.log(`Júlia is ${ages["Júlia"]}`);
// → Júlia is 62
console.log("Is Jack's age known?", "Jack" in ages);
// → Is Jack's age known? false
console.log("Is toString's age known?", "toString" in ages);
// → Is toString's age known? true
```

{{index "Object.prototype", "toString method"}}

În acest caz, proprietățile obiectelor sunt numite cu numele persoanelor iar valorile reprezintă vârstele acelor persoane. Evident, nimeni nu are numele `toString` în lista noastră. Totuși, deoarece obiectele astfel definite sunt derivate din `Object.prototype`, se pare că proprietatea este prezentă.

{{index "Object.create function", prototype}}

În consecință, utilizarea obiectelor simple ca și mapări este periculoasă. Avem mai multe variante de a evita problema. Mai întâi, putem crea obiecte care nu au un prototip. Dacă transmitem `null` către `Object.create`, obiectul rezultat nu va fi derivat din `Object.prototype` și poate fi folosit în siguranță ca și un map.

```
console.log("toString" in Object.create(null));
// → false
```

{{index [property, naming]}}

Numele proprietăților obiectelor trebuie să fie stringuri. Dacă aveți nevoie de un map ale cărui chei nu pot fi ușor convertite în stringuri - cum ar fi obiectele - nu puteți utiliza un obiect ca și un map.

{{index "Map class"}}

Din fericire, JavaScript definește o clasă `Map` exact în acest scop. Ea memorează o mapare și permite orice tip de valoare pentru chei.

```
let ages = new Map();
ages.set("Boris", 39);
ages.set("Liang", 22);
ages.set("Júlia", 62);

console.log(`Júlia is ${ages.get("Júlia")}`);
// → Júlia is 62
console.log("Is Jack's age known?", ages.has("Jack"));
// → Is Jack's age known? false
console.log(ages.has("toString"));
// → false
```

{{index [interface, object], "set method", "get method", "has method", encapsulation}}

Metodele `set`, `get` și `has` fac parte din interfața obiectului `Map`. Scrierea unei structuri care poate fi actualizată rapid și care permite căutarea într-un set mare de valori nu este ușoară. Dar nu trebuie nici să ne îngrijoreze. Altcineva a rezolvat deja problema și putem folosi această interfață simplă.

{{index "hasOwnProperty method", "in operator"}}

Dacă aveți un obiect simplu pe care trebuie să îl tratați ca pe un map, este util să știți că `Object.keys` returnează doar cheile _proprii_ ale obiectului, nu și pe cele ale prototipului. Ca o alternativă la operatorul `in` puteți folosi metoda `hasOwnProperty` care ignoră prototipul obiectului.

```
console.log({x: 1}.hasOwnProperty("x"));
// → true
console.log({x: 1}.hasOwnProperty("toString"));
// → false
```

## Polimorfismul

{{index "toString method", "String function", polymorphism, overriding, "object-oriented programming"}}

Când apelați funcția `String` asupra unui obiect (care convertește o valoare într-un string), aceasta va apela metoda `toString` a obiectului pentru a încerca să creeze un string care are semnificație. Unele prototipuri standard definesc propria versiune a metodei `toString` astfel încât pot crea un string ce conține o informație mai utilă decât `"[object Object]"`. Și noi putem face același lucru.

```{includeCode: "top_lines: 3"}
Rabbit.prototype.toString = function() {
  return `a ${this.type} rabbit`;
};

console.log(String(blackRabbit));
// → a black rabbit
```

{{index "object-oriented programming", [interface, object]}}

Acesta este un exemplu simplu al unei idei foarte puternice. Atunci când o bucată de cod este scrisă astfel încât poate funcționa pentru obiecte care au o anumită interfață - în acest caz metoda `toString` - orice obiect care suportă această interfață poate fi conectat la acel cod și codul va funcționa.

Această tehnică poartă denumirea de _polimorfism_. Codul polimorfic poate funcționa cu valori de tipuri diferite, cât timp aceste valori suportă interfața de care codul are nevoie.

{{index "for/of loop", "iterator interface"}}

Am menționat în [capitolul ?](data#for_of_loop) că o buclă `for`/`of` poate fi folosită pentru mai multe tipuri de structuri de date. Acesta este un alt exemplu de polimorfism - asemenea bucle se așteaptă ca structura de date să expună o anumită interfață, care este expusă de către array-uri și stringuri. Putem adăuga această interfață și obiectelor noastre! Dar înainte de a putea să facem acest lucru, trebuie să aflăm ce sunt simbolurile.

## Simbolurile

Este posibil ca interfețe diferite să utilizeze același nume al unei proprietăți pentru operații diferite. De exemplu, am putea defini o interfață în care metoda `toString` va converti un obiect în ceva nemaivăzut. Nu va fi posibil ca un obiect să se conformeze atât acelei interfețe cât și unei interfețe care utilizează metoda `toString` în mod standard.

Aceasta ar fi o idee proastă și problema aceasta nu este foarte frecventă. Majoritatea programatorilor JavaScript nu se gândesc la acest aspect. Dar arhitecții limbajului, care au sarcina de a se gândi la astfel de aspecte, ne-au dat o soluție.

{{index "Symbol function", [property, naming]}}

Când am spus că numele proprietăților sunt stringuri, nu am fost chiar exact. De regulă e adevărat, însă sunt și _simboluri_. Simbolurile sunt valori create cu funcția `Symbol`. Spre deosebire de stringuri, simbolurile nou create sunt unice - nu puteți crea de două ori același simbol.

```
let sym = Symbol("name");
console.log(sym == Symbol("name"));
// → false
Rabbit.prototype[sym] = 55;
console.log(blackRabbit[sym]);
// → 55
```

Stringul pe care îl transmiteți funcției `Symbol` este inclus atunci când îl convertiți într-un string și ușurează identificarea acestui simbol când, de exemplu, îl afișați în consolă. Dar nu are nici o semnificație suplimentară dincolo de faptul că mai multe simboluri nu pot avea aceleași nume.

Faptul că sunt unice și utilizabile ca și nume ale proprietăților face aceste simboluri potrivite pentru definirea interfețelor care pot trăi în pace lângă alte proprietăți, indiferent de numele acestora.

```{includeCode: "top_lines: 1"}
const toStringSymbol = Symbol("toString");
Array.prototype[toStringSymbol] = function() {
  return `${this.length} cm of blue yarn`;
};

console.log([1, 2].toString());
// → 1,2
console.log([1, 2][toStringSymbol]());
// → 2 cm of blue yarn
```

{{index [property, naming]}}

Putem include proprietăți definite cu simboluri în expresii de tip obiect și clase utilizând paranteze pătrate în jurul numelui proprietății. Prin aceasta numele proprietății va fi evaluat, similar notației cu paranteze pătrate pentru accesul proprietăților, care ne permite să ne referim la un binding ce referă acel simbol.

```
let stringObject = {
  [toStringSymbol]() { return "a jute rope"; }
};
console.log(stringObject[toStringSymbol]());
// → a jute rope
```

## Interfața iterator

{{index "iterable interface", "Symbol.iterator symbol", "for/of loop"}}

Ne așteptăm ca un obiect transmis unei bucle `for`/`of` să fie _iterabil_. Aceasta înseamnă că trebuie să aibă o metodă denumită cu simbolul `Symbol.iterator` (simbol definit de către limbaj și stocat ca o proprietate a funcției `Symbol`).

{{index "iterator interface", "next method"}}

Când este apelată, această metodă trebuie să returneze un obiect care expune o a doua interfață, _iterator_. Aceasta este cea care realizează iterarea propriu-zisă. Ea definește o metodă `next` care returnează următorul rezultat. Acel rezultat trebuie să fie un obiect care are proprietatea `value` ce returnează următoarea valoare, dacă există una și o proprietate `done` care trebuie să fie `true` dacă nu mai există alte rezultate și `false` în caz contrar.

De menționat că `next`, `value`, și `done` sunt nume de proprietăți care sunt stringuri, nu simboluri. Doar `Symbol.iterator`, care este probabil să fie adăugat la o mulțime de obiecte diferite, este de fapt un simbol.

Putem utiliza direct această interfață.

```
let okIterator = "OK"[Symbol.iterator]();
console.log(okIterator.next());
// → {value: "O", done: false}
console.log(okIterator.next());
// → {value: "K", done: false}
console.log(okIterator.next());
// → {value: undefined, done: true}
```

{{index "matrix example", "Matrix class", [array, "as matrix"]}}

{{id matrix}}

Să implementăm o structură de date iterabilă. Vom construi o clasă _matrix_, care va funcționa ca un array bidimensional.

```{includeCode: true}
class Matrix {
  constructor(width, height, element = (x, y) => undefined) {
    this.width = width;
    this.height = height;
    this.content = [];

    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        this.content[y * width + x] = element(x, y);
      }
    }
  }

  get(x, y) {
    return this.content[y * this.width + x];
  }
  set(x, y, value) {
    this.content[y * this.width + x] = value;
  }
}
```

Clasa își memorează conținutul într-un array cu _width_ × _height_ elemente. Elementele sunt memorate rând cu rând, de exemplu al treilea element de pe al cincilea rând (utilizând indexarea din zero) este memorat în poziția 4 × _width_ + 2.

Funcția constructor primeste `width`, `height` și o funcție opțională `element` utilizată pentru a determina valorile inițiale. Sunt definite metode `get` și `set` pentru a returna sau a actualiza elementele din matrice.

Când iterăm peste o matrice, de regulă ne interesează poziția fiecărui element, precum și elementul în sine, deci iteratorul nostru va trebui să producă elemente cu proprietățile `x`, `y` și `value`.

{{index "MatrixIterator class"}}

```{includeCode: true}
class MatrixIterator {
  constructor(matrix) {
    this.x = 0;
    this.y = 0;
    this.matrix = matrix;
  }

  next() {
    if (this.y == this.matrix.height) return {done: true};

    let value = {x: this.x,
                 y: this.y,
                 value: this.matrix.get(this.x, this.y)};
    this.x++;
    if (this.x == this.matrix.width) {
      this.x = 0;
      this.y++;
    }
    return {value, done: false};
  }
}
```

Clasa gestionează progresul iterării peste o matrice în proprietățile `x` și `y`. Metoda `next` începe prin a verifica dacă s-a ajuns la sfârșitul matricii. Dacă nu, _mai întâi_ crează un obiect ce reține valoarea curentă și _apoi_ îi actualizează poziția, mutându-se la următorul rând dacă este necesar.

Haideți să facem clasa `Matrix` iterabilă. Pe parcusul acestei cărți, ocazional, voi manipula prototipurile create pentru a adăuga metode claselor astfel încât bucățile de cod să rămână simple. Într-un program obișnuit, unde nu este necesară spargerea codului în bucăți mai mici, veți declara aceste metode direct în clasă.

```{includeCode: true}
Matrix.prototype[Symbol.iterator] = function() {
  return new MatrixIterator(this);
};
```

{{index "for/of loop"}}

Acum vom putea itera peste matrice cu `for`/`of`.

```
let matrix = new Matrix(2, 2, (x, y) => `value ${x},${y}`);
for (let {x, y, value} of matrix) {
  console.log(x, y, value);
}
// → 0 0 value 0,0
// → 1 0 value 1,0
// → 0 1 value 0,1
// → 1 1 value 1,1
```

## Getter-e, setter-e, și proprietăți statice

{{index [interface, object], [property, definition], "Map class"}}

Interfețele constau în principal din metode, dar puteți include și proprietăți care conțin valori care nu sunt de tip funcție. De exemplu, obiectele `Map` au o proprietate `size` care vă spune câte chei conține map-ul respectiv.

Nu este necesar pentru un asemenea obiect ca să calculeze și să memoreze o asemenea proprietate direct pe instanță. Chiar și proprietățile accesate direct ar putea ascunde un apel spre o metodă. Asemenea metode se numesc _getter-e_ și sunt definite folosind `get` în fața numelui metodei într-o expresie de tip obiect sau în declararea unei clase.

```{test: no}
let varyingSize = {
  get size() {
    return Math.floor(Math.random() * 100);
  }
};

console.log(varyingSize.size);
// → 73
console.log(varyingSize.size);
// → 49
```

{{index "temperature example"}}

Atunci când este consultată proprietatea `size` a obiectului, este de fapt apelată metoda asociată. Puteți proceda similar când vreți să scrieți într-o proprietate, utilizând un _setter_.

```{test: no, startCode: true}
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }
  get fahrenheit() {
    return this.celsius * 1.8 + 32;
  }
  set fahrenheit(value) {
    this.celsius = (value - 32) / 1.8;
  }

  static fromFahrenheit(value) {
    return new Temperature((value - 32) / 1.8);
  }
}

let temp = new Temperature(22);
console.log(temp.fahrenheit);
// → 71.6
temp.fahrenheit = 86;
console.log(temp.celsius);
// → 30
```

Clasa `Temperature` ne permite să citim temperatura fie în grade Celsius fie în grade Fahrenheit, dar stochează intern doar temperatura în grade Celsius și convertește automat în grade Fahrenheit în getter-ul și setter-ul `fahrenheit`.

{{index "static method"}}

Uneori, vreți să atașați proprietăți direct funcției constructor, nu prototipului clasei. Asemenea funcții nu vor avea acces la o instanță a clasei dar vor putea, de exemplu, să fie utilizate pentru a implementa moduri suplimentare de a crea instanțe.

În interiorul declarației unei clase, metodele care au `static` în fața numelui lor sunt memorate în constructor. Astfel încât clasa `Temperature` vă permite să scrieți `Temperature.fromFahrenheit(100)` pentru a crea un obiect pornind de la temperatura în grade Fahrenheit.

## Moștenirea

{{index inheritance, "matrix example", "object-oriented programming", "SymmetricMatrix class"}}

Unele matrice sunt _simetrice_. Dacă le oglindim față de diagonala stânga-sus:dreapta-jos, ele sunt nemodificate. Cu alte cuvinte, valoarea memorată în poziția _x_,_y_ este aceeași cu valoarea memorată în poziția _y_,_x_.

Imaginați-vă că avem nevoie de o structură de date similară cu `Matrix` dar care să impună faptul că matricea rămâne simetrică. Putem scrie aceasta structură de la zero, dar ar trebui să repetăm cod similar cu cel pe care l-am scris deja.

{{index overriding, prototype}}

Sistemul de prototipuri al JavaScript permite crearea unei _noi clase_, asemănătoare cu vechea clasă dar având definiții noi pentru o parte din proprietăți. Prototipul noii clase este derivat din vechiul prototip dar adaugă o nouă definiție pentru metoda `set`, de exemplu.

În terminologia programării orientate obiect, aceasta se numește _moștenire_. Noua clasă moștenește proprietățile și comportamentul de la vechea clasă.

```{includeCode: "top_lines: 17"}
class SymmetricMatrix extends Matrix {
  constructor(size, element = (x, y) => undefined) {
    super(size, size, (x, y) => {
      if (x < y) return element(y, x);
      else return element(x, y);
    });
  }

  set(x, y, value) {
    super.set(x, y, value);
    if (x != y) {
      super.set(y, x, value);
    }
  }
}

let matrix = new SymmetricMatrix(5, (x, y) => `${x},${y}`);
console.log(matrix.get(2, 3));
// → 3,2
```

Utilizarea cuvântului `extends` arată că această clasă nu trebuie să se bazeze direct pe prototipul implicit `Object` ci pe o altă clasă. Clasa pe care se bazează este denumită _superclasă_ iar cea nouă este denumită _subclasă_.

Pentru a inițializa o instanță `SymmetricMatrix`, constructorul apelează constructorul superclasei prin utilizarea cuvântului `super`. Aceasta este necesară deoarece noul obiect se va comporta în mare cam ca și un obiect `Matrix` și va avea nevoie de proprietățile pe care le au matricile. Pentru a ne asigura că matricea este simetrică, constructorul va împacheta funcția `element` pentru a interschimba coordonatele valorilor de sub diagonală.

Metoda `set` utilizează la rândul ei `super` dar nu pentru a apela constructorul ci pentru a apela o metodă specifică din mulțimea de metode a superclasei. Redefinim `set` însă vrem să folosim și comportamentul original. Deoarece `this.set` se referă la _noua_ metodă `set`, apelul acesta nu va funcționa. În interiorul metodelor unei clase putem apela metodele definite în superclasa acesteia folosind `super`.

Moștenirea ne permite să construim tipuri de date diferite pornind de la tipurile de date existente, fără un efort prea mare. Este o parte fundamentală a tradiției orientate-obiect, împreună cu încapsularea și polimorfismul. Dar în timp ce acestea sunt acceptate ca idei superbe, moștenirea este o idee mai controversată.

{{index complexity, reuse, "class hierarchy"}}

În timp ce încapsularea și polimorfismul pot fi utilizate pentru a _separa_ bucățile de cod, reducând complexitatea programului în ansamblu, moștenirea cuplează clasele împreună, ridicând complexitatea. Pentru a moșteni o clasă trebuie să cunoaștem mai multe despre cum funcționează decât atunci când doar o folosim. Moștenirea poate fi un instrument util pe care să îl folosim uneori în programele noastre dar nu trebuie să fie primul instrument pe care îl folosiți și probabil nu ar trebui să căutați activ oportunități pentru a construi ierarhii de clase (arbori de clase).


## Operatorul `instanceof`

{{index type, "instanceof operator", constructor, object}}

Uneori este util să știți dacă un obiect a fost derivat dintr-o anume clasă. Pentru aceasta, JavaScript vă pune la dispoziție operatorul binar `instanceof`.

```
console.log(
  new SymmetricMatrix(2) instanceof SymmetricMatrix);
// → true
console.log(new SymmetricMatrix(2) instanceof Matrix);
// → true
console.log(new Matrix(2, 2) instanceof SymmetricMatrix);
// → false
console.log([1] instanceof Array);
// → true
```

{{index inheritance}}

Operatorul va căuta și prin ierarhia de clase, astfel încât un obiect `SymmetricMatrix` este o instanță a clasei `Matrix`. Operatorul poate fi aplicat și asupra constructorilor standard cum ar fi `Array`. Aproape orice obiect este o instanță a   `Object`.

## Rezumat

Obiectele fac mult mai mult decât doar să stocheze propriile proprietăți. Ele au prototipuri, care sunt alte obiecte. Ele funcționează ca și cum ar avea proprietăți pe care nu le au, cât timp prototipurile lor au acele proprietăți. Obiectele simple au ca și prototip `Object.prototype`.

Constructorii, care sunt funcții al căror nume de regulă începe cu o literă mare, pot fi utilizați împreună cu operatorul `new` pentru a crea noi obiecte. Prototipul noului obiect va fi obiectul returnat de proprietatea `prototype` a constructorului. Puteți folosi aceasta în avantajul vostru prin plasarea tuturor proprietăților comune unui anume tip în prototipul lor. Există o notație pentru clase, care folosește `class` și oferă o modalitate mai clară de a defini un constructor și prototipul său.

Puteți defini getter-e și setter-e pentru a apela în secret metode de fiecare dată când se accesează o anume proprietate a unui obiect. Metodele statice sunt metode care se memorează în constructorul clasei și nu în prototipul acesteia.

Operatorul `instanceof` poate, dat fiind un obiect și un constructor, să determine dacă obiectul este sau nu o instanță a acelui constructor.

Un lucru util pe care îl putem face cu obiectele este să specificăm o interfață pentru ele și să informăm pe toată lumea că ar trebui să comunice cu obiectele noastre prin intermediul acelei interfețe. Restul detaliilor care constitui obiectul vor fi astfel _încapsulate_, ascunse în spatele interfeței.

Aceeași interfață poate fi implementată de către mai mult de un tip. Codul scris pentru a utiliza o interfață "știe" automat cum să lucrez cu orice număr de obiecte care oferă acea interfață. Acesta este _polimorfismul_.

Când implementăm mai multe clase care diferă între ele doar prin câteva detalii, ar putea fi util ca să scriem noi clase ca și _subclase_ ale unor clase existente, _moștenind_ parte din comportamentul acestora.

## Exerciții

{{id exercise_vector}}

### Un tip vector

{{index dimensions, "Vec class", coordinates, "vector (exercise)"}}

Scrieți o clasă `Vec` ce reprezintă un vector în două dimensiuni. Ea va primi parametrii `x` și `y` (numere), pe care îi va salva în proprietățile cu același nume.

{{index addition, subtraction}}

Adăugați prototipului `Vec` două metode, `plus` și `minus`, care primesc un alt vector ca și parametru și returnează suma, respectiv diferența celor doi vectori (`this` și parametrul primit).

Adăugați o proprietate getter `length` care va calcula lungimea vectorului - adică, distanța dintre punctul (_x_, _y_) și origine (0, 0).

{{if interactive

```{test: no}
// Your code here.

console.log(new Vec(1, 2).plus(new Vec(2, 3)));
// → Vec{x: 3, y: 5}
console.log(new Vec(1, 2).minus(new Vec(2, 3)));
// → Vec{x: -1, y: -1}
console.log(new Vec(3, 4).length);
// → 5
```
if}}

{{hint

{{index "vector (exercise)"}}

Uitați-vă la exemplul cu clasa `Rabbit` dacă nu sunteți siguri cum ar trebui să declarați o clasă.

{{index Pythagoras, "defineProperty function", "square root", "Math.sqrt function"}}

Adăugați un getter la constructor prin plasarea cuvântului `get` înainte de numele metodei. Pentru a calcula distanța de la (0, 0) la (x, y), puteți utiliza teorema lui Pitagora, conform căreia [√(x^2^ + y^2^)]{if html}[[$\sqrt{x^2 + y^2}$]{latex}]{if tex} este numărul pe care ar trebui să îl calculați iar `Math.sqrt` este funcția pe care o puteți folosi pentru calculul rădăcinii pătrate în JavaScript.

hint}}

### Grupuri

{{index "groups (exercise)", "Set class", "Group class", "set (data structure)"}}

{{id groups}}

Mediul standard JavaScript vă pune la dispoziție si o altă structură de date, numită `Set`. Asemănător unei instanțe `Map`, un set memorează o colecție de valori. Spre deosebire de `Map`, nu asociază aceste valori cu altele - doar urmărește ce valori fac parte din set. O anumită valoare face parte dintr-un set doar o dată - încercarea de a o adăuga încă o dată nu va avea nici un efect.

{{index "add method", "delete method", "has method"}}

Scrieți o clasă `Group` (deoarece `Set` este folosit deja). Adăugați ca și pentru `Set` metodele `add`, `delete` și `has`. Constructorul său va crea un grup gol, `add` va adăuga o (valoare numai dacă aceasta nu a fost deja adăugată), `delete` va elimina o valoare din grup (dacă aceasta face parte din grup), iar `has` returnează o valoare booleană ce precizează dacă argumentul său este membru al grupului.

{{index "=== operator", "indexOf method"}}

Utilizați operatorul `===` sau altceva echivalent cum ar fi `indexOf` pentru a determina dacă două valori sunt identice.

{{index "static method"}}

Adăugați clasei o metodă statică `from` care primește ca și argument un obiect iterabil și crează un grup care conține toate valorile produse prin iterarea asupra obiectului.

{{if interactive

```{test: no}
class Group {
  // Your code here.
}

let group = Group.from([10, 20]);
console.log(group.has(10));
// → true
console.log(group.has(30));
// → false
group.add(10);
group.delete(10);
console.log(group.has(10));
// → false
```

if}}

{{hint

{{index "groups (exercise)", "Group class", "indexOf method", "includes method"}}

Cel mai simplu mod de a reține elementele grupului este utilizarea unui array de membri ai grupului într-o proprietate a instanței. Metodele `includes` și `indexOf` pot fi utilizate pentru a verifica dacă o valoare dată aparține array-ului.

{{index "push method"}}

Constructorul clasei va inițializa colecția de membri la un array gol. Când se va apela `add`, va trebui să verificăm dacă valoarea există deja în array și dacă nu, să o adăugăm, folosind de exemplu `push`.

{{index "filter method"}}

Eliminarea unei element din array, în metoda `delete` nu este la fel de directă. Dar ați putea crea un array nou folosind `filter` care să elimine valoarea. Apoi va trebui să suprascrieți proprietatea care reține membrii cu versiunea filtrată a array-ului.

{{index "for/of loop", "iterable interface"}}

Metoda `from` poate utiliza o buclă `for`/`of` pentru a prelua valorile din obiectul iterabil și să apeleze `add` pentru a le adăuga pe rând la grupul nou creat.

hint}}

### Grupuri iterabile

{{index "groups (exercise)", [interface, object], "iterator interface", "Group class"}}

{{id group_iterator}}

Faceți clasa `Group` din exemplul anterior iterabilă. Consultați secțiunea despre interfața iterator din acest capitol dacă nu vă mai amintiți detaliile.

Dacă ați utilizat un array pentru a reprezenta membrii grupului, nu returnați pur și simplu iteratorul creat prin apelarea metodei `Symbol.iterator` asupra array-ului. Ar funcționa, dar ar fi împotriva scopului exercițiului.

Este ok dacă iteratorul vostru se comportă ciudat când grupul este modificat în timpul iterării.

{{if interactive

```{test: no}
// Your code here (and the code from the previous exercise)

for (let value of Group.from(["a", "b", "c"])) {
  console.log(value);
}
// → a
// → b
// → c
```

if}}

{{hint

{{index "groups (exercise)", "Group class", "next method"}}

Probabil că ar merita să definim o nouă clasă, `GroupIterator`. Instanțele iterator trebuie să aibă o proprietate care urmărește poziția curentă în grup. De fiecare dată când se apelează `next`, aceasta verifică dacă iterarea s-a încheiat, iar dacă nu, trece la următorul element și îl returnează.

Clasa `Group` primește o metodă numită ca și `Symbol.iterator` care, când va fi apelată, va returna o nouă instanță a clasei iterator pentru acel grup.

hint}}

### Împrumutarea unei metode

Anterior în acest capitol am menționat că `hasOwnProperty` poate fi utilizată pentru obiecte ca o alternativă mai robustă la operatorul `in`, atunci când vrem să ignorăm proprietățile prototipului. Dar cum procedăm dacă map-ul nostru ar trebui să includă cheia `"hasOwnProperty"`? Nu vom mai putea să apelăm acea metodă deorece proprietatea proprie a obiectului ascunde metoda și o face inaccesibilă.

Puteți să vă imaginați o modalitate de a apela `hasOwnProperty` asupra unui obiect care are deja o proprietate cu acel nume?

{{if interactive

```{test: no}
let map = {one: true, two: true, hasOwnProperty: true};

// Fix this call
console.log(map.hasOwnProperty("one"));
// → true
```

if}}

{{hint

Amintiți-vă că metodele care există pe obiecte simple provin din `Object.prototype`.

De asemenea, puteți apela o funcție cu un binding `this` specific utilizând metoda `call`.

hint}}
