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

Interfețele modulelor au multe în comun cu interfețele obiectelor, așa cum am văzut în [capitolul ?](object#interface). Ele permit ca o parte a modulului să fie disponibilă lumii exterioare și păstrează restul funcționalității privată. Prin restricționarea modului în care modulele pot interacționa între ele, sistemul începe să semene cu un LEGO, în care piesele interacționează prin conectori bine definiți.

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

Modulele CommonJS funcționează destul de bine și, în combinație cu NPM, au permis comunității JavaScript să partajeze cod pe scară largă.

{{index "exports object", linter}}

Dar ele rămân oarecum un hack. Notația este puțin ciudată - lucrurile pe care le adăugați la `exports` nu sunt disponibile în domeniul de vizibilitate local, de exemplu. Și, deoarece `require` este un apel normal de funcție ce poate accepta orice argument, nu doar un literal de tip string, poate fi dificil să determinați dependențele unui modul fără a-i executa codul.

{{index "import keyword", dependency, "ES modules"}}

{{id es}}

Acesta este motivul pentru care standardul din 2015 al JavaScript introduce propriul său sistem de module. De regulă este denumit _module ES_, unde _ES_ provine de la ECMAScript. Conceptele principale ale dependințelor și interfețelor rămân aceleași, dar detaliile diferă. Notația este acum integrată în limbaj. În loc să folosiți o funcție pentru a accesa o dependență, veți utiliza cuvântul cheie `import`.

```
import ordinal from "ordinal";
import {days, months} from "date-names";

export function formatDate(date, format) { /* ... */ }
```

{{index "export keyword", "formatDate module"}}

Similar, cuvântul cheie `export` este utilizat pentru a exporta lucruri. Poate să apară în fața unei definiții de funcție, clasă sau binding (`let`, `const` sau `var`).

{{index [binding, exported]}}

Interfața unui modul ES nu este o singură valoare ci un set de bindinguri. Modului anterior leagă `formatDate` spre o funcție. Când îl importați din alt modul, importați _bindingul_, nu valoarea, ceea ce înseamnă că modulul care o exportă ar putea schimba valoarea bindingului în orice moment, iar modulul care importă va vedea noua valoare.

{{index "default export"}}

Dacă există un binding denumit `default`, acesta este tratat ca fiind valoarea principală exportată. Dacă importați un modul cum ar fi `ordinal` din exemplu, fără acolade în jurul numelui bindingului, veți obține bindingul `default`. Asemenea module pot să exporte și alte bindinguri sub nume diferite, pe lângă exportul `default`.

Pentru a crea un export implicit, veți scrie `export default` înaintea unei expresii, a unei declarații de funcție sau a unei declarații de clasă.

```
export default ["Winter", "Spring", "Summer", "Autumn"];
```

Putem redenumi bindingurile importante folosind cuvântul `as`.

```
import {days as dayNames} from "date-names";

console.log(dayNames.length);
// → 7
```

O altă diferență importantă este că importurile unui modul ES au loc înainte de a începe rularea codului modulului. Înseamnă că declarațiile `import` nu pot fi plasate în interiorul funcțiilor sau blocurilor iar numele dependențelor trebuie să fie stringuri, nu expresii arbitrare.

Multe proiecte sunt scrise folosind module ES și apoi convertite automat în alt format, înainte de publicare. Suntem într-o perioadă de tranziție în care există două sisteme de module și ete util să putem citi și scrie cod în oricare dintre ele.

## Construirea și împachetarea

{{index compilation, "type checking"}}

De fapt, multe proiecte JavaScript nici măcar nu sunt scrise în JavaScript. Există extensii, cum ar fi dialectul care verifică tipurile menționat în [capitolul ?](error#typing), care sunt utilizate pe larg. Oamenii încep să utilizeze extensii planificate ale limbajului cu mult înainte de a fi adăugate la platformele care de fapt rulează JavaScript.

Pentru a face posibil acest lucru, își _compilează_ codul, translatându-l din dialectul JavaScript ales în JavaScript standard - sau chiar într-o versiune mai veche de JavaScript, care să suporte browsere mai vechi.

{{index latency, performance, [file, access], [network, speed]}}

Includerea unui program modular care constă din 200 de fișiere diferite într-o pagină web produce propriile provocări. Dacă transferarea unui singur fișier în rețea necesită 50 milisecunde, încărcarea întregului program va necesita 10 secunde, poate mai puțin dacă puteți încărca simultan mai multe fișiere. Este mult timp pierdut. Deoarece încărcarea unui singur fișier mare tinde să fie mai rapidă decât încărcarea multor fișiere mici, programatorii web au început să utilizeze unelte care împachetează programele într-un singur fișier mare înainte de a-l publica pe Web. Asemenea instrumente se numesc _bundlere_.

{{index "file size"}}

Și putem merge mai departe. Pe lângă numărul de fișiere, _dimensiunea_ fișierelor determină și ea rapiditatea cu care acestea sunt transferate prin rețea. Prin urmare, comunitatea JavaScript a inventat _minificatoare (minifiers)_. Aceastea sunt instrumente care iau un program JavaScript și îl micșorează prin eliminarea comentariilor și a spațiilor, redenumirea bindingurilor și înlocuirea unor bucăți de cod cu cod echivalent care consumă mai puțin spațiu.

{{index pipeline, tool}}

Prin urmare, nu este deloc neobișnuit ca ceea ce găsiți într-un pachet NPM sau ceea ce rulează într-o pagină web să fie cod care a parcurs mai multe etape de transformare - conversie din cod JavaScript modern în cod istoric, din formatul ES pentru module în formatul CommonJS, împachetat și minificat. Nu vom intra în detalii cu privire la aceste instrumente pentru că sunt plictisitoare și se modifică rapid. Doar conștientizați că programul JavaScript executat este adesea diferit de codul care a fost scris.

## Designul modulelor

{{index [module, design], [interface, module], [code, "structure of"]}}

Structurarea programelor este unul dintre aspectele mai subtile ale programării. Orice bucată netrivială de funcționalitate poate fi modelată în mai multe moduri.

Designul bun al programului este subiectiv - există compromisuri și gusturi. Cea mai bună metodă de a învăța valoarea designului bine structurat este să citiți și să lucrați la multe programe și să observați ce funcționează și ce nu. Nu presupuneți că un program dureros de întreținut este "ceea ce este". Puteți îmbunătăți structura aproape oricărui program cu mai mult efort de analiză.

{{index [interface, module]}}

Un aspect important al designului modular este ușurința în utilizare. Dacă veți concepe un program care va fi utilizat de către mai multe persoane - sau chiar de către voi, peste trei luni cînd nu mai rețineți detaliile a ceea ce ați făcut, este de mare ajutor ca interfața să fie simplă și predictibilă.

{{index "ini package", JSON}}

Aceasta ar putea însemna să respectați convențiile. Un exemplu bun este pachetul `ini`. Acest modul imită obiectul standard `JSON` prin oferirea funcțiilor `parse` și `stringify` (utilă pentru a scrie un fișier INI) și, ca și `JSON`, convertește între stringuri și obiecte. Astfel că interfața este compactă și familiară și, după ce l-ați folosit odată, probabil veți memora cum să îl utilizați.

{{index "side effect", "hard disk", composability}}

Chiar dacă nu există o funcție standard sau un pachet utilizat pe larg pe care să îl imitați, puteți scrie module predictibile prin utilizarea unor structuri simple de date și implementarea focalizată a unui singur lucru. Multe module pentru parsarea fișierelor INI, disponibile în NPM, vă pun la dispoziție o funcție care citește direct dintr-un fișier de pe harddisk și parsează rezultatul, de exemplu. Din această cauză, modulele nu pot fi utilizate direct în browser, unde nu avem acces direct la sistemul de fișiere și adaugă complexitate care ar fi fost mai bine adresată prin _compunerea_ modului cu o funcție de citire a unui fișier.

{{index "pure function"}}

Aceasta indică un alt aspect util al arhitecturii modulare - ușurința cu care se poate compune funcționalitatea cu cod din alte module. Modulele țintite care calculează valori sunt aplicabile la un domeniu mai larg de programe decât modulele mai mari care realizează acțiuni complicate, cu efecte secundare. Un cititor de fișiere INI care insistă să citească dintr-un fișier de pe disc este inutil într-un scenariu în care conținutul fișierului provine din altă sursă.

{{index "object-oriented programming"}}

În corelație, obiectele ce își păstrează starea sunt uneori utile sau chiar necesare, dar atunci când puteți scrie pur și simplu o funcție, preferați această abordare. Mai multe cititoare de fișiere INI din NPM vă oferă o interfață care vă solicită mai întâi să creați un obiect, apoi să încărcați fișierul în obiect și apoi să utilizați metode specializate pentru a obține rezultatele. Acest mod de lucru face parte din tradiția orientării-obiect și este îngrozitor. În locul unui simplu apel către o funcție, trebuie să parcurgeți întreg ritualul mutării obiectului vostru prin diverse stări. Și, deoarece acum datele sunt împachetate într-un obiect specializat, tot codul care interacționează cu el trebuie acum să fie conștient de tipul acelui obiect, ceea ce crează interdependențe inutile.

Adesea, definirea unor noi structuri de date nu poate fi evitată - doar câteva structuri de bază sunt disponibile în limbajul standard și multe tipuri de date trebuie să fie mult mai complexe decât un array sau un map. Dar atunci când un array este suficient, folosiți un array.

Un exemplu de structură puțin mai complexă este un graf, ca în [capitolul ?](robot). Nu există un mod evident de a reprezenta un graf în JavaScript. În acel capitol, am utilizat un obiect ale cărui proprietăți memorau array-uri de stringuri - nodurile care pot fi atinse din fiecare nod.

Există mai multe pachete în NPM utile pentru căutarea unei căi, dar nici unul dintre ele nu folosește acest format de graf. Ele permit de regulă asocierea unei ponderi pentru fiecare latură a unui graf (legătură între două noduri), care reprezintă costul sau distanța asociată acesteia. În reprezentarea noastră, acest lucru nu este posibil.

{{index "Dijkstra, Edsger", pathfinding, "Dijkstra's algorithm", "dijkstrajs package"}}

De exemplu, există un pachet `dijkstrajs`. O abordare bine cunoscută petnru determinarea drumurilor, destul de similară cu funcția noastră `findRoute`, este _algoritmul lui Dijkstra_, denumit după Edgar Dijkstra, care a fost primul care a rezolvat astfel problema. Sufixul `js` este adăugat la numele pachetului pentru a indica faptul că este scris în JavaScript. Acest pachet `dijkstrajs` utilizează un format al grafului asemănător cu formatul nostru, dar în loc de array-uri folosește obiecte ale căror valori ale proprietăților sunt numere - ponderile laturilor.

Astfel, dacă vrem să utilizăm acest pachet, trebuie să ne asigurăm că graful nostru a fost memorat în formatul așteptat. Toate laturile vor avea aceeași pondere deoarece modelul nostru simplificat tratează fiecare drum ca având același cost (o mutare).

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

Aceasta poate fi o barieră pentru compoziție - când diferite pachete utilizează structuri de date diferite pentru a descrie același lucru, combinarea lor este dificilă. Prin urmare, dacă vreți să concepeți module pentru compozabilitate, identificați ce structuri de date folosesc ceilalți și, de câte ori este posibil, urmați exemplele.

## Rezumat

Modulele oferă structură programelor mai mari prin separarea codului în bucăți ce au interfețe și dependețe clare. Interfața este partea din modul care este vizibilă altor module iar dependențele sunt celelalte module care sunt utilizate.

Deoarece JavaScript nu oferea un sistem de module, sistemul CommonJS a fost construit pe baza limbajului. Apoi, la un moment dat, a apărut un sistem încorporat, care acum coexistă cu sistemul CommonJS.

Un pachet este o bucată de cod care poate fi distribuită. NPM este o sursă (repository) de pachete JavaScript. Puteți descărca tot felul de pachete utile (și inutile) din el.

## Exerciții

### Un robot modular

{{index "modular robot (exercise)", module, robot, NPM}}

{{id modular_robot}}

Acestea sunt bindingurile create în proiectul din [capitolul ?](robot):

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

Dacă ar fi să scrieți acel proiect ca și un program modular, ce module ați crea? Care ar fi dependențele dintre module și cum ar arăta interfețele lor?

Ce părți ar putea disponibile în NPM? Ați prefera să folosiți un pachet NPM sau să le rescrieți?

{{hint

{{index "modular robot (exercise)"}}

Iată ce aș face (dar din nou, nu există un singur mod _corect_ de a concepe un modul dat):

{{index "dijkstrajs package"}}

Codul utilizat pentru a construi graful drumurilor se află în modulul `graph`. Deoarece aș utiliza mai degrabă `dijkstrajs` din NPM decât codul meu pentru găsirea drumurilor, voi construi tipul de graf pe care îl așteaptă `disjkstrajs`. Modulul `graph` va exporta o singură funcție, `buildGraph`. Această funcție va accepta un array de array-uri de câte două elemente, în loc de stringuri separate prin `-`, pentru ca modulul să fie mai puțin dependent de formatul datelor de intrare.

Modulul `roads` va conține datele brute despre drumuri (array-ul `roads`) și bindingul `roadGraph`. Acest modul ar depinde de `./graph` și ar exporta graful drumurilor.

{{index "random-item package"}}

Clasa `VillageState` aș plasa-o în modulul `state`. Ar depinde de modulul `./roads` deoarece trebuie să poată verifica dacă un drum anume există. De asemenea, ar avea nevoie de `randomPick`. Deoarece aceasta este o funcție de trei linii, am putea să o scriem în modulul `state` ca și o funcție helper internă. Dar și `randomRobot` are nevoie de ea. Deci ar trebui fie să o duplicăm, fie să o plasăm în propriul ei modul. Deoarece această funcție există în NPM în pachetul `random-item`, o soluție bună ar fi ca ambele module să depindă de acest pachet. Și funcția `runRobot` am putea să o adăugăm tot în modulul `state` deoarece este scurtă și legată de managementul stării. Modulul ar exporta atât clasa `VillageState` cât și funcția `runRobot`.

În sfârșit, roboții, împreună cu valorile de care depind, cum ar fi `mailRoute`, pot fi plasați în modulul `example-robots` care depinde de `./roads` și exportă funcțiile pentru roboți. Pentru ca `goalOrientedRobot` să poată să găsească rutele, modulul de exemple va depinde de `dijkstrajs`.

Delegând parte din muncă modulelor NPM, codul a devenit mai scurt. Fiecare modul individual realizează ceva simplu și este de sine stătător. Împărțirea codului în module sugerează și moduri suplimentare de a îmbunătăți designul programului. În acest exemplu, este puțin ciudat că `VillageState` și roboții depind de un anume graf al drumurilor. Ar putea fi o idee mai bună ca graful să fie un argument al constructorului și roboții să citească din starea obiectului - ceea ce ar reduce dependențele (un lucru bun, întotdeauna) și ar fi posibilă executarea unor simulări cu hărți diferite (ceea ce e chiar și mai bine).

Este o idee bună să folosim module NPM pentru lucruri pe care le-am fi putut scrie noi? În principiu, da - pentru lucruri netriviale, cum ar fi funcția de căutare a drumurilor, este posibil să faceți greșeli și să pierdeți timp. Pentru funcții scurte, cum ar fi `random-item`, scrierea codului propriu este destul de ușoară. Dar adăugarea lor în toate locurile în care aveți nevoie tinde să adauge dezordine în modulele proprii.

Totuși, nu subestimați efortul necesar pentru _găsirea_ unui pachet NPM adecvat. Și chiar dacă găsiți unul, s-ar putea să nu funcționeze corespunzător sau să îi lipsească funcționalitate de care aveți nevoie. În plus, dependența de pachete NPM înseamnă că trebuie să vă asigurați că acestea sunt instalate, trebuie să le distribuiți împreună cu programul vostru și s-ar putea să fiți nevoiți să le actualizați periodic.

Deci, din nou, acesta este un compromis și puteți decide să alegeți oricare variantă, în funcție de cât de utile vă sunt pachetele.

hint}}

### Modulul `roads`

{{index "roads module (exercise)"}}

Scrieți un modul CommonJS, bazat pe exemplul din [capitolul ?](robot), care conține array-ul de drumuri și exportă structura de date pentru graf, reprezentându-le ca și `roadGraph`. Va trebui să depindă de un modul `./graph` care exportă o funcție `buildGraph` care este utilizată pentru a construi graful. Această funcție așteaptă un array de array-uri de câte două elemente (începutul și sfârșitul drumului).

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

Deoarece acesta este un modul CommonJS, va trebui să folosiți `require` pentru a importa modulul `graph`. Acesta a fost descris ca exportând funcția `buildGraph`, pe care o puteți prelua din obiectul de interfață cu o declarație `const` și destructurare.

Pentru a exporta `roadGraph` veți adăuga o proprietate la obiectul `exports`. Deoarece `buildGraph` preia o structură de date care nu se potrivește exact cu `roads`, ruperea stringurilor ce descriu drumuri va trebui să fie realizată în modulul vostru.

hint}}

### Dependențe circulare

{{index dependency, "circular dependency", "require function"}}

O dependență circulară este o situație în care modulul A depinde de modulul B, dar modulul B depinde la rîndul său, direct sau indirect de modulul A. Multe sisteme modulare pur și simplu interzic asta deoarece indiferent de ordinea de încărcare pe care o alegeți, nu vă puteți asigura că depepndențele fiecărui modul au fost încărcate înainte de rulare.

Modulele CommonJS permit o formă limitată de dependență circulară. Cât timp modulele nu își înlocuiesc obiectul implicit `exports` și nu își accesează reciproc interfețele decât după ce au fost încărcate, dependențele ciclice sunt permise.

Funcția `require` folosită anterior [în acest capitol](modules#require) permite acest ciclu de dependență. Vă dați seama cum gestionează ciclurile? Ce ar putea funcționa greșit atunci când un modul dintr-un ciclu își modifică obiectul implicit `exports`?

{{hint

{{index overriding, "circular dependency", "exports object"}}

Trucul este că `require` adaugă modulele la propriul cache _înainte_ ca modulul să fie încărcate. Astfel, dacă un apel `require` la rulare încearcă să îl încarce, modulul este deja cunoscut și interfața sa curentă va fi încărcată, în loc să mai fie încărcat încă o dată (ceea ce ar putea duce la depășirea stivei).

Dacă un modul își suprascrie valoarea `module.exports`, orice alt modul care a primit deja interfața sa anterior va avea acces la obiectul implicit de interfață (care este probabil gol) și nu la interfața dorită.

hint}}
