{{meta {load_files: ["code/chapter/16_game.js", "code/levels.js"], zip: "html include=[\"css/game.css\"]"}}}

# Proiect: Un joc de platformă

{{quote {author: "Iain Banks", title: "The Player of Games", chapter: true}

Toată realitatea este un joc.

quote}}

{{index "Banks, Ian", "project chapter", simulation}}

{{figure {url: "img/chapter_picture_16.jpg", alt: "Picture of a game character jumping over lava", chapter: "framed"}}}

Mare parte din fascinația mea inițială relativ la computere, ca și a multor altor copii tocilari, era legată de jocurile pe computer. Am fost atras de micile lumi simulate pe care le puteam manipula și în care poveștile se desfășurau, mai degrabă datorită modului în care îmi proiectam imaginația asupra lor decât datorită posibilităților pe care le ofereau.

Nu-i doresc nimănui o carieră în programarea jocurilor. Ca și în industria muzicii, discrepanța dintre numărul mare de tineri care doresc să intre în acest domeniu și cererea reală pentru asemenea persoane crează un mediu nesănătos. Dar scrierea jocurilor pentru distracție este amuzantă.

{{index "jump-and-run game", dimensions}}

Acest capitol vă va introduce în implementarea unui mic joc de platformă. Jocurile de platformă (sau "jump and run") sunt jocuri în care jucătorul deplasează un personaj printr-o lume, de regulă bidimensională și privită din lateral, personajul făcând salturi pe și peste lucruri.

## Jocul

{{index minimalism, "Palef, Thomas", "Dark Blue (game)"}}

Jocul nostru se va baza pe [Dark Blue](http://www.lessmilk.com/games/10)[(_www.lessmilk.com/games/10_)]{if book} by Thomas Palef. Am ales acest joc deoarece este angajant și minimalist și poate fi construit fără a scrie prea mult cod. Jocul arată cam așa:

{{figure {url: "img/darkblue.png", alt: "The game Dark Blue"}}}

{{index coin, lava}}

Cutia neagră reprezintă jucătorul, a cărui sarcină este de a colecta cutiile galbene (monede), evitând zonele roții (lava). Un nivel este completat când au fost colectate toate monedele.

{{index keyboard, jumping}}

Jucătorul se poate mișca în scenă din tastele săgeți stânga-dreapta și poate sări cu tasta săgeată sus. Saltul este o specialitate a personajului din acest joc. Personajul poate sări de câteva ori înălțimea sa și poate să își schimbe direcția în aer. Acesta nu este un comportament foarte realist, dar dă jucătorului senzația de control complet asupra personajului de pe ecran.

{{index "fractional number", discretization, "artificial life", "electronic life"}}

Jocul constă dintr-un fundal static, afișat ca un grid, elementele în mișcare fiind plasate peste acest fundal. Fiecare câmp al gridului este fie gol, fie solid, fie lava. Elementele în mișcare sunt personajul, monedele și anumite bucăți de lavă. Poziția acestor elemente nu este constrânsă de grid - coordonatele lor pot să fie fracționare, permițând o mișcare lină.

## Tehnologia

{{index "event handling", keyboard, [DOM, graphics]}}

Vom utiliza DOM-ul browserului pentru a afișa jocul și vom citi intrarea de la utilizator prin gestiunea evenimentelor de tastatură.

{{index rectangle, "background (CSS)", "position (CSS)", graphics}}

Codul legat de ecran și tastatură este doar o mică parte din ceea ce avem de făcut pentru a construi acest joc. Deoarece totul este format din cutii colorate, desenarea nu este complicată: vom crea elemente DOM și le vom stiliza pentru a seta culoarea de fundal, dimensiunile și poziția.

{{index "table (HTML tag)"}}

Putem reprezenta fundalul sub forma unui tabel deoarece este un grid de pătrate care nu se modifică. Elementele care se mișcă liber pot fi suprapuse ca elemente poziționate absolut.

{{index performance, [DOM, graphics]}}

În jocuri și alte programe care animă grafica și răspund la intrarea utilizatorului fără întârzieri remarcabile, eficiența este importantă. Deși DOM nu a fost conceput inițial pentru grafică de performanță înaltă, este de fapt mai bun decât v-ați aștepta la aceasta. Ați văzut câteva animații în [capitolul ?](dom#animation). Pe o mașină modernă, un joc simplu ca și acesta va performa bine, deși nu ne vom preocupa prea mult de optimizări.

{{index canvas, [DOM, graphics]}}

În [capitolul următor](canvas), vom explora o altă tehnologie în browser, tagul `<canvas>`, care permite un mod mai tradițional de a desena, funcționând cu forme și pixeli, în loc de elemente DOM.

## Nivele

{{index dimensions}}

Ne dorim o modalitate lizibilă și editabilă de a defini nivelele. Deoarece am stabilit că totul va începe pe un grid, putem utiliza stringuri în care fiecare caracter reprezintă un element - fie parte din gridul din fundal, fie element în mișcare.

Planul pentru un nivel mic ar putea arăta astfel:

```{includeCode: true}
let simpleLevelPlan = `
......................
..#................#..
..#..............=.#..
..#.........o.o....#..
..#.@......#####...#..
..#####............#..
......#++++++++++++#..
......##############..
......................`;
```

{{index level}}

`.` reprezintă un spațiu liber, `#` reprezintă pereții iar `+` reprezintă lava. Poziția de început a jucătorului este reprezentată prin `@`. Fiecare `0` este o monedă iar semnul `=` reprezintă un bloc de lavă care se mișcă orizontal.

{{index bouncing}}

Vom avea încă două tipuri de blocuri pentru lava în mișcare: `|` pentru blocuri de lavă care se mișcă vertical, `v` pentru blocuri de lavă care se mișcă doar în jos și apoi încep din nou din poziția lor de start după ce ajung la bază (picuri de lavă)

Un joc complet constă din mai multe nivele pe care jucătorul trebuie să le completeze. Un nivel este complet atunci când toate monedele au fost colectate. Dacă jucătorul atinge lava, nivelul curent este restartat în poziția inițială și jucătorul poate încerca din nou.

{{id level}}

## Citirea unui nivel

{{index "Level class"}}

Clasa de mai jos memorează obiectul ce descrie un nivel. Argumentul său este stringul care definește nivelul.

```{includeCode: true}
class Level {
  constructor(plan) {
    let rows = plan.trim().split("\n").map(l => [...l]);
    this.height = rows.length;
    this.width = rows[0].length;
    this.startActors = [];

    this.rows = rows.map((row, y) => {
      return row.map((ch, x) => {
        let type = levelChars[ch];
        if (typeof type == "string") return type;
        this.startActors.push(
          type.create(new Vec(x, y), ch));
        return "empty";
      });
    });
  }
}
```

{{index "trim method", "split method", [whitespace, trimming]}}

Utilizăm metoda `trim` pentru a elimina spațiile albe de la începutul și de la sfârșitul unui string. Astfel putem începe cu o linie goală și toate celelalte linii ale planului unui nivel vor fi aliniate una sub alta. Stringul rezultat este împărțit pe linii și fiecare linie este distribuită într-un array, astfel încât va fi un array de caractere.

{{index [array, "as matrix"]}}

Prin urmare, `rows` memorează un array de array-uri de caractere, rândurile planului. De aici putem determina lățimea și înălțimea unui nivel. Dar trebui să separăm elementele în mișcare de elementele gridului. Elementele în mișcare le vom denumi _actori_. Ele vor fi stocate într-un array de obiecte. Fundalul va fi un array de array-uri de stringuri, fiecare având un tip al câmpului, cum ar fi `"empty"`, `"wall"` sau `"lava"`.

{{index "map method"}}

Pentru a crea aceste array-uri, mai întâi mapăm rândurile și apoi conținutul lor. Vă reamintesc că `map` transmite indexul din array ca și al doilea argument al funcției de mapare, ceea ce ne permite să determinăm coordonatele x și y ale unui caracter dat. Pozițiile în joc vor fi memorate ca și perechi de coordonate, colțul stânga sus fiind (0,0) și fiecare pătrat din fundal având lățimea și înălțimea egale cu o unitate.

{{index "static method"}}

Pentru a interpreta caracterele din plan, constructorul clasei `Level` utilizează obictul `levelChars` care mapează elementele fundalului în stringuri și caracterele-actori în clase. Când `type` este o clasă actor, metoda sa statică `create` este apelată pentru a crea un obiect, care este adăugat la `startActors` iar funcția de mapare returnează `"empty"` pentru pătratul de fundal.

{{index "Vec class"}}

Poziția actorului este memorată ca un obiect `Vec`. Acvesta este un vector bidimensional, un obiect cu proprietăți `x` și `y`, așa cum am văzut în exercițiul din [capitolul ?](object#exercise_vector).

{{index [state, in objects]}}

Pe măsură ce jocul rulează, actorii vor avea poziții diferite sau chiar vor dispărea (cum se întâmplă cu monedele după ce sunt colectate). Vom utiliza o clasă `State` pentru a urmări starea jocului.

```{includeCode: true}
class State {
  constructor(level, actors, status) {
    this.level = level;
    this.actors = actors;
    this.status = status;
  }

  static start(level) {
    return new State(level, level.startActors, "playing");
  }

  get player() {
    return this.actors.find(a => a.type == "player");
  }
}
```

Proprietatea `status` va comuta la `"lost"` sau `"won"` la sfârșitul jocului.

Aceasta este tot o structură de date persistentă - actualizarea stării jocului crează o nouă stare și o lasă pe cea veche nemodificată.

## Actorii

{{index actor, "Vec class", [interface, object]}}

Obiectele actori reprezintă poziția și starea curentă a unui element în mișcare din jocul nostru. Toate obiectele actori se conformează aceleiași interfețe. Proprietatea `pos` memorează coordonatele colțului stânga-sus al unui element iar proprietatea `size` reține dimensiunea.

Apoi, aceste obiecte au o metodă `update`, care este utilizată pentru a calcula noua lor stare și poziție, după un anumit interval de timp. Aceasta simulează ceea ce face actorul - se deplasează ca și răspuns la tastele săgeți (pentru jucător) sau se plimbă pe ecran (pentru lavă) - și returnează un nou obiect actor, actualizat.

Proprietatea `type` conține un string care identifică tipul actorului - `"player"`, `"coin"` sau `"lava"`. Aceasta este utilă pentru desenarea actorului - aspectul dreptunghiului pentru jucător este bazat pe tipul său.

Clasele actor au o metodă statică `create` care este utilizată de constructorul clasei `Level` pentru a crea un actor pe baza caracterului din planul nivelului. Primește ca argumente coordonatele unui caracter și caracterul însuși, ceea ce este necesar pentru că în clasa `Lava` gestionăm mai multe caractere diferite.

{{id vector}}

Mai jos puteți vedea clasa `Vec` pe care o vom utiliza pentru valorile noastre bidimensionale, cum ar fi poziția și dimensiunea fiecărui actor.

```{includeCode: true}
class Vec {
  constructor(x, y) {
    this.x = x; this.y = y;
  }
  plus(other) {
    return new Vec(this.x + other.x, this.y + other.y);
  }
  times(factor) {
    return new Vec(this.x * factor, this.y * factor);
  }
}
```

{{index "times method", multiplication}}

Metoda `times` scalează un vector cu un factor dat. Va fi utilă când vom avea nevoie să multiplicăm un vector de viteze cu o durată pentru a determina distanța străbătută în acel interval de timp.

Diferitele tipuri de actori au fiecare propria clasă, deoarece comportamentul lor este foarte diferit. Să definim aceste clase. Vom scrie metodele lor `update` puțin mai târziu.

{{index simulation, "Player class"}}

Proprietatea `speed` a clasei `Player` memorează viteza curentă a jucătorului pentru a simula gravitația.

```{includeCode: true}
class Player {
  constructor(pos, speed) {
    this.pos = pos;
    this.speed = speed;
  }

  get type() { return "player"; }

  static create(pos) {
    return new Player(pos.plus(new Vec(0, -0.5)),
                      new Vec(0, 0));
  }
}

Player.prototype.size = new Vec(0.8, 1.5);
```

Deoarece dreptunghiul pentru jucător are inălțimea de 1.5 pătrate, poziția sa inițială este setată cu 0.5 pătrate mai sus decât poziția în care a apărut caracterul `@`. Astfel, baza va fi aliniată cu baza pătratului în care este afișat.

Proprietatea `size` este aceași pentru toate instanțele `Player` astfel că o memorăm în prototip, nu în instanțe. Am fi putut utiliza un getter pentru `type`, dar astfel am fi creat și returnat un nou obiect `Vec` de fiecare dată când am fi citit proprietatea, ceea ce ar fi fost un consum inutil de resurse. (Stringurile, fiind imutabile, nu trebuie să fie recreate de fiecare dată când sunt evaluate.)

{{index "Lava class", bouncing}}

Când construim un actor `Lava`, trebuie să inițializăm obiectul diferit în funcție de caracterul pe baza căruia este construit. Lava dinamică se mișcă la viteza sa curentă până când ajunge la un obstacol. Apoi, dacă are o proprietate `reset`, va sări înapoi în poziția sa inițială (picura). Dacă nu, își va inversa sensul de mișcare și va continua în direcția opusă (bouncing).

Metoda `create` consultă caracterul transmis de constructorul `Level` și crează un actor `Lava` corespunzător.

```{includeCode: true}
class Lava {
  constructor(pos, speed, reset) {
    this.pos = pos;
    this.speed = speed;
    this.reset = reset;
  }

  get type() { return "lava"; }

  static create(pos, ch) {
    if (ch == "=") {
      return new Lava(pos, new Vec(2, 0));
    } else if (ch == "|") {
      return new Lava(pos, new Vec(0, 2));
    } else if (ch == "v") {
      return new Lava(pos, new Vec(0, 3), pos);
    }
  }
}

Lava.prototype.size = new Vec(1, 1);
```

{{index "Coin class", animation}}

Actorii `Coin` sunt relativ simpli. Ei sunt statici în poziția setată. Dar pentru a face jocul puțin mai atractiv, îi animăm printr-o vibrație ușoară pe verticală. Pentru aceasta, un obiect `Coin` memorează o poziție de bază precum și o proprietate `wobble` care urmărește faza mișcării. Împreună, acestea determină poziția monedei (memorată în proprietatea `pos`).

```{includeCode: true}
class Coin {
  constructor(pos, basePos, wobble) {
    this.pos = pos;
    this.basePos = basePos;
    this.wobble = wobble;
  }

  get type() { return "coin"; }

  static create(pos) {
    let basePos = pos.plus(new Vec(0.2, 0.1));
    return new Coin(basePos, basePos,
                    Math.random() * Math.PI * 2);
  }
}

Coin.prototype.size = new Vec(0.6, 0.6);
```

{{index "Math.random function", "random number", "Math.sin function", sine, wave}}

În [capitolul ?](dom#sin_cos), am văzut că `Math.sin` determină coordonata y a unui punct pe un cerc. Această coordonată se deplaseză sus-jos pe măsură ce efectuăm rotația pe cerc, ceea ce face funcția "sinus" utilă pentru modelarea unei mișcări de unduire.

{{index pi}}

Pentru a evita situația în care toate monedele se mișcă sincron în sus și în jos, faza inițială a fiecărei monede este setată aleator. Perioada undei `Math.sin`, lățimea undei pe care o produce, este 2π. Dacă multiplicăm valoarea returnată de `Math.random` cu acest număr, putem aloca fiecărei monede, o aleator, o poziție de start pe această undă.

{{index map, [object, "as map"]}}

Acum putem defini obiectul `levelChars`, care mapează caracterele planului fie la tipuri de elemente pentru gridul de fundal, fie la clase-actor.

```{includeCode: true}
const levelChars = {
  ".": "empty", "#": "wall", "+": "lava",
  "@": Player, "o": Coin,
  "=": Lava, "|": Lava, "v": Lava
};
```

Astfel, avem toate părțile de care avem nevoie pentru a crea o instanță `Level`. 

```{includeCode: strip_log}
let simpleLevel = new Level(simpleLevelPlan);
console.log(`${simpleLevel.width} by ${simpleLevel.height}`);
// → 22 by 9
```

Sarcina următoare este de a afișa asemenea nivele pe ecran și a modela timpul și mișcarea pentru ele.

## Încapsularea ca o povară

{{index "programming style", "program size", complexity}}

Majoritatea codului din acest capitol nu se preocupă prea mult de încapsulare, din două motive. Mai întâi, încapsularea necesită efort suplimentar. Ea crește dimensiunea programelor și necesită introducerea unor concepte și interfețe suplimentare. Deoarece cantitatea de cod care poate fi pusă în fața cititorului nu este prea mare, am făcut efortul de a menține programul la o dimensiune mică.

{{index [interface, design]}}

În al doilea rând, diferitele elemente în acest joc sunt atât de puternic legate unele de altele încât dacă s-ar schimba comportamentul unuia dintre ele este puțin probabil ca oricare dintre celelalte să rămână nemodificat. Interfețele dintre elemente ar codifica multe presupuneri despre modul în care jocul funcționează. Aceasta le-ar face mult mai ineficiente - oricând modificați o parte din sistem, va trebui să vă preocupe impacvtul asupra celorlalte părți deoarece interfețele lor nu vor acoperi noua situație.

Unele _puncte cheie_ dintr-un sistem beneficiază mult de separarea prin interfețe riguroase, dar altele nu. Încercarea de a încapsula ceva ce nu reprezintă o graniță adecvată este o modalitate sigură de a pierde multă energie. Când faceți această greșeală, veți observa de regulă că interfețele voastre devin anormal de mari și detaliate și suferă frecvent modificări, pe măsură ce programul evoluează.

{{index graphics, encapsulation, graphics}}

Totuți vom încapsula o parte, și anume subsistemul de desenare. Motivul pentru această decizie este legat de faptul că vom afișa același joc într-un mod diferit în [capitolul următor](canvas#canvasdisplay). Plasând desenarea în spatele unei interfețe, vom putea încărca același program al jocului și să conectăm un nou modul pentru afișare. 

{{id domdisplay}}

## Desenarea

{{index "DOMDisplay class", [DOM, graphics]}}

Încapsularea codului pentru desenare este realizată prin definirea unui obiect pentru afișare, care va afișa un nivel dat și starea acestui. Tipul de afișare pe care îl definim în acest capitol se numește `DOMDisplay` deoarece utilizează elemente DOM pentru a afișa nivelul jocului.

{{index "style attribute", CSS}}

Vom utiliza o foaie de stil pentru a seta culorile și alte proprietăți fixe ale elementelor din joc. De asemenea, vom putea atribui direct valori prin proprietatea `style` a elementelor, la momentul creerii lor, dar programul ar crește în dimensiune.

{{index "class attribute"}}

Următoarea funcție helper oferă o modalitate succintă de a crea un element și a îi asocia unele atribute, precum și noduri copil:

```{includeCode: true}
function elt(name, attrs, ...children) {
  let dom = document.createElement(name);
  for (let attr of Object.keys(attrs)) {
    dom.setAttribute(attr, attrs[attr]);
  }
  for (let child of children) {
    dom.appendChild(child);
  }
  return dom;
}
```

O afișare este creată prin transmiterea unui element părinte la care să se adauge și un obiect pentru descrierea nivelului de joc.

```{includeCode: true}
class DOMDisplay {
  constructor(parent, level) {
    this.dom = elt("div", {class: "game"}, drawGrid(level));
    this.actorLayer = null;
    parent.appendChild(this.dom);
  }

  clear() { this.dom.remove(); }
}
```

{{index level}}

Gridul din fundalul nivelului, care nu se modifică niciodată, este desenat o singură dată. Actorii sunt redesenați de fiecare dată când afișarea este actualizată pe baza unei anumite stări. Proprietatea `actorLayer` va fi utilizată pentru a urmări elementul care conține actorii, astfel încât aceștia vor putea fi ușor eliminați sau înlocuiți.

{{index scaling, "DOMDisplay class"}}

Coordonatele și dimensiunile sunt setate în unități ale gridului, adică o dimensiune sau o distanță de 1 înseamnă un bloc al gridului. Dacă am lucra în pixeli, ar trebui să scalăm în sus aceste coordonate - toate elementele jocului ar fi ridicol de mici daca fiecare pătrat ar avea latura de un pixel. Constanta `scale` reprezintă numărul de pixeli pentru o singură unitate, pe ecran.

```{includeCode: true}
const scale = 20;

function drawGrid(level) {
  return elt("table", {
    class: "background",
    style: `width: ${level.width * scale}px`
  }, ...level.rows.map(row =>
    elt("tr", {style: `height: ${scale}px`},
        ...row.map(type => elt("td", {class: type})))
  ));
}
```

{{index "table (HTML tag)", "tr (HTML tag)", "td (HTML tag)", "spread operator"}}

După cum am menționat, fundalul este desenat ca un element `<table>`. O astfel de structură corespunde cu proprietatea `rows` a nivelului - fiecare rând al grilei este transformat într-un rând al tabelului (element `<tr>`). Stringurile din grilă sunt utilizate ca și nume ale claselor pentru celulele tabelului (elemente `<td>`). Operatorul `...` este utilizat pentru a transmite array-uri de noduri-copil către `elt` ca și argumente separate.

{{id game_css}}

Regulile CSS de mai jos setează aspectul tabelului așa cum îl dorim:

```{lang: "text/css"}
.background    { background: rgb(52, 166, 251);
                 table-layout: fixed;
                 border-spacing: 0;              }
.background td { padding: 0;                     }
.lava          { background: rgb(255, 100, 100); }
.wall          { background: white;              }
```

{{index "padding (CSS)"}}

Unele dintre acestea (`table-layout`, `border-spacing` și `padding`) sunt utilizate pentru a suprima comportament nedorit implicit. Nu vrem ca aspectul tabelului să depindă de conținutul său și nu vrem spații între celulele tabelului, sau padding în interiorul acestora.

{{index [DOM, graphics]}}

Desenăm fiecare actor prin crearea unui element DOM pentru el și setarea poziției și a dimensiunii pe baza proprietăților actorului. Valorile trebuie să fie multiplicate cu `scale` pentru a trece de la unitățile jocului la pixeli.

```{includeCode: true}
function drawActors(actors) {
  return elt("div", {}, ...actors.map(actor => {
    let rect = elt("div", {class: `actor ${actor.type}`});
    rect.style.width = `${actor.size.x * scale}px`;
    rect.style.height = `${actor.size.y * scale}px`;
    rect.style.left = `${actor.pos.x * scale}px`;
    rect.style.top = `${actor.pos.y * scale}px`;
    return rect;
  }));
}
```

{{index "position (CSS)", "class attribute"}}

Pentru a seta mai multe clase pentru un element, separăm numele lor prin spații. În codul CSS care urmează, clasa `actor` setează poziționare absolută pentru elemente. Tipul lor este utilizat ca și o clasă suplimentară pentru a le seta culoarea. Nu trebuie să definim clasa `lava` din nou deoarece reutilizăm clasa pentru pătratele din grilă pe care am definit-o anterior.

```{lang: "text/css"}
.actor  { position: absolute;            }
.coin   { background: rgb(241, 229, 89); }
.player { background: rgb(64, 64, 64);   }
```

{{index graphics, optimization, efficiency, [state, "of application"], [DOM, graphics]}}

Metoda `syncState` este utilizată pentru a afișa o anumită stare. Mai întâi se elimină vechile obiecte pentru actori, apoi se redesenează actorii în noile lor poziții. Ar putea fi tentant să încercăm să reutilizăm elementele DOM pentru actori, dar pentru ca această abordare să funcționeze ar trebui să facem multă contabilitate în plus pentru a asocia actorii cu elementele DOM și pentru a ne asigura că eliminăm elementele când dispar actorii asociați cu ele. Deoarece numărul actorilor din scenă va fi relativ mic, redesenarea tuturor actorilor nu este costisitoare.

```{includeCode: true}
DOMDisplay.prototype.syncState = function(state) {
  if (this.actorLayer) this.actorLayer.remove();
  this.actorLayer = drawActors(state.actors);
  this.dom.appendChild(this.actorLayer);
  this.dom.className = `game ${state.status}`;
  this.scrollPlayerIntoView(state);
};
```

{{index level, "class attribute"}}

Adăugând starea curentă a nivelului ca și nume de clasă, putem stiliza actorul-jucător ușor diferit când jocul a fost câștigat sau pierdut prin adăugarea unei reguli CSS care se aplică doar dacă elementul pentru jucător are un element ancestor cu o anumită clasă.

```{lang: "text/css"}
.lost .player {
  background: rgb(160, 64, 64);
}
.won .player {
  box-shadow: -4px -7px 8px white, 4px -7px 8px white;
}
```

{{index player, "box shadow (CSS)"}}

După ce atinge lava, culoarea jucătorului se modifică. Când a fost colectată ultima monedă, adăugăm două umbre albe - în partea de sus, în stânga și în dreapta, pentru a crea un efect de halo.

{{id viewport}}

{{index "position (CSS)", "max-width (CSS)", "overflow (CSS)", "max-height (CSS)", viewport, scrolling, [DOM, graphics]}}

Nu putem presupune că nivelul încape întotdeauna în _viewport_ - elementul în interiorul căruia desenăm jocul. De aceea este necesar apelul funcției `scrollPlayerIntoView`. Aceasta ne asigură că, dacă nivelul depășește limitele viewportului, repoziționăm viewportul pentru a ne asigura că jucătorul este aproximativ în mijlocul scenei. Regula CSS de mai jos dă elementului DOM care conține scena jocului o dimensiune maximă și ne asigură că tot ceea ce ar fi plasat în afara cutiei elementului nu este vizibil. De asemenea, îi dăm o poziție relativă astfel încât vom putea plasa actorii relativ la colțul stânga-sus al nivelului desenat.

```{lang: "text/css"}
.game {
  overflow: hidden;
  max-width: 600px;
  max-height: 450px;
  position: relative;
}
```

{{index scrolling}}

În metoda `scrollPlayerIntoView` determinăm poziția jucătorului și actualizăm poziția de scroll a elementului ce conține jocul. Schimbăm poziția de scroll prin manipularea proprietăților `scrollLeft` și `scrollTop` când jucătorul este prea aproape de margini.

```{includeCode: true}
DOMDisplay.prototype.scrollPlayerIntoView = function(state) {
  let width = this.dom.clientWidth;
  let height = this.dom.clientHeight;
  let margin = width / 3;

  // The viewport
  let left = this.dom.scrollLeft, right = left + width;
  let top = this.dom.scrollTop, bottom = top + height;

  let player = state.player;
  let center = player.pos.plus(player.size.times(0.5))
                         .times(scale);

  if (center.x < left + margin) {
    this.dom.scrollLeft = center.x - margin;
  } else if (center.x > right - margin) {
    this.dom.scrollLeft = center.x + margin - width;
  }
  if (center.y < top + margin) {
    this.dom.scrollTop = center.y - margin;
  } else if (center.y > bottom - margin) {
    this.dom.scrollTop = center.y + margin - height;
  }
};
```

{{index center, coordinates, readability}}

Modul în care este determinat centrul jucătorului demonstrează cum putem folosi metodele de pe tipul nostru `Vec` pentru a descrie calculele pe obiecte într-un mod relativ lizibil. Pentru a determina centrul unui actor, adunăm la poziția sa (colțul său stânga-sus) jumătate din dimensiunea sa. Acesta este centrul în coordonate ale jocului, dar trebuie să îl calculăm în pixeli, deci va trebui să înmulțim vectorul resultat cu factorul de scală pe care îl folosim în afișare.

{{index validation}}

Apoi, o serie de verificări validează că poziția jucătorului nu este în afara domeniului permis. Uneori, vom seta coordonate pentru scroll care nu fac sens (negative sau dincolo de zona derulabilă). Nu este nici o problemă, DOM le va constrânge la valori acceptabile. Dacă setăm `scrollLeft` la -10 va deveni 0.

Ar fi fost mai simplu dacă am fi derulat tot timpul jucătorul în centrul viewportului. Dar efectul vizual ar fi fost enervant. Când ar sări, view-ul s-a deplasa sus și jos. Este mai plăcut să avem o zonă "neutră" în centrul ecranului în care jucătorul se poate deplasa fără a cauza scroll.

{{index [game, screenshot]}}

Acum putem afișa nivelul nostru.

```{lang: "text/html"}
<link rel="stylesheet" href="css/game.css">

<script>
  let simpleLevel = new Level(simpleLevelPlan);
  let display = new DOMDisplay(document.body, simpleLevel);
  display.syncState(State.start(simpleLevel));
</script>
```

{{if book

{{figure {url: "img/game_simpleLevel.png", alt: "Our level rendered",width: "7cm"}}}

if}}

{{index "link (HTML tag)", CSS}}

Tagul `<link>`, utilizat cu `rel="stylesheet"`, este o metodă de a încărca un fișier CSS în pagină. Fișierul `game.css` conține toate regulile de stil necesare pentru jocul nostru.

## Mișcare și coliziuni

{{index physics, [animation, "platform game"]}}

Acum suntem în punctul în care putem adăuga mișcare - cel mai interesant aspect al jocului. Abordarea de bază, utilizată de toate jocurile de acest gen, este de a împărți linia de timp a jocului în mici pași și, pentru fiecare pas, deplasează actorii pe o distanță ce corespunde vitezei lor, multiplicată cu durata pasului de timp. Vom măsura timpul în secunde, astfel încât vitezele vor fi exprimate în unități pe secundă.

{{index obstacle, "collision detection"}}

Deplasarea lucrurilor este simplă. Partea dificilă este gestionarea interacțiunilor dintre elemente. Când jucătorul atinge un perete sau podeaua, nu ar trebui să se deplaseze prin ea. Jocul trebuie să detecteze atunci când mișcarea cauzează o ciocnire a unui obiect cu un alt obiect și să reacționeze corespunzător. Pentru pereți, deplasarea trebuie să fie blocată. Când este lovită o monedă, aceasta trebuie să fie colectată. Când se atinge lava, jocul va fi pierdut.

Rezolvarea acestei probleme în general este o sarcină dificilă. Dar puteți găsi librării, numite _physics engines_ care simulează interacțiunea dintre obiectele fizice în două sau trei dimensiuni. Abordarea noastră va fi mai modestă în acest capitol, gestionând doar coliziuni între obiecte dreptunghiulare, într-un mod simplu.

{{index bouncing, "collision detection", [animation, "platform game"]}}

Înainte de a muta jucătorul sau un bloc de lava, testăm dacă mișcarea îl va duce în interiorul unui perete. Dacă da, renunțăm la deplasare. Răspunsul la o astfel de coliziune depinde de tipul actorului - jucătorul se va opri, blocul de lavă își va schimba sensul mișcării.

{{index discretization}}

Această abordare necesită ca pasul temporal să fie relativ mic deoarece mișcarea va fi oprită înainte ca obiectele să se atingă. Dacă pasul temporal (ți în consecință, pasul mișcării) este prea mare, jucătorul va ajunge să se oprească plutin la o distanță observabilă de podea. O altă abordare, mai bună dar și mai complicată, ar fi să determinăm punctul de coliziune și să mutăm jucătorul în acel punct înainte de a-l opri. Vom considera abordarea mai simplă și îi vom ascunde problemele prin utilizarea unor pași mici în animație.

{{index obstacle, "touches method", "collision detection"}}

{{id touches}}

Metoda de mai jos determină dacă un dreptunghi (specificat prin poziție și dimensiune) atinge un element al gridului de un anume tip.

```{includeCode: true}
Level.prototype.touches = function(pos, size, type) {
  let xStart = Math.floor(pos.x);
  let xEnd = Math.ceil(pos.x + size.x);
  let yStart = Math.floor(pos.y);
  let yEnd = Math.ceil(pos.y + size.y);

  for (let y = yStart; y < yEnd; y++) {
    for (let x = xStart; x < xEnd; x++) {
      let isOutside = x < 0 || x >= this.width ||
                      y < 0 || y >= this.height;
      let here = isOutside ? "wall" : this.rows[y][x];
      if (here == type) return true;
    }
  }
  return false;
};
```

{{index "Math.floor function", "Math.ceil function"}}

Metoda calculează setul de pătrate ale gridului pe care se suprapune corpul, utilizând pentru aceasta `Math.floor` și `Math.ceil` asupra coordonatelor sale. Gridul folosește pătrate cu dimensiunea 1 pe 1. Rotunjind lungimile laturilor cutiei (în sus și în jos), determinăm domeniul pătratelor din fundal pe care cutia le atinge.

{{figure {url: "img/game-grid.svg", alt: "Finding collisions on a grid",width: "3cm"}}}

Ciclăm printre aceste pătrate determinate prin rotunjirea coordonatelor și returnăm `true` dacă găsim un pătrat care se potrivește. Pătratele din afara nivelului sunt întotdeauna tratate ca și `"wall"` pentru a ne asigura că jucătorul nu poate părăsi lumea jocului și că nu vom încerca să citim accidental din afara limitelor array-ului nostru `rows`.

Metoda `update` pentru actualizarea stării utilizează `touches` pentru a determina dacă jucătorul atinge lava.

```{includeCode: true}
State.prototype.update = function(time, keys) {
  let actors = this.actors
    .map(actor => actor.update(time, this, keys));
  let newState = new State(this.level, actors, this.status);

  if (newState.status != "playing") return newState;

  let player = newState.player;
  if (this.level.touches(player.pos, player.size, "lava")) {
    return new State(this.level, actors, "lost");
  }

  for (let actor of actors) {
    if (actor != player && overlap(actor, player)) {
      newState = actor.collide(newState);
    }
  }
  return newState;
};
```

Acestei metode i se transmite un pas temporal și o structură de date care comunică ce taste sunt apăsate. Primul lucru pe care îl face este să apeleze metoda `update` asupra tuturor actorilor, producând un array de actori actualizați. Actorii primesc și ei pasul temporal, tastele și starea, astfel încât își vor putea actualiza starea pe baza acestora. Doar jucătorul va citi de fapt tastele deoarece el este singurul actor controlat de la tastatură.

Dacă jocul s-a încheiat, nu se mai face nici o prelucrare (jocul nu poate fi câștigat după ce a fost pierdut, sau invers). Altfel, metoda testează dacă jucătorul atinge lava de pe fundal. Dacă da, jocul a fost pierdut și suntem gata. În sfârșit, dacă jocul trebuie să continue, se testează dacă oricare dintre ceilalți actori se suprapune cu jucătorul.

Suprapunerea dintre actori este detectată în funcția `overlap`. Aceasta primește două obiecte de tip actor și returnează `true` dacă ei se ating - ceea ce se întâmplă dacă ei se suprapun atât pe axa x cât și pe axa y.

```{includeCode: true}
function overlap(actor1, actor2) {
  return actor1.pos.x + actor1.size.x > actor2.pos.x &&
         actor1.pos.x < actor2.pos.x + actor2.size.x &&
         actor1.pos.y + actor1.size.y > actor2.pos.y &&
         actor1.pos.y < actor2.pos.y + actor2.size.y;
}
```

Dacă oricare actor se suprapune, metoda sa `collide` are o șansă să actualizeze starea. Atingerea unui actor lava setează starea jocului la `"lost"`. Monedele dispar când sunt atinse și starea este actualizată la `"won"` când s-a atins ultima dintre monede.

```{includeCode: true}
Lava.prototype.collide = function(state) {
  return new State(state.level, state.actors, "lost");
};

Coin.prototype.collide = function(state) {
  let filtered = state.actors.filter(a => a != this);
  let status = state.status;
  if (!filtered.some(a => a.type == "coin")) status = "won";
  return new State(state.level, filtered, status);
};
```

{{id actors}}

## Actualizările actorului

{{index actor, "Lava class", lava}}

Metodele `update` ale obiectelor actor primesc ca și argumente pasul temporal, starea obiectului și un obiect `keys`. Cele pentru actori `Lava` ignoră obiectul `keys`.

```{includeCode: true}
Lava.prototype.update = function(time, state) {
  let newPos = this.pos.plus(this.speed.times(time));
  if (!state.level.touches(newPos, this.size, "wall")) {
    return new Lava(newPos, this.speed, this.reset);
  } else if (this.reset) {
    return new Lava(this.reset, this.speed, this.reset);
  } else {
    return new Lava(this.pos, this.speed.times(-1));
  }
};
```

{{index bouncing, multiplication, "Vec class", "collision detection"}}

Metoda `update` de mai sus calculează o nouă poziție prin adăugarea produsului dintre pasul temporal și viteza curentă la vechea poziție. Dacă nu există un obstacol care să blocheze noua poziție, se deplasează acolo. Dacă există un obstacol, comportamentul depinde de tipul blocului de lavă. Blocurile de lavă care se mișcă pe verticală au o funcție `reset` care le mută în poziția inițială, iar cele care se mișcă pe direcție orizontală își inversează sensul de mișcare prin multiplicarea vitezei cu -1.

{{index "Coin class", coin, wave}}

Monedele își utilizează mettoda `update` pentru a oscila. Ele ignoră coliziunile cu gridul deoarece ele oscilează în interiorul propriului lor pătrat din grid.

```{includeCode: true}
const wobbleSpeed = 8, wobbleDist = 0.07;

Coin.prototype.update = function(time) {
  let wobble = this.wobble + time * wobbleSpeed;
  let wobblePos = Math.sin(wobble) * wobbleDist;
  return new Coin(this.basePos.plus(new Vec(0, wobblePos)),
                  this.basePos, wobble);
};
```

{{index "Math.sin function", sine, phase}}

Proprietatea `wobble` este incrementată pentru a urmări timpul și apoi este utilizată ca și argument pentru `Math.sin` pentru a determina noua poziție pe undă. Poziția curentă a monedei este apoi calculată folosind poziția sa de bază și un offset bazat pe unda generată de funcția sinus.

{{index "collision detection", "Player class"}}

Așa că ne mai rămâne doar jucătorul. Mișcarea jucătorului este gestionată separat pe cele două axe, deoarece atingerea podelei nu trebuie să blocheze mișcarea pe orizontală, iar atingerea unui perete nu ar trebui să influențeze căderea sau saltul pe verticală.

```{includeCode: true}
const playerXSpeed = 7;
const gravity = 30;
const jumpSpeed = 17;

Player.prototype.update = function(time, state, keys) {
  let xSpeed = 0;
  if (keys.ArrowLeft) xSpeed -= playerXSpeed;
  if (keys.ArrowRight) xSpeed += playerXSpeed;
  let pos = this.pos;
  let movedX = pos.plus(new Vec(xSpeed * time, 0));
  if (!state.level.touches(movedX, this.size, "wall")) {
    pos = movedX;
  }

  let ySpeed = this.speed.y + time * gravity;
  let movedY = pos.plus(new Vec(0, ySpeed * time));
  if (!state.level.touches(movedY, this.size, "wall")) {
    pos = movedY;
  } else if (keys.ArrowUp && ySpeed > 0) {
    ySpeed = -jumpSpeed;
  } else {
    ySpeed = 0;
  }
  return new Player(pos, new Vec(xSpeed, ySpeed));
};
```

{{index [animation, "platform game"], keyboard}}

Mișcarea pe orizontală este calculată pe baza stării cheilor pentru săgeata stânga și săgeata dreapta. Când nu există un bloc care să blocheze mișcarea, noua poziție este utilizată. Altfel, se păstrează vechea poziție.

{{index acceleration, physics}}

Mișcarea pe verticală funcționează asemănător dar trebuie să simuleze saltul și gravitația. Viteza pe verticală a jucătorului (`ySpeed`) este accelerată pentru a ține cont de gravitație.

{{index "collision detection", keyboard, jumping}}

Din nouă verificăm pereții. Dacă nu atingem vreunul, noua poziție este utilizată. Dacă există un perete, avem două rezultate posibile. Când a fost apăsată tasta săgeată sus _și_ ne mișcăm în jos (adică obiectul lovit se află sub jucător), viteza este setată la o valoare mare, negativă. Aceasta face ca jucătorul să sară. Dacă nu acesta este cazul, înseamnă că jucătorul s-a lovit de ceva și viteza este setată la zero.

Tăria gravitației, viteza de salt și cam toate celelalte constante din acest joc au fost setate prin "trial and error". Adică am tot testat diverse valori până când am obținut o combinație satisfăcătoare.

## Urmărirea tastelor

{{index keyboard}}

Pentru un joc ca și acesta, nu vrem să avem efectul tastelor doar o singură dată pentru apăsarea unei taste. Am vrea ca efectul (deplasarea jucătorului) să rămână cât timp tastele sunt menținute apăsate.

{{index "preventDefault method"}}

Trebuie să setăm un handler de eveniment pentru taste care stochează starea curentă a tastelor stânga, dreapta și sus. De asemenea, vom apela `preventDefault` pentru aceste taste, pentru a evita efectul implicit de derulare a paginii.

{{index "trackKeys function", "key code", "event handling", "addEventListener method"}}

Funcția ce urmează primește un array cu numele unor taste și va returna un obiect care urmărește poziția curentă a acelor taste. Vom înregistra handlere de eveniment pentru `"keydown"` și `"keyup"` și, dacă codul tastei din eveniment este prezent în setul de coduri care sunt urmărite, obiectul va fi actualizat.

```{includeCode: true}
function trackKeys(keys) {
  let down = Object.create(null);
  function track(event) {
    if (keys.includes(event.key)) {
      down[event.key] = event.type == "keydown";
      event.preventDefault();
    }
  }
  window.addEventListener("keydown", track);
  window.addEventListener("keyup", track);
  return down;
}

const arrowKeys =
  trackKeys(["ArrowLeft", "ArrowRight", "ArrowUp"]);
```

{{index "keydown event", "keyup event"}}

Aceeași funcție handler este utilizată pentru ambele tipuri de evenimente. Ea consultă proprietatea `type` a obiectului evenimentului pentru a determina dacă starea tastei trebuie actualizată la `true` (`"keydown"`) sau `false` (`"keyup"`).

{{id runAnimation}}

## Rularea jocului

{{index "requestAnimationFrame function", [animation, "platform game"]}}

Funcția `requestAnimationFrame`, pe care am utilizat-o în [capitolul ?](dom#animationFrame), este o modalitate bună pentru a anima un joc. Dar interfața sa este destul de primitivă - utilizarea ei necesită să urmărim momentul de timp la care funcția a fost apelată ultima dată și să apelăm `requestAnimationFrame` din nou, după fiecare cadru.

{{index "runAnimation function", "callback function", [function, "as value"], [function, "higher-order"], [animation, "platform game"]}}

Haideți să definim o funcție helper care împachetează aceste părți plictisitoare într-o interfață convenabilă și ne permite să apelăm pur și simplu `runAnimation`, care primește o funcție care așteaptă o diferență de timp și desenează un singur cadru. Când această funcție returnează valoarea `false`, animația se oprește.

```{includeCode: true}
function runAnimation(frameFunc) {
  let lastTime = null;
  function frame(time) {
    if (lastTime != null) {
      let timeStep = Math.min(time - lastTime, 100) / 1000;
      if (frameFunc(timeStep) === false) return;
    }
    lastTime = time;
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}
```

{{index time, discretization}}

Am setat un pas maxim între două cadre de 100 milisecunde (o zecime de secundă). Când tabul sau fereastra browserului este ascuns, apelurile către `requestAnimationFrame` vor fi suspendate până când tabul sau fereastra vor fi afișate din nou. În acest caz, diferența dintre `lastTime` și `time` va fi durata de timp în care pagina a fost ascunsă. Avansarea jocului cu o asemenea cantitate într-un singur pas ar arăta prostesc și ar putea cauza efecte secundare ciudate, cum ar fi jucătorul care cade prin podea.

Funcția convertește pașii de timp în secunde, care sunt o cantitate mai ușor de perceput decât milisecundele.

{{index "callback function", "runLevel function", [animation, "platform game"]}}

Funcția `runLevel` primește un obiect `Level` și un constructor pentru afișare și returnează o promisiune. Ea afișează un nivel (în `document.body`) și permite utilizatorului să îl joace. Când nivelul se termină (pierdut sau câștigat), `runLevel` așteaptă încă o secundă (pentru a permite utilizatorului să vadă ceea ce se întâmplă) și apoi curăță afișajul, oprește animația și rezolvă promisiune cu starea finală a nivelului.

```{includeCode: true}
function runLevel(level, Display) {
  let display = new Display(document.body, level);
  let state = State.start(level);
  let ending = 1;
  return new Promise(resolve => {
    runAnimation(time => {
      state = state.update(time, arrowKeys);
      display.syncState(state);
      if (state.status == "playing") {
        return true;
      } else if (ending > 0) {
        ending -= time;
        return true;
      } else {
        display.clear();
        resolve(state.status);
        return false;
      }
    });
  });
}
```

{{index "runGame function"}}

Un joc este o secvență de nivele. De fiecare dată când jucătorul "moare", nivelul curent este repornit. Când un nivel este completat, trecem la nivelul următor. Aceasta poate fi exprimată prin următoarea funcție, care primește un array de planuri pentru nivele (stringuri) și un constructor pentru afișare:

```{includeCode: true}
async function runGame(plans, Display) {
  for (let level = 0; level < plans.length;) {
    let status = await runLevel(new Level(plans[level]),
                                Display);
    if (status == "won") level++;
  }
  console.log("You've won!");
}
```

{{index "asynchronous programming", "event handling"}}

Deoarece `runLevel` returnează o promisiune, `runGame` poate fi scris ca o funcție `async`, așa cum am discutat în [capitolul ?](async). Această funcție va returna o altă promisiune, care se rezolvă când jucătorul finalizează jocul.

{{index game, "GAME_LEVELS data set"}}

Găsiți mai multe planuri pentru nivele în bindingul `GAME_LEVELS` [în sandbox-ul acestui capitol](https://eloquentjavascript.net/code#16)[([_https://eloquentjavascript.net/code#16_](https://eloquentjavascript.net/code#16))]{if book}. Această pagină le încarcă în `runGame`, pentru a începe un joc.

```{sandbox: null, focus: yes, lang: "text/html", startCode: true}
<link rel="stylesheet" href="css/game.css">

<body>
  <script>
    runGame(GAME_LEVELS, DOMDisplay);
  </script>
</body>
```

{{if interactive

Vedeți dacă le puteți trece. A fost distractiv să le construiesc.

if}}

## Exerciții

### Game over

{{index "lives (exercise)", game}}

Este o tradiție pentru jocurile de platformă ca jucătorul să înceapă cu un număr limitat de vieți și să scădem câte una de fiecare dată când "moare". Când jucătorul nu mai are vieți. jocul repornește de la început.

{{index "runGame function"}}

Ajustați `runGame` pentru a implementa "viețile". Jucătorul să înceapă cu trei vieți. Afișați numărul de vieți rămase (cu `console.log`) la începutul fiecărui nivel.

{{if interactive

```{lang: "text/html", test: no, focus: yes}
<link rel="stylesheet" href="css/game.css">

<body>
<script>
  // The old runGame function. Modify it...
  async function runGame(plans, Display) {
    for (let level = 0; level < plans.length;) {
      let status = await runLevel(new Level(plans[level]),
                                  Display);
      if (status == "won") level++;
    }
    console.log("You've won!");
  }
  runGame(GAME_LEVELS, DOMDisplay);
</script>
</body>
```

if}}

### Puneți jocul pe pauză

{{index "pausing (exercise)", "escape key", keyboard}}

Faceți posibilă suspendarea și reactivarea jocului prin apăsarea tastei Esc.

{{index "runLevel function", "event handling"}}

Aceasta se poate implementa prin modificarea funcție `runLevel` ca să mai utilizeze încă un handler de evenimente de la tastatură care întrerupe și reia afișsarea ori de câte ori este apăsată tasta Esc.

{{index "runAnimation function"}}

Interfața `runAnimation` ar putea să nu pară adecvată pentru acest scop, dar va fi, dacă rearanjați modul în care `runLevel` o apelează.

{{index [binding, global], "trackKeys function"}}

După ce reușiți să implementați, puteți încerca și altceva. Modul în care am înregistrat handlerele pentru evenimentele de la tastatură este oarecum problematic. Obiectul `arrowKeys` este deocamdată un binding global și handlerele sale pentru evenimente sunt menținute chiar și atunci când nu rulează nici un joc. Putem spune că s-au _scurs (leak)_ în afara sistemului nostru. Extindeți `trackKeys` pentru a oferi o modalitate de a deînregistra handlerele sale și apoi modificați `runLevel` pentru a își înregistra handlerele atunci când începe și să le deînregistreze după ce se termină execuția.

{{if interactive

```{lang: "text/html", focus: yes, test: no}
<link rel="stylesheet" href="css/game.css">

<body>
<script>
  // The old runLevel function. Modify this...
  function runLevel(level, Display) {
    let display = new Display(document.body, level);
    let state = State.start(level);
    let ending = 1;
    return new Promise(resolve => {
      runAnimation(time => {
        state = state.update(time, arrowKeys);
        display.syncState(state);
        if (state.status == "playing") {
          return true;
        } else if (ending > 0) {
          ending -= time;
          return true;
        } else {
          display.clear();
          resolve(state.status);
          return false;
        }
      });
    });
  }
  runGame(GAME_LEVELS, DOMDisplay);
</script>
</body>
```

if}}

{{hint

{{index "pausing (exercise)", [animation, "platform game"]}}

O animație poate fi întreruptă prin returnarea valorii `false` din funcția transmisă către `runAnimation`. Și apoi poate fi reluată prin apelarea `runAnimation` din nou.

{{index closure}}

Deci, trebuie să comunicăm faptul că întrerupem jocul către funcția transmisă spre `runAnimation`. Pentru aceasta, putem folosi un binding la care au acces atât handlerul de eveniment cât și funcția respectivă.

{{index "event handling", "removeEventListener method", [function, "as value"]}}

Când căutați o modalitate de a deînregistra handlerele înregistrate de către `trackKeys`, amintiți-vă că exact acceași funcție care a fost transmisă către `addEventListener` trebuie să fie transmisă către `removeEventListener` pentru a elimina cu succes un handler. Prin urmare, valoarea de tip funcție `handler` creată în `trackKeys` trebuie să fie disponibilă și codului care deînregistrează handlerele.

Puteți adăuga o proprietate la obiectul returnat de către `trackKeys`, care să conțină fie acea valoare de tip funcție fie o metodă care gestionează direct deînregistrarea.

hint}}

### Un monstru

{{index "monster (exercise)"}}

Este o tradiție în jocurile de tip platformă să avem dușmani pe care să putem sări pentru a îi învinge. Acest exercițiu vă cere să adăugați un asemenea tip de actor în joc.

Îl vom denumi "monstru". Monștrii se vor mișca doar pe direcție orizontală. Îi puteți face să se miște în direcția jucătorului, să se miște stânga-dreapta ca și blocurile de lavă orizontale, sau să aibă orice alt comportament dinamic vreți. Clasa nu va trebui să implementeze căderea, dar monștrii nu trebuie să treacă prin pereți.

Când un monstru atinge jucătorul, efectul este influențat de faptul că jucătorul a sărit pe el sau nu. Puteți aproxima aceasta prin a verifica dacă marginea de jos a jucătorului este în apropierea marginii de sus a monstrului. Dacă da, monstrul va dispărea. Dacă nu, jocul este pierdut.

{{if interactive

```{test: no, lang: "text/html", focus: yes}
<link rel="stylesheet" href="css/game.css">
<style>.monster { background: purple }</style>

<body>
  <script>
    // Complete the constructor, update, and collide methods
    class Monster {
      constructor(pos, /* ... */) {}

      get type() { return "monster"; }

      static create(pos) {
        return new Monster(pos.plus(new Vec(0, -1)));
      }

      update(time, state) {}

      collide(state) {}
    }

    Monster.prototype.size = new Vec(1.2, 2);

    levelChars["M"] = Monster;

    runLevel(new Level(`
..................................
.################################.
.#..............................#.
.#..............................#.
.#..............................#.
.#...........................o..#.
.#..@...........................#.
.##########..............########.
..........#..o..o..o..o..#........
..........#...........M..#........
..........################........
..................................
`), DOMDisplay);
  </script>
</body>
```

if}}

{{hint

{{index "monster (exercise)", "persistent data structure"}}

Dacă vreți să implementați un tip de mișcare care este realistă, cum ar fi mișcarea stânga-dreapta, asigurați-vă că rețineți starea necesară în obiectul-actor - includeți-o ca și argument în constructor și adăugați-o ca și proprietate.

Vă reamintesc că `update` returnează un _nou_ obiect, în loc să îl modifice pe cel vechi.

{{index "collision detection"}}

Când gestionați coliziunile, căutați jucătorul în `state.actors` și comparați poziția sa cu poziția mosntrului. Pentru a determina marginea de jos a jucătorului trebuie să îi adăugați dimensiunea sa verticală la poziția sa pe verticală. Crearea unei stări actualizate va fi asemănătoare fie cu metoda `collide` pentru `Coin` (eliminarea actorului), fie cu cu cea din clasa `Lava` (schimbarea stării la `"lost"`), în funcție de poziția jucătorului.

hint}}
