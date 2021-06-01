{{meta {load_files: ["code/packages_chapter_10.js", "code/chapter/07_robot.js"]}}}

# Module

{{quote {author: "Tef", title: "Programming is Terrible", chapter: true}

Scrieți cod care este ușor de șters, nu ușor de extins.

quote}}

{{index "Yuan-Ma", "Book of Programming"}}

{{figure {url: "img/chapter_picture_10.jpg", alt: "Imaginea unei construcții din piese modulare", chapter: framed}}}

{{index organization, [code, "structure of"]}}

Programul ideal are o structură clară ca a unui cristal. Modul în care funcționează este ușor de explicat și fiecare parte are un rol bine definit.

{{index "organic growth"}}

Un program tipic are o creștere organică. Noile piese de funcționalitate sunt adăugate pe măsură ce sunt necesare. Structurarea - și păstrarea structurii - sunt eforturi suplimentare. Este un efort care va fi recompensat doar în viitor, următoarea dată când cineva va lucra asupra programului. Astefel că este tentant să ignorăm aecastă parte și să permitem ca părți ale programului să devină profund încâlcite.

{{index readability, reuse, isolation}}

Aceasta cauzează două probleme din punct de vedere practic. Mai întâi, înțelegerea unui asemenea sistem este dificilă. Dacă orice poate influența orice altă parte este dificil să privim orice piesă dată în izolare. Trebuie să dezvoltați o înțelegere holistică asupra întregului sistem. Apoi, dacă încercați să utilizați orice funcționalitate dintr-un asemenea program într-o altă situație, rescrierea ei ar putea fi mai ușoară decât încercarea de a descâlci piesa din contextul în care se află.

Expresia "o mare minge de noroi (big ball of mud)" este adesea utilizată pentru asemenea programe mari și fără structură. Totul este lipit împreună iar atunci când încercați să extrageți o piesă, totul vine după ea și vă "murdăriți pe mâini".

## Module

{{index dependency, [interface, module]}}

_Modulele_ sunt o încercare de a evita aceste probleme. Un _modul_ este o bucată de program care specifică pe ce alte piese se bazează și ce funcționalitate oferă pentru a fi utilizată de către alte module (_interfața_ sa).

{{index "big ball of mud"}}

Interfețele modulellor au multe în comun cu interfețele obiectelor, așa cum am văzut în [capitolul ?](object#interface). Ele permit ca o parte a modulului să fie disponibilă lumii exterioare și păstrează restul funcționalității privată. Prin restricționarea modului în care modulele pot interacționa între ele, sistemul începe să semene cu un LEGO, în care piesele interacționează prin conectori bine definiți.

{{index dependency}}

Relațiile dintre module se numesc _dependențe_. Când un modul are nevoie de o piesă din alt modul, spunem că depinde de acel modul. Atunci când acest fapt este clar specificat în modul, poate fi utilizat pentru a determina ce alte module trebuie să fie prezente pentru a putea utiliza un anumit modul, dar și pentru a încărca automat dependențele.

Pentru a separa modulele în acest mod, fiecare are nevoie de propriul său domeniu de vizibilitate privat.

Doar plasarea codului JavaScript în fișiere diferite nu satisface aceste cerințe. Fișierele încă partajează același spațiu de nume global. Ele pot, intenționat sau accidental, să interfere la nivelul bindingurilor. Iar structura dependențelor rămâne neclară. Putem găsi soluții mai bune, așa cum vom vedea în acest capitol.

{{index design}}

Conceperea unei structuri modulare adecvate pentru un program poate fi dificilă. În faza în care încă explorați problema, încercând diferite abordări pentru a vedea dacă acestea funcționează, ar putea fi o idee bună să nu vă concentrați prea mult pe modularitate deoarece acesta poate fi un factor major de distragere. Imediat ce aveți o abordare ce pare solidă, puteți începe să o organizați pe module.

## Pachete (Packages)

{{index bug, dependency, structure, reuse}}

Unul dintre avantajele construirii unui program din piese separate, și al abilității de a rula acele piese independent, este că ați putea utiliza aceeași piese în programe diferite.

{{index "parseINI function"}}

Dar cum ați putea realiza asta? Să presupunem că aș dori să utilizez funcția `parseINI` din [capitolul ?](regexp#ini) într-un alt program. Dacă este clar de ce depinde funcția (în acest caz, nimic), pot doar să copiez codul necesar în noul meu proiect și apoi să îl utilizez. Dar apoi, dacă descopăr o greșeală în acel cod, probabil că o voi repara în programul în care lucrez pe moment și voi uita să o repar și în celălat program.

{{index duplication, "copy-paste programming"}}

Când începeți să duplicați codul, veți descoperi rapid că pierdeți timp și energie cu mutarea copiilor și actualizarea lor.

Aici este locul în care se vede utilitatea _pachetelor (packages)_. Un pachet este o bucată de cod care poate fi distribuită (copiată și instalată). Ar putea conține unul sau mai multe module și conține informații despre alte pachete de care depinde. De asemenea, un pachet este de obicei însoțit de documentație care explică ce funcționalitate conține astfel încât să fie ușor de utilizat de către alții.

Atunci când se descoperă o problemă într-un pachet sau se adaugă nouă funcționalitate, pachetul este actualizat. Acum, programele care depind de el (care pot să fie la rândul lor pachete), pot fi actualizate la noua versiune.

{{id modules_npm}}

{{index installation, upgrading, "package manager", download, reuse}}

Acest mod de lucru necesită o infrastructură. Avem nevoie de un loc în care să reținem și să căutăm pachete și de un mod convenabil de a le instala și actualiza. În ecosistemul JavaScript, această structură este oferită de către NPM ([_https://npmjs.org_](https://npmjs.org)).

NPM înseamnă două lucruri: un serviciu online unde se pot descărca sau încărca pachete, dar și un program (parte din Node.js) care ajută cu instalarea și gestionarea pachetelor.

{{index "command line"}}

[Capitolul ?](node) vă va arăta cum să instalați astfel de pachete local utilizând programul `npm` în linia de comandă.

Disponibilitatea pentru descărcare a unor pachete de calitate este extrem de valoroasă. Înseamnă că adesea veți putea evita să reinventați un program scris anterior de către 100 de oameni și să aveți o implementare solidă și bine testată la câteva taste distanță.

{{index maintenance}}

Software-ul este ieftin de copiat astfel că, odată ce a fost scris, distribuirea sa către alte persoane este un proces eficient. Dar scrierea sa este muncă și va trebui să răspundeți persoanelor care poate au găsit probleme în cod, sau doresc să propună noi funcționalități, înseamnă și mai multă muncă.

Implicit, dețineți drepturile de autor pentru codul pe care îl scrieți și alte persoane îl pot utiliza doar cu permisiunea voastră. Dar deoarece unele persoane sunt drăguțe și publicarea unui software de calitate vă aduce puțină faimă printre programatori, multe pachete sunt publicate sub o licență care permite explicit utilizarea lor de către alte persoane.

Majoritatea codului din NPM este licențiat în acest mod. Unele licențe vă solicită să publicați codul pe care îl construiți pe baza unui pachet sub aceați licență. Altele sunt mai puțin pretențioase, solicitându-vă doar să păstrați licența în codul pe care îl distribuiți. Comunitatea JavaScript utilizează cel mai frecvent al doilea tip de licență. Când utilizați pachetele altora, asigurați-vă că sunteți conștienți de licența lor.

## Module improvizate

Până în 2015, limbajul JavaScript nu avea un sistem de module. Deși oamenii construiau sisteme de mari dimensiuni în JavaScript de mai mult de o decadă și aveau nevoie de module.

{{index [function, scope], [interface, module], [object, as module]}}

Astfel că au construit propriul lor sistem de module pe baza limbajului. Puteți utiliza funcțiile JavaScript pentru a crea domenii de vizibilitate locale și obiectele pentru a reprezenta interfețele modulelor.

{{index "Date class", "weekDay module"}}

Acesta este un modul pentru a converti între numele și indexul zilelor săptămânii (similar cu metoda `Date.getDay`). Interfața sa constă din `weekDay.name` și `weekDay.number`, iar bindingul local `names` este ascuns în interiorul domeniului de vizibilitate al unei expresii de tip funcție invocate imediat (IIFE - Immediately Invoked Function Expression).

```
const weekDay = function() {
  const names = ["Sunday", "Monday", "Tuesday", "Wednesday",
                 "Thursday", "Friday", "Saturday"];
  return {
    name(number) { return names[number]; },
    number(name) { return names.indexOf(name); }
  };
}();

console.log(weekDay.name(weekDay.number("Sunday")));
// → Sunday
```

{{index dependency, [interface, module]}}

Acest stil de a scrie module permite un anumit grad de izolare, dar nu declară dependențele. În loc de aceasta, doar îți plasează interfața în domeniul de vizibilitate global și așteaptă ca dependențele sale, dacă există, să facă același lucru. Mult timp, aceasta a fost principala abordare în web development, dar este o abordare învechită acum.

Dacă vrem ca relațiile de dependență să facă parte din cod, trebuie să preluăm controlul asupra încărcării dependențelor. Pentru aceasta, avem nevoie de abilitatea de executa stringuri ca și cod. JavaScript poate să facă asta.

{{id eval}}

## Evaluarea datelor ca și cod

{{index evaluation, interpretation}}

Sunt mai multe moduri în care puteți prelua date (un string de cod) și să le rulați ca și parte a programului curent.

{{index isolation, eval}}

Cel mai evident mod este operatorul special `eval`, care va executa un string în domeniul de vizibilitate _curent_. Aceasta este de regulă o idee proastă deoarece încalcă unele dintre proprietățile domeniilor de vizibilitate, cum ar fi predictibilitatea asupra bindingurilor la care se referă un nume.

```
const x = 1;
function evalAndReturnX(code) {
  eval(code);
  return x;
}

console.log(evalAndReturnX("var x = 2"));
// → 2
console.log(x);
// → 1
```

{{index "Function constructor"}}

Un mod mai puțin intimidant de a interpreta datele ca și cod este utilizarea constructorului `Function`. Acesta primește două argumente: un string ce conține o listă de nume ale argumentelor, separate prin virgulă, și un string ce conține corpul funcției. Codul va fi împachetat într-o valoare de tip funcție așa că va genera propriul său domeniu de vizibilitate și nu va acționa impredictibil asupra altor domenii de vizibilitate.

```
let plusOne = Function("n", "return n + 1;");
console.log(plusOne(4));
// → 5
```

Cu siguranță este ceea ce ne trebuie pentru un sistem de module. Putem împacheta codul modulului într-o funcție și putem utiliza domeniul de vizibilitate al funcției ca și domeniu de vizibilitate al modulului.

## CommonJS

{{id commonjs}}

{{index "CommonJS modules"}}

Cea mai frecvent utilizată abordare pentru a construi module în JavaScript sunt _modulele CommonJS_. Node.js o utilizează și acesta este sistemul utilizat de majoritatea pachetelor din NPM.

{{index "require function", [interface, module]}}

Conceptul principal din modulele CommonJS este o funcție numită `require`. Când apelați această funcție cu numele modulului ce reprezintă o dependență, se asigură că modulul este încărcat și returnează interfața sa.

{{index "exports object"}}

Deoarece loader-ul împachetează codul modulului într-o funcție, modulele vor avea automat propriul domeniu de vizibilitate local. Tot ceea ce trebuie să facă este să apeleze `require` pentru a își rezolva propriile dependențe și să își plaseze interfața în obiectul legat de `exports`.

{{index "formatDate module", "Date class", "ordinal package", "date-names package"}}

Modulul dat mai jos ca și exemplu pune la dispoziție o funcție pentru formatarea datelor. Utilizează două pachete din NPM - `ordinal` pentru a converti numere în stringuri, cum ar fi `"1st"` și `"2nd"`, și `date-names` pentru a obține numele în limba engleză pentru zilele săptămânii și lunile anului. Exportă o singură funcție, `formatDate` care primește ca argumente un obiect `Date` și un string de formatare.

Stringul de formatare poate conține coduri care determină formatul, cum ar fi `YYYY` pentru anul complet și `Do` pentru numeralul ordinal reprezentând ziua din lună. Puteți specifica un string cum ar fi `"MMMM Do YYYY"` pentru a obține un rezultat "November 22nd 2017".

```
const ordinal = require("ordinal");
const {days, months} = require("date-names");

exports.formatDate = function(date, format) {
  return format.replace(/YYYY|M(MMM)?|Do?|dddd/g, tag => {
    if (tag == "YYYY") return date.getFullYear();
    if (tag == "M") return date.getMonth();
    if (tag == "MMMM") return months[date.getMonth()];
    if (tag == "D") return date.getDate();
    if (tag == "Do") return ordinal(date.getDate());
    if (tag == "dddd") return days[date.getDay()];
  });
};
```

{{index "destructuring binding"}}

Interfața `ordinal` este o singură funcție, iar `date-names` exportă un obiect ce conține mai multe lucruri - `days` și `months` sunt array-uri de nume. Destructurarea este foarte convenabilă când se crează bindinguri pentru interfețele importate.

Modulul își adaugă propria interfață în `exports` astfel încât modulele care depind de el primesc acces la aceasta. Am putea utiliza modulul astfel:

```
const {formatDate} = require("./format-date");

console.log(formatDate(new Date(2017, 9, 13),
                       "dddd the Do"));
// → Friday the 13th
```

{{index "require function", "CommonJS modules", "readFile function"}}

{{id require}}

Am puttea defini `require` într-o formă minimală, astfel:

```{test: wrap, sandbox: require}
require.cache = Object.create(null);

function require(name) {
  if (!(name in require.cache)) {
    let code = readFile(name);
    let module = {exports: {}};
    require.cache[name] = module;
    let wrapper = Function("require, exports, module", code);
    wrapper(require, module.exports, module);
  }
  return require.cache[name].exports;
}
```

{{index [file, access]}}

În acest cod, `readFile` este o funcție inventată care ar citi dintr-un fișier și ar returna conținutul său ca și string. JavaScript standard nu oferă o asemenea funcționalitate dar diferitele medii JavaScript, cum ar fi browserele și Node.js vă pun la dispoziție modul lor propriu de a accesa fișierele. Exemplul doar "pretinde" că `readFile` există.

{{index cache, "Function constructor"}}

Pentru a evita încărcarea aceluiași modul de mai multe ori, `require` menține un index (cache) al modulelor deja încărcate. Când va fi apelată, această funcție verifică mai întâi dacă modulul solicitat a fost încărcat și, dacă nu, atunci îl încarcă. Încărcarea presupune citirea codului modulului, împachetarea codului într-o funcție și apelarea acesteia.

{{index "ordinal package", "exports object", "module object", [interface, module]}}

Interfața pachetului `ordinal` folosit anterior nu este un obiect ci o funcție. Un capriciu al modulelor CommonJS, cu toate că sistemul de module va crea un obiect gol pentru interfață (legat de `exports`), îl puteți înlocui cu orice valoare prin suprascrierea `module.exports`. Acest procedeu este utilizat de multe module pentru a exporta o singură valoare în locul unui obiect pentru interfață.

Prin definirea `require`, `exports` și `module` ca și parametrii pentru funcția de împachetare generată (și transmiterea valorilor corespunzătoare la apel), loader-ul se asigură că aceste bindinguri sunt disponibile în domeniul de vizibilitate al modulului.

{{index resolution, "relative path"}}

Modul în care stringul transmis la `require` este translatat în numele unui fișier sau o adresă web diferă de la un sistem la altul. Când începe cu `"./"` sau `"../"`, este în general interpretat ca și o cale relativă la numele fișierului ce conține modulul. `"./format-date"` s-a referi la fișierul `format-date.js` din același folder.

Când numele nu este relativ, Node.js va căuta un pachet instalat cu acel nume. În exemplele din acest capitol vom interpreta asemenea nume ca referindu-se la pachete NPM. Vom intra în mai multe detalii despre cum putem instala și utiliza module NPM în [capitolul ?](node).

{{id modules_ini}}

{{index "ini package"}}

Acum, în loc de a scrie propriul vostru parser pentru fișiere INI, putem utiliza unul din NPM.

```
const {parse} = require("ini");

console.log(parse("x = 10\ny = 20"));
// → {x: "10", y: "20"}
```

## Module ECMAScript

((CommonJS modules)) work quite well and, in combination with NPM, 
have allowed the JavaScript community to start sharing code on a large
scale.

{{index "exports object", linter}}

But they remain a bit of a duct-tape ((hack)). The ((notation)) is
slightly awkward—the things you add to `exports` are not available in
the local ((scope)), for example. And because `require` is a normal
function call taking any kind of argument, not just a string literal,
it can be hard to determine the dependencies of a module without
running its code.

{{index "import keyword", dependency, "ES modules"}}

{{id es}}

This is why the JavaScript standard from 2015 introduces its own,
different module system. It is usually called _((ES modules))_, where
_ES_ stands for ((ECMAScript)). The main concepts of dependencies and
interfaces remain the same, but the details differ. For one thing, the
notation is now integrated into the language. Instead of calling a
function to access a dependency, you use a special `import` keyword.

```
import ordinal from "ordinal";
import {days, months} from "date-names";

export function formatDate(date, format) { /* ... */ }
```

{{index "export keyword", "formatDate module"}}

Similarly, the `export` keyword is used to export things. It may
appear in front of a function, class, or binding definition (`let`,
`const`, or `var`).

{{index [binding, exported]}}

An ES module's interface is not a single value but a set of named
bindings. The preceding module binds `formatDate` to a function. When
you import from another module, you import the _binding_, not the
value, which means an exporting module may change the value of the
binding at any time, and the modules that import it will see its new
value.

{{index "default export"}}

When there is a binding named `default`, it is treated as the module's
main exported value. If you import a module like `ordinal` in the
example, without braces around the binding name, you get its `default`
binding. Such modules can still export other bindings under different
names alongside their `default` export.

To create a default export, you write `export default` before an
expression, a function declaration, or a class declaration.

```
export default ["Winter", "Spring", "Summer", "Autumn"];
```

It is possible to rename imported bindings using the word `as`.

```
import {days as dayNames} from "date-names";

console.log(dayNames.length);
// → 7
```

Another important difference is that ES module imports happen before
a module's script starts running. That means `import` declarations
may not appear inside functions or blocks, and the names of
dependencies must be quoted strings, not arbitrary expressions.

At the time of writing, the JavaScript community is in the process of
adopting this module style. But it has been a slow process. It took
a few years, after the format was specified, for browsers and Node.js
to start supporting it. And though they mostly support it now, this
support still has issues, and the discussion on how such modules
should be distributed through ((NPM)) is still ongoing.

Many projects are written using ES modules and then automatically
converted to some other format when published. We are in a
transitional period in which two different module systems are used
side by side, and it is useful to be able to read and write code in
either of them.

## Building and bundling

{{index compilation, "type checking"}}

In fact, many JavaScript projects aren't even, technically, written in
JavaScript. There are extensions, such as the type checking
((dialect)) mentioned in [Chapter ?](error#typing), that are widely
used. People also often start using planned extensions to the language
long before they have been added to the platforms that actually run
JavaScript.

To make this possible, they _compile_ their code, translating it from
their chosen JavaScript dialect to plain old JavaScript—or even to a
past version of JavaScript—so that old ((browsers)) can run it.

{{index latency, performance, [file, access], [network, speed]}}

Including a modular program that consists of 200 different
files in a ((web page)) produces its own problems. If fetching a
single file over the network takes 50 milliseconds, loading
the whole program takes 10 seconds, or maybe half that if you can
load several files simultaneously. That's a lot of wasted time.
Because fetching a single big file tends to be faster than fetching a
lot of tiny ones, web programmers have started using tools that roll
their programs (which they painstakingly split into modules) back into
a single big file before they publish it to the Web. Such tools are
called _((bundler))s_.

{{index "file size"}}

And we can go further. Apart from the number of files, the _size_ of
the files also determines how fast they can be transferred over the
network. Thus, the JavaScript community has invented _((minifier))s_.
These are tools that take a JavaScript program and make it smaller by
automatically removing comments and whitespace, renaming bindings, and
replacing pieces of code with equivalent code that take up less space.

{{index pipeline, tool}}

So it is not uncommon for the code that you find in an NPM package or
that runs on a web page to have gone through _multiple_ stages of
transformation—converted from modern JavaScript to historic
JavaScript, from ES module format to CommonJS, bundled, and minified.
We won't go into the details of these tools in this book since they
tend to be boring and change rapidly. Just be aware that the
JavaScript code you run is often not the code as it was written.

## Module design

{{index [module, design], [interface, module], [code, "structure of"]}}

Structuring programs is one of the subtler aspects of programming. Any
nontrivial piece of functionality can be modeled in various ways.

Good program design is subjective—there are trade-offs involved and
matters of taste. The best way to learn the value of well-structured
design is to read or work on a lot of programs and notice what works
and what doesn't. Don't assume that a painful mess is "just the way it
is". You can improve the structure of almost everything by putting
more thought into it.

{{index [interface, module]}}

One aspect of module design is ease of use. If you are designing
something that is intended to be used by multiple people—or even by
yourself, in three months when you no longer remember the specifics of
what you did—it is helpful if your interface is simple and
predictable.

{{index "ini package", JSON}}

That may mean following existing conventions. A good example is the
`ini` package. This module imitates the standard `JSON` object by
providing `parse` and `stringify` (to write an INI file) functions,
and, like `JSON`, converts between strings and plain objects. So the
interface is small and familiar, and after you've worked with it once,
you're likely to remember how to use it.

{{index "side effect", "hard disk", composability}}

Even if there's no standard function or widely used package to
imitate, you can keep your modules predictable by using simple ((data
structure))s and doing a single, focused thing. Many of the INI-file
parsing modules on NPM provide a function that directly reads such a
file from the hard disk and parses it, for example. This makes it
impossible to use such modules in the browser, where we don't have
direct file system access, and adds complexity that would have been
better addressed by _composing_ the module with some file-reading
function.

{{index "pure function"}}

This points to another helpful aspect of module design—the ease with
which something can be composed with other code. Focused modules that
compute values are applicable in a wider range of programs than bigger
modules that perform complicated actions with side effects. An INI
file reader that insists on reading the file from disk is useless in a
scenario where the file's content comes from some other source.

{{index "object-oriented programming"}}

Relatedly, stateful objects are sometimes useful or even necessary,
but if something can be done with a function, use a function. Several
of the INI file readers on NPM provide an interface style that
requires you to first create an object, then load the file into your
object, and finally use specialized methods to get at the results.
This type of thing is common in the object-oriented tradition, and
it's terrible. Instead of making a single function call and moving on,
you have to perform the ritual of moving your object through various
states. And because the data is now wrapped in a specialized object
type, all code that interacts with it has to know about that type,
creating unnecessary interdependencies.

Often defining new data structures can't be avoided—only a few 
basic ones are provided by the language standard, and many types of
data have to be more complex than an array or a map. But when an
array suffices, use an array.

An example of a slightly more complex data structure is the graph from
[Chapter ?](robot). There is no single obvious way to represent a
((graph)) in JavaScript. In that chapter, we used an object whose
properties hold arrays of strings—the other nodes reachable from that
node.

There are several different pathfinding packages on ((NPM)), but none
of them uses this graph format. They usually allow the graph's edges to
have a weight, which is the cost or distance associated with it. That isn't
possible in our representation.

{{index "Dijkstra, Edsger", pathfinding, "Dijkstra's algorithm", "dijkstrajs package"}}

For example, there's the `dijkstrajs` package. A well-known approach
to pathfinding, quite similar to our `findRoute` function, is called
_Dijkstra's algorithm_, after Edsger Dijkstra, who first wrote it
down. The `js` suffix is often added to package names to indicate the
fact that they are written in JavaScript. This `dijkstrajs` package
uses a graph format similar to ours, but instead of arrays, it uses
objects whose property values are numbers—the weights of the edges.

So if we wanted to use that package, we'd have to make sure that our
graph was stored in the format it expects. All edges get the same
weight since our simplified model treats each road as having the
same cost (one turn).

```
const {find_path} = require("dijkstrajs");

let graph = {};
for (let node of Object.keys(roadGraph)) {
  let edges = graph[node] = {};
  for (let dest of roadGraph[node]) {
    edges[dest] = 1;
  }
}

console.log(find_path(graph, "Post Office", "Cabin"));
// → ["Post Office", "Alice's House", "Cabin"]
```

This can be a barrier to composition—when various packages are using
different data structures to describe similar things, combining them
is difficult. Therefore, if you want to design for composability,
find out what ((data structure))s other people are using and, when
possible, follow their example.

## Summary

Modules provide structure to bigger programs by separating the code
into pieces with clear interfaces and dependencies. The interface is
the part of the module that's visible from other modules, and the
dependencies are the other modules that it makes use of.

Because JavaScript historically did not provide a module system, the
CommonJS system was built on top of it. Then at some point it _did_ get
a built-in system, which now coexists uneasily with the CommonJS
system.

A package is a chunk of code that can be distributed on its own. NPM
is a repository of JavaScript packages. You can download all kinds of
useful (and useless) packages from it.

## Exercises

### A modular robot

{{index "modular robot (exercise)", module, robot, NPM}}

{{id modular_robot}}

These are the bindings that the project from [Chapter ?](robot)
creates:

```{lang: "text/plain"}
roads
buildGraph
roadGraph
VillageState
runRobot
randomPick
randomRobot
mailRoute
routeRobot
findRoute
goalOrientedRobot
```

If you were to write that project as a modular program, what modules
would you create? Which module would depend on which other module, and
what would their interfaces look like?

Which pieces are likely to be available prewritten on NPM? Would you
prefer to use an NPM package or write them yourself?

{{hint

{{index "modular robot (exercise)"}}

Here's what I would have done (but again, there is no single _right_
way to design a given module):

{{index "dijkstrajs package"}}

The code used to build the road graph lives in the `graph` module.
Because I'd rather use `dijkstrajs` from NPM than our own pathfinding
code, we'll make this build the kind of graph data that `dijkstrajs`
expects. This module exports a single function, `buildGraph`. I'd have
`buildGraph` accept an array of two-element arrays, rather than
strings containing hyphens, to make the module less dependent on the
input format.

The `roads` module contains the raw road data (the `roads` array) and
the `roadGraph` binding. This module depends on `./graph` and exports
the road graph.

{{index "random-item package"}}

The `VillageState` class lives in the `state` module. It depends on
the `./roads` module because it needs to be able to verify that a
given road exists. It also needs `randomPick`. Since that is a
three-line function, we could just put it into the `state` module as
an internal helper function. But `randomRobot` needs it too. So we'd
have to either duplicate it or put it into its own module. Since this
function happens to exist on NPM in the `random-item` package, a good
solution is to just make both modules depend on that. We can add the
`runRobot` function to this module as well, since it's small and
closely related to state management. The module exports both the
`VillageState` class and the `runRobot` function.

Finally, the robots, along with the values they depend on such as
`mailRoute`, could go into an `example-robots` module, which depends
on `./roads` and exports the robot functions. To make it possible for
`goalOrientedRobot` to do route-finding, this module also depends
on `dijkstrajs`.

By offloading some work to ((NPM)) modules, the code became a little
smaller. Each individual module does something rather simple and can
be read on its own. Dividing code into modules also often suggests
further improvements to the program's design. In this case, it seems a
little odd that the `VillageState` and the robots depend on a specific
road graph. It might be a better idea to make the graph an argument to
the state's constructor and make the robots read it from the state
object—this reduces dependencies (which is always good) and makes it
possible to run simulations on different maps (which is even better).

Is it a good idea to use NPM modules for things that we could have
written ourselves? In principle, yes—for nontrivial things like the
pathfinding function you are likely to make mistakes and waste time
writing them yourself. For tiny functions like `random-item`, writing
them yourself is easy enough. But adding them wherever you need them
does tend to clutter your modules.

However, you should also not underestimate the work involved in
_finding_ an appropriate NPM package. And even if you find one, it
might not work well or may be missing some feature you need. On top
of that, depending on NPM packages means you have to make sure they
are installed, you have to distribute them with your program, and you
might have to periodically upgrade them.

So again, this is a trade-off, and you can decide either way depending
on how much the packages help you.

hint}}

### Roads module

{{index "roads module (exercise)"}}

Write a ((CommonJS module)), based on the example from [Chapter
?](robot), that contains the array of roads and exports the graph
data structure representing them as `roadGraph`. It should depend on a
module `./graph`, which exports a function `buildGraph` that is used
to build the graph. This function expects an array of two-element
arrays (the start and end points of the roads).

{{if interactive

```{test: no}
// Add dependencies and exports

const roads = [
  "Alice's House-Bob's House",   "Alice's House-Cabin",
  "Alice's House-Post Office",   "Bob's House-Town Hall",
  "Daria's House-Ernie's House", "Daria's House-Town Hall",
  "Ernie's House-Grete's House", "Grete's House-Farm",
  "Grete's House-Shop",          "Marketplace-Farm",
  "Marketplace-Post Office",     "Marketplace-Shop",
  "Marketplace-Town Hall",       "Shop-Town Hall"
];
```

if}}

{{hint

{{index "roads module (exercise)", "destructuring binding", "exports object"}}

Since this is a ((CommonJS module)), you have to use `require` to
import the graph module. That was described as exporting a
`buildGraph` function, which you can pick out of its interface object
with a destructuring `const` declaration.

To export `roadGraph`, you add a property to the `exports` object.
Because `buildGraph` takes a data structure that doesn't precisely
match `roads`, the splitting of the road strings must happen in your
module.

hint}}

### Circular dependencies

{{index dependency, "circular dependency", "require function"}}

A circular dependency is a situation where module A depends on B, and
B also, directly or indirectly, depends on A. Many module systems
simply forbid this because whichever order you choose for loading such
modules, you cannot make sure that each module's dependencies have
been loaded before it runs.

((CommonJS modules)) allow a limited form of cyclic dependencies. As
long as the modules do not replace their default `exports` object and
don't access each other's interface until after they finish loading,
cyclic dependencies are okay.

The `require` function given [earlier in this
chapter](modules#require) supports this type of dependency cycle. Can
you see how it handles cycles? What would go wrong when a module in a
cycle _does_ replace its default `exports` object?

{{hint

{{index overriding, "circular dependency", "exports object"}}

The trick is that `require` adds modules to its cache _before_ it
starts loading the module. That way, if any `require` call made while
it is running tries to load it, it is already known, and the current
interface will be returned, rather than starting to load the module
once more (which would eventually overflow the stack).

If a module overwrites its `module.exports` value, any other module
that has received its interface value before it finished loading will
have gotten hold of the default interface object (which is likely
empty), rather than the intended interface value.

hint}}
