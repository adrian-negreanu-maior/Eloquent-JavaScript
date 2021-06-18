# Document Object Model

{{quote {author: "Friedrich Nietzsche", title: "Beyond Good and Evil", chapter: true}

Păcat! Aceeași poveste! Imediat ce ai terminat construcția casei îți dai seama că ai învățat accidental ceva ce ar fi trebuit să știi înainte de a începe.

quote}}

{{figure {url: "img/chapter_picture_14.jpg", alt: "Picture of a tree with letters and scripts hanging from its branches", chapter: "framed"}}}

{{index drawing, parsing}}

Când deschizi o pagină în browser, browserul solicită codul HTML al paginii și apoi îl parsează, cam așa cum parserul nostru din [capitolul ?](language#parsing) parsa programele. Browserul construiește un model al structurii documentului și utilizează acest model pentru a afișa pagina pe ecran.

{{index "live data structure"}}

Această reprezentare a documentului este una dintre jucăriile la care are acces programul JavaScript din sandbox-ul său. Este o structură de date care poate fi citită și modificată. Funcționează ca o structură de date _live_: când este modificată, pagina de pe ecran este actualizată pentru a reflecta schimbarea.

## Structura documentului

{{index [HTML, structure]}}

Vă puteți imagina un document HTML ca un set de cutii plasate unele în altele. Tagurile cum ar fi `<body>` și `</body>` includ alte taguri, care la rândul lor pot conține taguri sau text. Să reluăm documentul dat ca exemplu în [capitolul anterior](browser):

```{lang: "text/html", sandbox: "homepage"}
<!doctype html>
<html>
  <head>
    <title>My home page</title>
  </head>
  <body>
    <h1>My home page</h1>
    <p>Hello, I am Marijn and this is my home page.</p>
    <p>I also wrote a book! Read it
      <a href="http://eloquentjavascript.net">here</a>.</p>
  </body>
</html>
```

Această pagină are următoarea structură:


{{figure {url: "img/html-boxes.svg", alt: "HTML document as nested boxes", width: "7cm"}}}

{{indexsee "Document Object Model", DOM}}

Structura de date utilizată de browser pentru a reprezenta documentul urmărește forma de mai sus. Pentru fiecare cutie, există un obiect cu care putem interacționa pentru a descopri lucruri cum ar fi - ce tag HTML este reprezentat și ce cutii și text conține. Această reprezentare poartă numele de _Document Object Model_, sau _DOM_, pe scurt.

{{index "documentElement property", "head property", "body property", "html (HTML tag)", "body (HTML tag)", "head (HTML tag)"}}

Bindingul global `document` ne permite accesul la aceste obiecte. Proprietatea sa `documentElement` se referă la obiectul ce reprezintă tagul `<html>`. Deoarece fiecare document HTML are un antet și un corp, obiectul `document` are și proprietățile `head` și `body` care indică aceste elemente.

## Arbori

{{index [nesting, "of objects"]}}

Să revenim puțin la arborii de sintaxă din [capitolul ?](language#parsing). Structura lor este foarte similară cu cea a unui document în browser. Fiecare _nod_ poate să refere alte noduri _descendente (children)_, care pot avea la rândul lor noduri descendente. O asemenea formă este clasică pentru structurile în care elementele pot conține subelemente similare lor.

{{index "documentElement property", [DOM, tree]}}

O structură de date se numește _arbore (tree)_ când are o reprezentare ramificată, nu are cicluri (un nod nu se poate include pe el însuși, direct sau indirect) și are o _rădăcină_ unică și bine definită. În cazul DOM, `document.documentElement` are rolul de rădăcină.

{{index sorting, ["data structure", "tree"], "syntax tree"}}

Arborii sunt frecvent utilizați în computer science. Pe lângă reprezentarea structurilor recursive, cum ar fi documentele HTML sau programele, sunt adesea folosiți pentru a stoca seturi sortate de date, deoarece regăsirea sau inserarea elementelor într-un arbore este mai eficientă decât în cazul utilizării unui array.

{{index "leaf node", "Egg language"}}

Un arbore tipic are tipuri diferite de noduri. Arborele de sintaxa pentru [limbajul Egg](language) are identificatori, valori și aplicații. Nodurile pentru aplicații pot avea descendenți, în timp ce identificatorii și valorile sunt _frunze_ (noduri fără descendenți).

{{index "body property", [HTML, structure]}}

Același lucru este valid și pentru DOM. Nodurile pentru _elemente_, care reprezintă taguri HTML, determină structura documentului. Acestea pot avea descendenți. Un exemplu este `document.body`. Unii dintre acești descendenți pot fi frunze, cum ar fi o bucată de text sau un nod de comentariu.

{{index "text node", element, "ELEMENT_NODE code", "COMMENT_NODE code", "TEXT_NODE code", "nodeType property"}}

Fiecare obiect din DOM are o proprietate `nodeType`, ce conține un cod numeric ce identifică tipul acelui nod. Elementele au codul 1, care este definit și sub forma constantei `Node.ELEMENT_NODE`. Nodurile text, reprezentând o secțiune de text din cadrul documentului, primesc codul 3 (`Node.TEXT_NODE`). Comentariile au codul 8 (`Node.COMMENT_NODE`).

Un alt mod de a vizualiza arborele documentului este cel din figura de mai jos:

{{figure {url: "img/html-tree.svg", alt: "HTML document as a tree",width: "8cm"}}}

Frunzele sunt noduri text iar săgețile indică relația părinte-copil dintre noduri.

{{id standard}}

## Standardul

{{index "programming language", [interface, design], [DOM, interface]}}

Utilizarea unor coduri numerice criptice pentru a reprezenta tipurile nodurilor nu pare a fi aliniată cu stilul JavaScript. Vom vedea mai târziu că și alte părți ale interfeței DOM par ciudate. Motivul este legat de faptul că DOM nu a fost conceput doar pentru JavaScript. Mai degrabă, DOM încearcă să fie o interfață neutră față de limbajele de programare care ar putea fi folosite în alte sisteme - nu doar pentru HTML dar și pentru XML, care este un format generic pentru date, cu o sintaxa asemănătoare cu cea a HTML.

{{index consistency, integration}}

Acest lucru este nefericit. Standardele sunt adesea utile dar în acest caz, avantajul (consistența interfeței) nu este chiar convingător. O interfață integrată corespunzător cu limbajul pe care îl folosiți va economisi mai mult timp decât o interfață familiară în mai multe limbaje.

{{index "array-like object", "NodeList type"}}

Ca și exemplu pentru această integrare sărăcăcioasă, să considerăm proprietatea `childNodes` pe care nodurile pentru elemente din DOM o au. Această proprietate memorează un obiect de tip array, cu o proprietate `length` și proprietățile etichetate cu numere pentru a putea accesa nodurile descendente. Însă aceasta este o instanță a tipului `NodeList`, nu un array propri-zis, așa că nu are metoda cum ar fi `slice` sau `map`.

{{index [interface, design], [DOM, construction], "side effect"}}

Apoi există probleme legate de designul sărăcăcios. De exemplu, nu puteți crea un nod și imemdiat să îi adăugați descendenții sau atributle. Mai întâi va trebui să creați nodul, apoi să adăugați descendenții și atributele, pe rând, utilizând efecte secundare. Codul care interacționează masiv cu DOM tinde să fie lung, repetitiv și urât.

{{index library}}

Dar aceste defecte nu sunt fatale. Deoarece JavaScript ne permite să ne creem propriile abstracții, putem concepe moduri îmbunătățite de a exprima operațiile pe care vrem să le efectuăm. Multe librării pentru programarea în browser vin cu asemenea instrumente.

## Deplasarea prin arbore

{{index pointer}}

Nodurile DOM vin cu o multitudine de legături spre alte noduri. Diagrama de mai jos ilustrează acest lucru:

{{figure {url: "img/html-links.svg", alt: "Links between DOM nodes",width: "6cm"}}}

{{index "child node", "parentNode property", "childNodes property"}}

Deși diagrama prezintă doar o singură legătură de fiecare tip, fiecare nod are o proprietate `parentNode` care indică nodul din care nodul curent face parte. Similar, fiecare element (nod de tip 1) are o proprietate `childNodes` care indică spre un obiect asemănător unui array ce reține descendenții săi direcți.

{{index "firstChild property", "lastChild property", "previousSibling property", "nextSibling property"}}

În teorie, puteți să vă deplasați oriunde în arbore folosind doar aceste legături părinte-copil. Dar JavaScript vă pune la dispoziție și alte legături, pentru comoditate. Proprietățile `firstChild` și `lastChild` indică primul și ultimul element copil sau au valoarea `null` pentru elementele care nu au descendenți. Similar, `previousSibling` și `nextSibling` indică nodurile adiacente, care sunt noduri ce au același părinte cu nodul consultat și apar imediat înainte sau imediat după acest nod. Pentru primul descendent, `previousSibling` va fi `null`, iar pentru ultimul descendent, `nextSibling` va fi `null`.

{{index "children property", "text node", element}}

Avem la dispoziție și proprietatea `children`, similară cu `childNodes`, dar care conține doar descendenții direcți de tip 1 (elemente), nu și alte tipuri de noduri. Această proprietate poate fi utilă dacă nu vă interesează nodurile de tip text.

{{index "talksAbout function", recursion, [nesting, "of objects"]}}

Atunci când aveți de a face cu structuri de date imbricate, funcțiile recursive își dovedesc adesea utilitatea. Funcția de mai jos scanează un document pentru un nod text ce conține un anume string și returnează `true` dacă îl găsește:

{{id talksAbout}}

```{sandbox: "homepage"}
function talksAbout(node, string) {
  if (node.nodeType == Node.ELEMENT_NODE) {
    for (let child of node.childNodes) {
      if (talksAbout(child, string)) {
        return true;
      }
    }
    return false;
  } else if (node.nodeType == Node.TEXT_NODE) {
    return node.nodeValue.indexOf(string) > -1;
  }
}

console.log(talksAbout(document.body, "book"));
// → true
```

{{index "nodeValue property"}}

Proprietatea `nodeValue` a unui nod text memorează stringul ce reprezintă textul respectiv.

## Căutarea elementelor

{{index [DOM, querying], "body property", "hard-coding", [whitespace, "in HTML"]}}

Navigarea legăturilor dintre noduri este adesea utilă. Dar dacă vrem să găsim un anumit nod în document, pornirea din `document.body` și urmarea unei căi fixe de proprietăți este o idee proastă. Procedând astfel, facem presupuneri despre structura exactă a documentului - o structură pe care poate că o vom schimba mai târziu. Un alt motiv pentru care ne complicăm este că se vor crea noduri chiar și pentru spațiile albe dintre noduri. În exemplul de document, tagul `<body>` nu are doar trei copii (`<h1>` și două elemente `<p>`) ci șapte: cele trei, plus spațiile înainte, după și dintre ele.

{{index "search problem", "href attribute", "getElementsByTagName method"}}

Deci, dacă vrem să determinăm atributul `href` al linkului din document, nu vrem să ne exprimăm "Obține cel de-al doilea copil al celui de-al șaselea copil al tagului `<body>`. Ar fi mai bine dacă am putea spune "Găsește primul link din document", ceea ce de altfel putem face.

```{sandbox: "homepage"}
let link = document.body.getElementsByTagName("a")[0];
console.log(link.href);
```

{{index "child node"}}

Toate elementele au o metodă `getElementsByTagName`, care colectează toate elementele descendente (direct sau indirect) ale respectivului nod, care potrivesc numele dat pentru tag și le returnează într-o structură asemănătoare unui array.

{{index "id attribute", "getElementById method"}}

Pentru a identifica un _singur_ nod, îi puteți preciza un atribut `id` și apoi să folosiți metoda `document.getElementById`.

```{lang: "text/html"}
<p>My ostrich Gertrude:</p>
<p><img id="gertrude" src="img/ostrich.png"></p>

<script>
  let ostrich = document.getElementById("gertrude");
  console.log(ostrich.src);
</script>
```

{{index "getElementsByClassName method", "class attribute"}}

O a treia metodă similară este `getElementsByClassName` care, ca și `getElementsByTagName`, caută prin conținutul unui element de tip nod și returnează toate elementele care au stringul dat ca parametru în atributul `class`.

## Modificarea documentului

{{index "side effect", "removeChild method", "appendChild method", "insertBefore method", [DOM, construction], [DOM, modification]}}

Putem modifica aproape orice în structura de date DOM. Forma arborelui documentului poate fi modificată prin modificarea relațiilor de legătură părinte-copil. Nodurile au o metodă `remove` prin care le putem elimina din părintele curent. Pentru a adăuga un copil la un nod de tip element, putem utiliza `appendChild`, care plasează nodul la sfârșitul istei de copii a nodului, sau `insertBefore` care inserează nodul dat ca prim argument înaintea nodului reprezentat de al doilea argument.

```{lang: "text/html"}
<p>One</p>
<p>Two</p>
<p>Three</p>

<script>
  let paragraphs = document.body.getElementsByTagName("p");
  document.body.insertBefore(paragraphs[2], paragraphs[0]);
</script>
```

Un nod poate exista în document într-un singur loc. Prin urmare, inserarea paragrafului _Three_ în fața paragrafului _One_ va elimina mai întâi paragraful de la sfârșitul documentului și apoi îl va insera la început. Toate operațiile care inserează un nod existent undeva în document vor cauza, ca și efect secundar, eliminarea nodului din poziția curentă.

{{index "insertBefore method", "replaceChild method"}}

Metoda `replaceChild` poate fi utilizată pentru a înlocui un nod cu altul. Ea primește ca argumente două noduri: un nod nou și nodul ce urmează a fi înlocuit. Nodul ce urmează a fi înlocuit trebuie să fie copil al elementului pentru care a fost apelată metoda. Atât `replaceChild` cât și `insertBefore` așteaptă _noul_ nod ca și prim argument.

## Crearea nodurilor

{{index "alt attribute", "img (HTML tag)"}}

Să presupunem că vrem să scriem un script care înlocuiește toate imaginile (taguri `<img>`) din document cu textul memorat în atributul lor alt, care reprezintă reprezentarea textuală alternativă a imaginii.

{{index "createTextNode method"}}

Aceasta presupune nu doar eliminarea imaginii ci și înlocuirea ei cu un nod nou de tip text. Nodurile de tip text pot fi create cu metoda `document.createTextNode`.

```{lang: "text/html"}
<p>The <img src="img/cat.png" alt="Cat"> in the
  <img src="img/hat.png" alt="Hat">.</p>

<p><button onclick="replaceImages()">Replace</button></p>

<script>
  function replaceImages() {
    let images = document.body.getElementsByTagName("img");
    for (let i = images.length - 1; i >= 0; i--) {
      let image = images[i];
      if (image.alt) {
        let text = document.createTextNode(image.alt);
        image.parentNode.replaceChild(text, image);
      }
    }
  }
</script>
```

{{index "text node"}}

Pentru un string dat, `createTextNode` ne returnează un nod text pe care îl putem insera în document pentru a îl face să apară pe ecran.

{{index "live data structure", "getElementsByTagName method", "childNodes property"}}

Bucla care parcurge imaginile începe de la sfârșitul listei. Este necesar să procedăm astfel deoarce lista de noduri returnată de metoda cum ar fi `getElementsByTagName` (sau o proprietate cum ar fi `childNodes`) este _live_. Adică, va fi actualizată la schimbarea documentului. Dacă porneam de la început, eliminarea primei imagini ar fi produs eliminarea din listă a primului element, astfel încât a doua iterație din buclă, când `i` este 1, nu ar fi fost executată pentru că am fi avut o colecție de lungime 1.

{{index "slice method"}}

Dacă preferați o colecție de noduri _solidă_ în locul uneia _live_, puteți converti colecția într-un array real, folosind un apel către `Array.from`.

```
let arrayish = {0: "one", 1: "two", length: 2};
let array = Array.from(arrayish);
console.log(array.map(s => s.toUpperCase()));
// → ["ONE", "TWO"]
```

{{index "createElement method"}}

Pentru a crea noduri de tip element, puteți utiliza metoda `document.createElement`. Această metodă primește numele unui tag și returnează un nod gol de tip element.

{{index "Popper, Karl", [DOM, construction], "elt function"}}

{{id elt}}

Exemplul care urmează definește o funcție `elt`, care crează un nod de tip element și tratează restul argumentelor ca și copii ai acestui nod. Apoi utilizăm aceasă funcție pentru a atribui autorului un citat.

```{lang: "text/html"}
<blockquote id="quote">
  No book can ever be finished. While working on it we learn
  just enough to find it immature the moment we turn away
  from it.
</blockquote>

<script>
  function elt(type, ...children) {
    let node = document.createElement(type);
    for (let child of children) {
      if (typeof child != "string") node.appendChild(child);
      else node.appendChild(document.createTextNode(child));
    }
    return node;
  }

  document.getElementById("quote").appendChild(
    elt("footer", "—",
        elt("strong", "Karl Popper"),
        ", preface to the second edition of ",
        elt("em", "The Open Society and Its Enemies"),
        ", 1950"));
</script>
```

{{if book

Documentul rezultat arată astfel:

{{figure {url: "img/blockquote.png", alt: "A blockquote with attribution",width: "8cm"}}}

if}}

## Atribute

{{index "href attribute", [DOM, attributes]}}

Atributele unor elemente, cum ar fi `href` pentru hiperlegături, pot fi accesate printr-o proprietate cu același nume a obiectului din DOM corespunzător elementului. Acesta este cazul pentru majoritatea atributelor standard frecvent utilizate.

{{index "data attribute", "getAttribute method", "setAttribute method", attribute}}

Dar HTML ne permite să setăm orice atribute dorim awsupra nodurilor. Ceea ce este util deoarece ne permită să stocăm informații suplimentare în document. Dacă vă creați propriile nume pentru atribute, acestea nu vor fi disponibile ca și proprietăți în nodul elementului. Prin urmare, va trebui să folosiți metodele `getAttribute` și `setAttribute` pentru a le putea utiliza.

```{lang: "text/html"}
<p data-classified="secret">The launch code is 00000000.</p>
<p data-classified="unclassified">I have two feet.</p>

<script>
  let paras = document.body.getElementsByTagName("p");
  for (let para of Array.from(paras)) {
    if (para.getAttribute("data-classified") == "secret") {
      para.remove();
    }
  }
</script>
```

O practică recomandată este utilizarea prefixului `data-` pentru asemenea atribute, pentru a vă asigura că nu intrați în conflict cu alte atribute.

{{index "getAttribute method", "setAttribute method", "className property", "class attribute"}}

Un atribut utilizat frecvent, `class`, este de asemenea un cuvânt-cheie în limbajul JavaScript. Din motive istorice - unele implementări mai vechi ale JavaScript nu pot gestiona numele proprietăților care sunt și cuvinte-cheie. Proprietatea utilizată pentru a accesa acest atribut se numește `className`. Dar o puteți accesa și cu numele său real, `"class"`, dacă utilizați metodele `getAttribute` și `setAttribute`.

## Layout - aspectul

{{index layout, "block element", "inline element", "p (HTML tag)", "h1 (HTML tag)", "a (HTML tag)", "strong (HTML tag)"}}

Probabil ați remarcat că diferitele tipuri de elemente sunt poziționate diferit. Unele, cum ar fi paragrafele (`<p>`) sau titlurile (`<h1>`), ocupă toată lățimea documentului și sunt redate pe linii separate. Acestea sunt elemente de tip _bloc_. Altele, cum ar fi legăturile (`<a>`) sau marcajul `<strong>`, sunt redate pe aceeași linie cu textul în care sunt plasate. Asemenea elemente sunt elemente _inline_.

{{index drawing}}

Pentru orice document dat, browserele pot calcula un aspect, care dă fiecărui element o dimensiune și o poziție pe baza tipului său și a conținutului. Apoi aceste calcule sunt utilizate pentru a desena documentul pe ecran.

{{index "border (CSS)", "offsetWidth property", "offsetHeight property", "clientWidth property", "clientHeight property", dimensions}}

Dimensiunea și poziția unui element pott fi accesate din JavaScript. Proprietățile `offsetWidth` și `offsetHeight` returnează dimensiunile unui element în _pixeli_. Pixelul este unitatea de măsură în browser. Tradițional, el corespunde celui mai mic punct pe care un ecran îl poate afișa, dar pe ecranele moderne, care pot afișa puncte foarte mici, situația ar putea să fie modificată și un pixel în browser să reprezinte mai multe puncte de pe ecran.

Similar, `clientWidth` și `clientHeight` vă returnează dimensiunea spațiului din interiorul elementului, ignorând lățimea bordurii elementului.

```{lang: "text/html"}
<p style="border: 3px solid red">
  I'm boxed in
</p>

<script>
  let para = document.body.getElementsByTagName("p")[0];
  console.log("clientHeight:", para.clientHeight);
  console.log("offsetHeight:", para.offsetHeight);
</script>
```

{{if book

Dacă pentru un paragraf setăm proprietatea `border` (în atributul `style` de exemplu), va fi desenat un dreptunghi în jurul lui.

{{figure {url: "img/boxed-in.png", alt: "A paragraph with a border",width: "8cm"}}}

if}}

{{index "getBoundingClientRect method", position, "pageXOffset property", "pageYOffset property"}}

{{id boundingRect}}

Cel mai eficient mod de a determina poziția exactă a unui element pe ecran este apelarea metodei `getBoundingClientRect`.
Aceasta returnează proprietățile `top`, `bottom`, `left` și `right`, relativ la colțul stânga sus al ecranului. Dacă vreți să le determinați relativ la întregul document, trebuie să țineți cont de poziția curentă de scroll, pe care o puteți determina din bindigurile `pageXOffset` și `pageYOffset`.

{{index "offsetHeight property", "getBoundingClientRect method", drawing, laziness, performance, efficiency}}

Punerea în pagină a unui document poate presupune un efort considerabil. Pentru rapidizare, motoarele browserelor nu reașează documentul în pagină de fiecare dată când îl modificați ci așteaptă cât de mult pot. Când un program JavaScript care a schimbat documentul își încheie execuția, browserul va trebui să calculeze un nou aspect pentru a desena documentul modificat pe ecran. Când un program _cere_ poziția sau dimensiunea unui element prin citirea unor proprietăți cum ar fi `offsetHeight` sau apelul metodei `getBoundingClientRect`, returnarea informațiilor corecte presupune de asemenea recalcularea aspectului.

{{index "side effect", optimization, benchmark}}

Un program care alternează în mod repetat citirea informațiilor despre aspectul DOM cu modificarea DOM implică o mulțime de calcule și, prin urmare, va rula foarte încet. Codul de mai jos este un exemplu. Acesta conține două programe diferite care construiesc o linie de caractere _X_ cu lățimea de 2000 pixeli și măsoară durata fiecărei construcții.

```{lang: "text/html", test: nonumbers}
<p><span id="one"></span></p>
<p><span id="two"></span></p>

<script>
  function time(name, action) {
    let start = Date.now(); // Current time in milliseconds
    action();
    console.log(name, "took", Date.now() - start, "ms");
  }

  time("naive", () => {
    let target = document.getElementById("one");
    while (target.offsetWidth < 2000) {
      target.appendChild(document.createTextNode("X"));
    }
  });
  // → naive took 32 ms

  time("clever", function() {
    let target = document.getElementById("two");
    target.appendChild(document.createTextNode("XXXXX"));
    let total = Math.ceil(2000 / (target.offsetWidth / 5));
    target.firstChild.nodeValue = "X".repeat(total);
  });
  // → clever took 1 ms
</script>
```

## Stilizarea

{{index "block element", "inline element", style, "strong (HTML tag)", "a (HTML tag)", underline}}

Am văzut că diferitele elemente HTML sunt desenate diferit. Unele sunt afișate ca blocuri, altele inline. Unele au stilizare suplimentară - `<strong>` va avea conținutul _bold_, iar `<a>` îl va avea albastru și subliniat.

{{index "img (HTML tag)", "default behavior", "style attribute"}}

Modul în care este afișat tagul `<img>` sau `<a>` este direct legat de tipul elementului. Dar putem modifica stilizarea asociată cu un element. Iată un exemplu de utilizare a proprietății `style`:

```{lang: "text/html"}
<p><a href=".">Normal link</a></p>
<p><a href="." style="color: green">Green link</a></p>
```

{{if book

Cea de a doua legătură va fi afișată cu culoarea verde în locul culorii implicite.

{{figure {url: "img/colored-links.png", alt: "A normal and a green link",width: "2.2cm"}}}

if}}

{{index "border (CSS)", "color (CSS)", CSS, "colon character"}}

Atributul `style` poate conține una sau mai mutle _declarații_, fiecare sub forma `proprietate: valoare` (`color: green`). Atunci când vrem să precizăm mai multe asemenea declarații, le separăm prin `;`, ca și în `"color: red; border: none"`.

{{index "display (CSS)", layout}}

Multe aspecte ale documentului pot fi influențate prin stilizare. De exemplu, proprietatea `display` controlează dacă un element este afișat sub formă de bloc sau inline.

```{lang: "text/html"}
This text is displayed <strong>inline</strong>,
<strong style="display: block">as a block</strong>, and
<strong style="display: none">not at all</strong>.
```

{{index "hidden element"}}

Elementul afișat ca și bloc va fi afișat pe propria sa linie, nu aliniat cu textul care îl înconjoară. Iar un element pentru care setăm `display: none` nu va fi afișat deloc pe ecran. Astfel putem ascunde elemente, ceea ce este adesea preferabil în loc să le eliminăm, deorece le putem afișa ulterior cu ușurință.

{{if book

{{figure {url: "img/display.png", alt: "Different display styles",width: "4cm"}}}

if}}

{{index "color (CSS)", "style attribute"}}

Codul JavaScript poate manipula direct stilul unui element prin proprietatea `style` a elementului respectiv. Această proprietate este un obiect care are proprietăți definite pentru toate regulile de stilizare posibile. Valorile acestor proprietăți sunt stringuri, pe care le putem seta pentru a modifica anumite aspecte ale unui element.

```{lang: "text/html"}
<p id="para" style="color: purple">
  Nice text
</p>

<script>
  let para = document.getElementById("para");
  console.log(para.style.color);
  para.style.color = "magenta";
</script>
```

{{index "camel case", capitalization, "hyphen character", "font-family (CSS)"}}

Unele nume de proprietăți conțin caractere nepermise în obiecte, cum ar fi `font-family`. Din această cauză, în JavaScript suntem obligați să folosim sintaxa cu paranteze drepte(`style["font-family"]`), dar numele proprietăților în obiectul `style` sunt modificate în scrierea "camelCase" (`style.fontFamily`).

## Cascadarea stilurilor

{{index "rule (CSS)", "style (HTML tag)"}}

{{indexsee "Cascading Style Sheets", CSS}}
{{indexsee "style sheet", CSS}}

Sistemul de stilizare pentru HTML se numește CSS, de la _Cascading Style Sheets_. O _foaie de stil_ este un set de reguli pentru stilizarea elementelor dintr-un document. Putem preciza regulile în interiorul unui tag `<style>`.

```{lang: "text/html"}
<style>
  strong {
    font-style: italic;
    color: gray;
  }
</style>
<p>Now <strong>strong text</strong> is italic and gray.</p>
```

{{index "rule (CSS)", "font-weight (CSS)", overlay}}

_Cascadarea_ se referă la faptul că mai multe asemenea reguli sunt combinate pentru a produce stilul final pentru un element. În exemplu, stilizarea implicită pentru taguri `<strong>`, care le aplică `font-weight: bold`, este suprapusă cu regula din tagul `<style>`, care adaugă `font-style` și `color`.

{{index "style (HTML tag)", "style attribute"}}

Când mai multe reguli definesc o valoare pentru aceeași proprietate, cea mai recent citită primește o prioritate mai mare și câștigă. Astfel, dacă regula din tag-ul `<style>` ar fi inclus `font-weight: normal`, ce contrazice regula implicită `font-weight`, textul s-ar fi afișat normal, _nu_ bold. Stilurile aplicate prin atributul `style`, direct asupra nodului au cea mai mare prioritate și întotdeauna câștigă.

{{index uniqueness, "class attribute", "id attribute"}}

Este posibil să selectăm în regulile CSS și altfel elementele. O regulă pentru `.abc` se aplică tuturor elementelor care au `"abc"` în atributul pentru `class`. O regulă pentru `#xyz` se aplică elementului ce are atributul `id` cu valoarea `"xyz"` (care ar trebui să fie unic în document).

```{lang: "text/css"}
.subtle {
  color: gray;
  font-size: 80%;
}
#header {
  background: blue;
  color: white;
}
/* p elements with id main and with classes a and b */
p#main.a.b {
  margin-bottom: 20px;
}
```

{{index "rule (CSS)"}}

Regula de precedență ce favorizează cea mai recentă regulă CSS se aplică doar atunci când regulile CSS au aceeași _specificitate_. Specificitatea unei reguli CSS se referă la precizia cu care aceasta descrie elementele ce se potrivesc, determinată de număr și tip (tag, clasă sau id) al aspectelor pe care le conține în selector. De exemplu, o regulă care selectează `p.a` este mai specifică decât una care selectează `p` sau `.a` și va avea prioritate.

{{index "direct child node"}}

Notația `p > a {…}` aplică regulile de stilizare tagurilor `<a>` care sunt descendenți direcți ai unui tag `<p>`. Iar `p a {…}` se aplică tuturor tagurilor `<a>` aflate în interiorul unui tag `<p>`, indiferent dacă acestea sunt descendenți direcți sau indirecți.

## Query selectors

{{index complexity, CSS}}

Nu vom utiliza pre mult foile de stil în această carte. Înțelegerea lor este utilă când programați în browser, dar sunt destul de complicate pentru a face subiectul unei alte cărți.

{{index "domain-specific language", [DOM, querying]}}

Motivul principal pentru care am introdus sintaxa pentru _selectori_ - notația utilizată în foile de stil pentru a determina căror elemente li se aplică un anume set de reguli - este că putem utiliza același mini-limbaj ca și o modalitate eficientă de a găsi elemente în DOM.

{{index "querySelectorAll method", "NodeList type"}}

Metoda `querySelectorAll`, care este definită atât pentru obiectul `document` cât și pentru nodurile de tip element, primește ca parametru un string selector și returnează un `NodeList` ce conține toate elementele potrivite.

```{lang: "text/html"}
<p>And if you go chasing
  <span class="animal">rabbits</span></p>
<p>And you know you're going to fall</p>
<p>Tell 'em a <span class="character">hookah smoking
  <span class="animal">caterpillar</span></span></p>
<p>Has given you the call</p>

<script>
  function count(selector) {
    return document.querySelectorAll(selector).length;
  }
  console.log(count("p"));           // All <p> elements
  // → 4
  console.log(count(".animal"));     // Class animal
  // → 2
  console.log(count("p .animal"));   // Animal inside of <p>
  // → 2
  console.log(count("p > .animal")); // Direct child of <p>
  // → 1
</script>
```

{{index "live data structure"}}

Spre deosebire de metode cum ar fi `getElementsByTagName`, obiectul returnat de către `querySelectorAll` _nu_ este live. Nu se va modifica odată cu modificarea documentului. Dar nu este nici un array și va trebui să apelați `Array.from` dacă vreți să îl tratați ca pe un array.

{{index "querySelector method"}}

Metoda `querySelector` (fără `All`) funcționează similar. Aceasta este utilă dacă aveți de căutat un singur element, specific. Ea va returna doar primul element pe care îl potrivește sau` null` în cazul în care nu găsește nici o potrivire.

{{id animation}}

## Poziționarea și animarea

{{index "position (CSS)", "relative positioning", "top (CSS)", "left (CSS)", "absolute positioning"}}

Proprietatea de stil `position` influențează mult aspectul. Implicit, ea are valoarea `static`, însemnând că elementul se află în poziția sa normală în cadrul documentului. Dacă este setată la valoarea `relative`, elementul va continua să ocupe spațiu în document, dar acum vom putea utiliza proprietățile `top` și `left` pentru a îl muta relativ la poziția sa normală. Când `position` se setează ca `absolute`, elementul este eliminat din fluxul normal al documentului - adică nu mai ocupă spațiu și poate fi suprapus peste alte elemente. Proprietățile `top` și `left` vor putea fi acum folosite pentru a poziționa absolut elementul relativ la colțul stânga-sus al celui mai apropiat părinte a cărui proprietate `position` nu are valoarea static sau relativ la colțul stânga-sus al documentului, în cazul în care un asemenea element nu există.

{{index [animation, "spinning cat"]}}

Putem utiliza acest comportament pentru a crea o animație. Documentul de mai jos afișează imaginea unei pisici care se mișcă pe o traiectorie eliptică:


```{lang: "text/html", startCode: true}
<p style="text-align: center">
  <img src="img/cat.png" style="position: relative">
</p>
<script>
  let cat = document.querySelector("img");
  let angle = Math.PI / 2;
  function animate(time, lastTime) {
    if (lastTime != null) {
      angle += (time - lastTime) * 0.001;
    }
    cat.style.top = (Math.sin(angle) * 20) + "px";
    cat.style.left = (Math.cos(angle) * 200) + "px";
    requestAnimationFrame(newTime => animate(newTime, time));
  }
  requestAnimationFrame(animate);
</script>
```

{{if book

Săgeata gri arată traiectoria imaginii.

{{figure {url: "img/cat-animation.png", alt: "A moving cat head",width: "8cm"}}}

if}}

{{index "top (CSS)", "left (CSS)", centering, "relative positioning"}}

Imaginea noastră este centrată în pagină și are `position` setat la `relative`. Actualizăm repetat proprietățile `top` și `left` ale imaginii pentru a o mișca.

{{index "requestAnimationFrame function", drawing, animation}}

{{id animationFrame}}

Scriptul folosește `requestAnimationFrame` pentru a programa execuția funcției `animate` atunci când browserul este pregătit să redeseneze ecranul. Funcția `animate` va apela din nou `requestAnimationFrame` pentru a programa următoarea actualizare. Când fereastra sau tabul browserului sunt active, actualizările se vor realiza cu o rată de aproximativ 60 cadre pe secundă, ceea ce va produce o animație bună.

{{index timeline, blocking}}

Dacă doar actualizăm DOM-ul într-o buclă, pagina va îngheța și nu se va afișa nimic pe ecran. Browserele nu își actualizează ecranul cât timp există un program JavaScript care rulează și nici nu permit interacțiunea cu pagina. De aceea avem nevoie de `requestAnimationFrame` - aceasta informează browserul că suntem gata și poate să facă ceea ce are de făcut, inclusiv actualizarea ecranului și răspunsul la acțiunile utilizatorului.

{{index "smooth animation"}}

Funcției de animație i se transmite timpul curent ca și argument. Pentru a ne asigura că mișcarea imaginii este stabilă, viteza de modificare a unghiului se calculează pe baza diferenței dintre timpul curent și momentul de timp la care funcția a fost executată pentru ultima dată. Dacă am fi ales să modificăm unghiul cu o valoare fixă pe fiecare pas, mișcarea ar fi sacadat dacă, de exemplu, o altă sarcină intensivă ar fi blocat execuția funcției de animare pentru o fracțiune de secundă.

{{index "Math.cos function", "Math.sin function", cosine, sine, trigonometry}}

{{id sin_cos}}

Mișcarea pe un cerc este realizată cu funcțiile trigonometrice `Math.cos` și `Math.sin`. Pentru cei care nu sunt familiari, le vom discuta pe scurt, deoarece le folosim ocazional în această carte.

{{index coordinates, pi}}

`Math.cos` și `Math.sin` sunt utile pentru a găsi puncte pe un cerc cu centrul în punctul (0,0) și raza 1. Ambele funcții interpretează argumentul ca fiind poziția pe acest cerc, 0 reprezentând punctul cel mai din dreapta al cercului pe direcția orizontală și parcurgearea în sensul acelor de ceas până la 2π (cam 6.28) ne permite să parcurgem întreg cercul. `Math.cos` ne returnează coordonata x a poziției date iar `Math.sin` ne dă coordonata y. Pozițiile sau unghiurile mai mari decât 2π sau mai mici decât zero sunt valide - rotația se repetă astfel încât _a_+2π referă acelați unghi ca și _a_.

{{index "PI constant"}}

Această unitate de măsură pentru unghiuri se numește _radian_ - un cerc complet reprezintă 2π radiani, sau 360 de grade. Constanta π este disponibilă ca `Math.PI` în JavaScript.

{{figure {url: "img/cos_sin.svg", alt: "Using cosine and sine to compute coordinates",width: "6cm"}}}

{{index "counter variable", "Math.sin function", "top (CSS)", "Math.cos function", "left (CSS)", ellipse}}

Codul animației menține un contor, `angle` pentru unghiul curent al animației și îl incrementează la fiecare apel al funcției `animate`. Apoi utilizează acest unghi pentru a calcula poziția curentă a elementului. Proprietatea `top` este calculată folosinf `Math.sin` și multiplicând cu 20 (raza verticală a elipsei noastre). Proprietatea `left` se bazează pe `Math.cos` și folosește o multiplicare cu 200, astfel că elipsa este mult aplatizată pe verticală.

{{index "unit (CSS)"}}

Propreitățile au nevoie de _unități_. În acest caz am adăugat `"px"` numărului pentru a informa browserul că am calculat în pixeli (puteam folosi și alte unități utilizate în CSS). E ușor să uitați să specificați unitățile. Dar dacă utilizați numere fără unități, regula de stil va fi ignorată - cu excepția cazului în care numărul este 0, ce are aceeași semnificație indiferent de unitatea folosită.

## Rezumat

Programele JavaScript pot inspecta și interfera cu documentul afișat de către browser printr-o structură de date numită DOM. Această structură reprezintă modelul în browser al documentului și programul JavaScript o poate modifica pentru a schimba aspectul documentului vizibil.

DOM este organizat ca un arbore, în care elementele sunt aranjate ierarhic, conform structurii documentului. Obiectele ce reprezintă elemente au proprietăți cum ar fi `parentNode` și `childNodes`, care pot fi utilizate pentru a naviga prin acest arbore.

Modul în care este afișat un document poate fi influențat prin _stilizare_, atât prin atașarea unor stiluri direct nodurilor cât și prin definirea unor reguli care potrivesc anumite noduri. Există multe proprietăți de stil, cum ar fi `color` sau `display`. Codul JavaScript poate manipula stilul unui element direct prin proprietatea `style` a acestuia.

## Exerciții

{{id exercise_table}}

### Construirea unui tabel

{{index "table (HTML tag)"}}

Un tabel HTML este construit cu următoarea structură de taguri:


```{lang: "text/html"}
<table>
  <tr>
    <th>name</th>
    <th>height</th>
    <th>place</th>
  </tr>
  <tr>
    <td>Kilimanjaro</td>
    <td>5895</td>
    <td>Tanzania</td>
  </tr>
</table>
```

{{index "tr (HTML tag)", "th (HTML tag)", "td (HTML tag)"}}

Pentru fiecare _rând_, tagul `<table>` conține un tag `<tr>`. În interiorul tagurilor `<tr>` putem insera celule: fie antete (`<th>`) fie celule obișnuite (`<td>`).

Fiind dat un set de munți, un array de obiecte cu proprietățile `name`, `height` și `place`, generați structura DOM a unui tabel ce enumeră obiectele. Acesta trebuie să conțină câte o coloană pentru fiecare cheie și câte un rând pentru fiecare obiect, precum și un rând de antet cu elemente `<th>` ce conțin ca și text numele coloanelor.

Scrieți codul astfel încât coloanele sunt deduse automat din structura obiectelor. Pentru aceasta, folosiți numele proprietăților din primul obiect din setul de date.

Adăugați tabelul rezultat la elementul al cărui atribut `id` are valoarea `"mountains"` astfel încât acesta să fie vizibil în document.

{{index "right-aligning", "text-align (CSS)"}}

După ce ați reușit, aliniați la dreapta celulele care conțin valori numerice prin setarea proprietății lor `style.textAlign` la valoarea `"right"`.

{{if interactive

```{test: no, lang: "text/html"}
<h1>Mountains</h1>

<div id="mountains"></div>

<script>
  const MOUNTAINS = [
    {name: "Kilimanjaro", height: 5895, place: "Tanzania"},
    {name: "Everest", height: 8848, place: "Nepal"},
    {name: "Mount Fuji", height: 3776, place: "Japan"},
    {name: "Vaalserberg", height: 323, place: "Netherlands"},
    {name: "Denali", height: 6168, place: "United States"},
    {name: "Popocatepetl", height: 5465, place: "Mexico"},
    {name: "Mont Blanc", height: 4808, place: "Italy/France"}
  ];

  // Your code here
</script>
```

if}}

{{hint

{{index "createElement method", "table example", "appendChild method"}}

Puteți utiliza `document.createElement` pentru a crea noi noduri pentru elemente și `document.createTextNode` pentru a crea noduri text, apoi cu metoda `appendChild` veți putea plasa nodurile în interiorul altor noduri.

{{index "Object.keys function"}}

Veți itera numele cheilor pentru a completa rândul de antet și apoi, pentru fiecare obiect din array veți construi un rând ân tabel. Pentru a obține numele cheilor din primul obiect, puteți folosi `Object.keys`.

{{index "getElementById method", "querySelector method"}}

Pentru a putea adăuga tabelul la părintele corect puteți utiliza `document.getElementById` sau `document.querySelector` pentru a găsi nodul cu atributul `id` menționat.

hint}}

### Elemente după numele tag-ului

{{index "getElementsByTagName method", recursion}}

Metoda `document.getElementsByTagName` returnează toate elementele descendente cu un tag dat. Implementați propria versiune a acestei funcții care primește un nod și un string (numele tagului) și returnează un array ce conține toți descendenții nodului cu numele de tag dat.

{{index "nodeName property", capitalization, "toLowerCase method", "toUpperCase method"}}

Pentru a determina numele tagului unui element folosiți proprietatea `nodeName`. Dar atenție că aceasta va returna numele tagului cu litere mari. Utilizați metodele `toLowerCase` sau `toUpperCase` pentru stringuri ca să compensați.

{{if interactive

```{lang: "text/html", test: no}
<h1>Heading with a <span>span</span> element.</h1>
<p>A paragraph with <span>one</span>, <span>two</span>
  spans.</p>

<script>
  function byTagName(node, tagName) {
    // Your code here.
  }

  console.log(byTagName(document.body, "h1").length);
  // → 1
  console.log(byTagName(document.body, "span").length);
  // → 3
  let para = document.querySelector("p");
  console.log(byTagName(para, "span").length);
  // → 2
</script>
```
if}}

{{hint

{{index "getElementsByTagName method", recursion}}

Soluția poate fi exprimată cel mai ușor cu o funcție recursivă, similar cu [funcția `talksAbout`](dom#talksAbout) definită anterior în acest capitol.

{{index concatenation, "concat method", closure}}

Puteți apela `byTagname` recursiv și să concatenați array-urile rezultate pentru a produce rezultatul final. Sau ați putea crea o funcție internă care se apelează recursiv și care are acces la un binding de tip array definit în funcția exterioară, la care va putea adăuga elementele pe care le găsește. Nu uitați că funcția interioară trebuie apelată din funcția exterioară pentru a începe procesul.

{{index "nodeType property", "ELEMENT_NODE code"}}

Funcția recursivă trebuie să verifice tipul nodului. Ne interesează doar nodurile de tip 1 (`Node.ELEMENT_NODE`). Pentru asemenea noduri, trebuie să iterăm copiii și, pentru fiecare copil, să verificăm dacă potrivește interogarea dar și un apel recursiv pentru a verifica descendenții fiecărui nod.

hint}}

### Pălăria pisicii

{{index "cat's hat (exercise)", [animation, "spinning cat"]}}

Extindeți animația definită [anterior](dom#animation) astfel încât atât pisica cât și pălăria ei (`<img src="img/hat.png">`) orbitează pe elipsă.

Sau pălăria să se miște în jurul pisicii. Sau modificați animația într-un alt mod interesant.

{{index "absolute positioning", "top (CSS)", "left (CSS)", "position (CSS)"}}

Pentru a poziționa mai multe obiecte mai ușor, probail e o idee bună să utilizați poziționarea absolută. Aceasta înseamnă că proprietățile `top` și `left` sunt calculate relativ la colțul stânga-sus al documentului. Pentru a evita utilizarea unor coordonate negative, puteți adăuga un număr fix de pixeli la valorile pentru poziție.

{{if interactive

```{lang: "text/html", test: no}
<style>body { min-height: 200px }</style>
<img src="img/cat.png" id="cat" style="position: absolute">
<img src="img/hat.png" id="hat" style="position: absolute">

<script>
  let cat = document.querySelector("#cat");
  let hat = document.querySelector("#hat");

  let angle = 0;
  let lastTime = null;
  function animate(time) {
    if (lastTime != null) angle += (time - lastTime) * 0.001;
    lastTime = time;
    cat.style.top = (Math.sin(angle) * 40 + 40) + "px";
    cat.style.left = (Math.cos(angle) * 200 + 230) + "px";

    // Your extensions here.

    requestAnimationFrame(animate);
  }
  requestAnimationFrame(animate);
</script>
```

if}}

{{hint

`Math.cos` și `Math.sin` folosesc unghiurile în radiani, un cerc complet are 2π radiani. Pentru un unghi dat, puteți determina unghiul opus adăugand jumătate din această valoare, ceea ce este `Math.PI`. Aceasta vă ajută ca să plasați pălăria pe partea opusă a orbitei.

hint}}
