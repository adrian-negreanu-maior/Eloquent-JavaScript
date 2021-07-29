{{meta {load_files: ["code/chapter/19_paint.js"], zip: "html include=[\"css/paint.css\"]"}}}

# Proiect: Un editor Pixel Art

{{quote {author: "Joan Miro", chapter: true}

Mă uit la multele culori din fața mea. Mă uit la canvasul gol. Apoi, încerc să aplic culorile cum se aplică cuvintele pentru a forma poeme sau notele pentru a forma muzica.

quote}}

{{index "Miro, Joan", "drawing program example", "project chapter"}}

{{figure {url: "img/chapter_picture_19.jpg", alt: "Picture of a tiled mosaic", chapter: "framed"}}}

Materialul din capitolul anterior vă pune la dispoziție toate elementele de care aveți nevoie pentru a construi o aplicație simplă. In acest capitol vom construi această aplicație.

{{index [file, image]}}

Aplicația noastră va fi un program de desenare pe pixeli, în care veți putea modifica o imagine pixel cu pixel prin manipularea unei vizualizări mărite a imaginii, afișsată pe un grid de pătrate. Veți putea utiliza programul pentru a deschide fișiere, pe care să le modificați apoi cu mouseul sau alt dispozitiv și, în final, să le salvați. Aplicația va arătă la sfârșit cam așa:

{{figure {url: "img/pixel_editor.png", alt: "The pixel editor interface, with colored pixels at the top and a number of controls below that", width: "8cm"}}}

Pictura pe computer este extraordinară. Nu trebuie să vă îngrijorați în privința materialelor, îndemânării sau a talentului. Doar începeți să pictați.

## Componente

{{index drawing, "select (HTML tag)", "canvas (HTML tag)", component}}

Interfața pentru aplicație afișează un element `<canvas>` în partea de sus și mai multe câmpuri sub el. Utilizatorul desenează pe imagine prin selectarea unui instrument dintr-un câmp `<select>` urmată de click-uri, atingeri sau trageri peste canvas. Vom construi instrumente pentru desenarea unui singur pixel sau a unui dreptunghi, pentru umplerea unei zone și pentru selectarea unei culori din imagine.

{{index [DOM, components]}}

Vom structura interfața editorului ca un număr de _componente_, obiecte care vor fi responsabile pentru o parte din DOM și care vor putea conține alte componente în interiorul lor.

{{index [state, "of application"]}}

Starea aplicației constă din imaginea curentă, instrumentul selectat și culoarea selectată. Vom seta lucrurile astfel încât starea curentă să fie tocată într-o singură valoare iar componentele interfeței să fie afișate conform stării curente.

Pentru a înțelege de ce este important să procedăm astfel, haideți să considerăm varianta alternativă - distribuim părți ale stării peste tot în interfață. Până la un punct, va fi mai ușor să programăm astfel. Am putea să utilizăm un câmp pentru culoare și să îi citim valoarea atunci când vrem să determinăm culoarea curentă.

Dar apoi adăugăm instrumentul pentru selectarea culorii - un instrument care vă permite să efectuați click pe imagine pentru a selecta culoarea unui pixel dat. Pentru a afișa culoarea corectă în câmpul de culoare, acel tool ar trebui să știe de existența acelui câmp și să îl actualizeze la selectarea unei alte culori. Dacă veți adăuga un element care afișează culoarea, va trebui să actualizați din nou codul de modificare a culorii pentru a sincroniza și acel element.

{{index modularity}}

Prin urmare, se crează astfel o problema legată de faptul că fiecare parte a interfeței trebuie să fie conștientă de prezența tuturor celorlalte părți, ceea ce nu este foarte bine din punct de vedere al modularității. Pentru aplicații mici, cum este cea din acest capitol, nu este o problemă. Dar pentru proiecte mai mari, aceasta poate deveni repede un coșmar.

Pentru a evita problema, în principiu, vom fi stricți în privința _fluxului de date_. Există o stare și interfața este desenată pe baza acelei stări. O componentă a interfeței poate răspunde la acțiunile utilizatorului prin actualizarea stării, moment în care componentele au ocazia să se sincronizeze cu noua stare.

{{index library, framework}}

În practică, fiecare componentă este setată astfel încât, atunci când primește o nouă stare, va notifica componentele-copil pentru a fi actualizate. Setarea acestui mod de interacțiune între componente este o oarecare bătaie de cap. Micșorarea efortului în această direcție este principalul aspect de vânzare al multor librării pentru programarea în browser. Dar pentru o aplicație de mici dimensiuni, putem realiza acest lucru fără o asemenea infrastructură.

{{index [state, transitions]}}

Actualizările stării sunt reprezentate de către obiecte, pe care le vom numi _acțiuni_. Componentele pot crea asemenea acțiuni și să le _expedieze_ - să le transmită unei funcții centrale de gestionare. Acea funcție calculează noua stare, după care componentele interfeței se actualizează la noua stare.

{{index [DOM, components]}}

Vom lua sarcina încâlcită de a rula o interfață utilizator și să îi aplicăm o structură. Deși bucățile legate de DOM sunt pline de efecte secundare, ele sunt susținute de o coloană vertebrală conceptual simplă: ciclul de actualizare a stării. Starea determină aspectul DOM-ului și singurul mod în care evenimentele DOM pot modifica starea este expedierea acșiunilor către stare.

{{index "data flow"}}

Sunt _multe_ variante ale acestei abordări, fiecare cu propriile beneficii și probleme, dar ideea centrală este aceeași: modificările stării trebuie să treacă printr-un canal unic și bine definit, nu să aibă loc peste tot în aplicație.

{{index "dom property", [interface, object]}}

Componentele noastre vor fi clase ce se conformează unei interfețe. Constructorul lor va primi o stare - și o va utiliza pentru a construi o proprietate `dom`. Acesta este elementul DOM care reprezintă componenta. Majoritatea constructorilor vor primi și alte valori care nu se vor modifica în timp, cum ar fi funcția pe care o pot utiliza pentru a expedia o acțiune.

{{index "syncState method"}}

Fiecare componentă are o metodă `syncState` care este utilizată pentru a se sincroniza la o nouă valoare a stării. Metoda primește un singur argument, starea, care este de același tip ca și primul său argument al constructorului.

## Starea aplicației

{{index "Picture class", "picture property", "tool property", "color property", "Matrix class"}}

Starea aplicației va fi un obiect cu proprietățile `picture`, `tool` și `color`. Imaginea va fi și ea un obiect care va stoca lungimea, lățimea și conținutul pixelilor din imagine. Pixelii sunt memorați într-un array, similar cu clasa pentru matrice din [capitolul ?](object) - rând cu rând, de sus în jos.

```{includeCode: true}
class Picture {
  constructor(width, height, pixels) {
    this.width = width;
    this.height = height;
    this.pixels = pixels;
  }
  static empty(width, height, color) {
    let pixels = new Array(width * height).fill(color);
    return new Picture(width, height, pixels);
  }
  pixel(x, y) {
    return this.pixels[x + y * this.width];
  }
  draw(pixels) {
    let copy = this.pixels.slice();
    for (let {x, y, color} of pixels) {
      copy[x + y * this.width] = color;
    }
    return new Picture(this.width, this.height, copy);
  }
}
```

{{index "side effect", "persistent data structure"}}

Vrem să putem trata o imagine ca o valoare imutabilă, din motive asupra cărora vom reveni mai târziu în acest capitol. Dar, de asemenea, uneori vom dori să actualizăm un întreg set de pixeli deodată. Pentru aceasta, clasa va avea o metodă `draw` care primește un array de pixeli actualizați - obiecte cu proprietăți `x`, `y` și `color` - și crează o nouă imagine cu acei pixeli suprascriși. Această metodă utilizează `slice` fără argumente pentru a copia întregul array de pixeli - începutul tăierii este implicit 0 iar sfârșitul, lungimea array-ului.

{{index "Array constructor", "fill method", ["length property", "for array"], [array, creation]}}

Metoda `empty` utilizează două funcționalități pentru array-uri pe care nu le-am mai întâlnit. Constructorul `Array` poate fi apelat cu un număr ca argument, pentru a crea un array de lungime dată. Metoda `fill` poate fi apoi utlizată pentru a umple acest array cu o valoare dată. Acestea sunt utilizate pentru a crea un array în care toți pixelii au aceeași culoare.

{{index "hexadecimal number", "color component", "color field", "fillStyle property"}}

Culorile sunt memorate ca și stringuri ce conțin coduri tradiționale CSS, construite cu un symbol `#` și apoi 6 cifre hexazecimale (baza 16), câte două pentru fiecare componentă de culoare RGB. Aceasta este o modalitate destul de criptică și neconvenabilă de a scrie culorile, dar este formatul pe care îl folosește și câmpul HTML pentru culori, și poate fi utilizată în proprietatea `fillStyle` a contextului de desenare al canvasului, așa că este practic suficientă pentru modurile în care vom utiliza culorile în acest program.

{{index black}}

Negrul, pentru care toate componentele au valoarea zero, se scrie `"#000000"`, iar magenta s-ar scrie cam așa: `"#ff00ff"`, cu componentele roșu și albastru setate la valoarea maximă de 255, scrisă `ff` cu cifre hexazecimale (unde folosim literele _a_ - _f_ pentru a reprezenta cifrele de la 10 la 15).

{{index [state, transitions]}}

Vom permite interfeței să expedieze acșiuni ca și obiecte ale căror proprietăți suprascriu propreitățile stării anterioare. Câmpul de culoare, atunci când este modificat de către utilizator, ar putea expedia un obiect `{color: field.value}`, cu ajutorul căruia funcția de actualizare ar putea calcula o nouă stare.

{{index "updateState function"}}

```{includeCode: true}
function updateState(state, action) {
  return Object.assign({}, state, action);
}
```

{{index "period character", spread, "Object.assign function"}}

Acest șablon oarecum greoi, în care `Object.assign` este utilizat mai întâi pentru a adăuga proprietățile stării unui obiect gol, urmând apoi să suprascriem unele dintre ele cu cele din obiectul `action`, este frecvent întâlnit în codul JavaScript care utilizează obiecte imutabile. O notație mai convenabilă ar fi operatorul `...` pentru a include toate proprietățile dintr-un alt obiect într-o expresie de tip obiect, dar aceasta este în curs de standardizare. Cu o astfel de abordare, ați putea scrie `{...state, ...action}`.

## Construirea DOM-ului

{{index "createElement method", "elt function", [DOM, construction]}}

Unul dintre principalele lucruri pe care le realizează componentele interfeței este crearea unei structuri DOM. Din nou, nu vrem să utilizăm metodele DOM detaliate pentru aceasta, astfel că mai jos este o versiune mai extinsă a funcției `elt`:

```{includeCode: true}
function elt(type, props, ...children) {
  let dom = document.createElement(type);
  if (props) Object.assign(dom, props);
  for (let child of children) {
    if (typeof child != "string") dom.appendChild(child);
    else dom.appendChild(document.createTextNode(child));
  }
  return dom;
}
```

{{index "setAttribute method", "attribute", "onclick property", "click event", "event handling"}}

Principala diferență dintre această versiune și cea pe care am utilizat-o în [capitolul ?](game#domdisplay) este că se asignează _proprietăți_ către nodurile DOM, nu către _atribute_. Aceasta înseamnă că nu o putem utiliza pentru a seta atribute dar o _putem_ utiliza pentru a seta proprietăți ale căror valori nu sunt stringuri, cum ar fi `onclick` care poate fi setat la o funcție pentru a înregistra un handler pentru evenimentul click.

{{index "button (HTML tag)"}}

Astfel, este permis următorul stil de a înregistra handlere de evenimente:

```{lang: "text/html"}
<body>
  <script>
    document.body.appendChild(elt("button", {
      onclick: () => console.log("click")
    }, "The button"));
  </script>
</body>
```

## Canvasul

Prima componentă pe care o vom defini este partea din interfață care afișează imaginea ca un grid de pătrate colorate. Această componentă este responsabilă pentru două lucruri: afișarea unei imagini și comunicarea evenimentelor asociate cu pointerul mouseului către restul aplicației.

{{index "PictureCanvas class", "callback function", "scale constant", "canvas (HTML tag)", "mousedown event", "touchstart event", [state, "of application"]}}

Astfel, îl putem defini ca o componentă care este conștientă doar despre imaginea curentă, nu întreaga stare a aplicației. Deoarece nu cunoaște cum funcționează aplicația ca și întreg, nu poate expedia direct acțiunile. Mai degrabă, când răspunde la evenimente ale pointerului mouseului, va apela o funcție de callback furnizată de către codul care l-a creat, care va gestiona părțile specifice aplicației.

```{includeCode: true}
const scale = 10;

class PictureCanvas {
  constructor(picture, pointerDown) {
    this.dom = elt("canvas", {
      onmousedown: event => this.mouse(event, pointerDown),
      ontouchstart: event => this.touch(event, pointerDown)
    });
    this.syncState(picture);
  }
  syncState(picture) {
    if (this.picture == picture) return;
    this.picture = picture;
    drawPicture(this.picture, this.dom, scale);
  }
}
```

{{index "syncState method", efficiency}}

Desenăm fiecare pixel ca un pătrat 10x10, fapt determinat de către constanta `scale`. Pentru a evita efortul inutil, componenta urmărește imaginea curentă și redesenează doar atunci când `syncState` primește o nouă imagine.

{{index "drawPicture function"}}

Funcția propriu-zisă de desenare setează dimensiunea canvasului pe baza scalei și a dimensiunii imaginii și apoi o umple cu o serie de pătrate, câte unul pentru fiecare pixel.

```{includeCode: true}
function drawPicture(picture, canvas, scale) {
  canvas.width = picture.width * scale;
  canvas.height = picture.height * scale;
  let cx = canvas.getContext("2d");

  for (let y = 0; y < picture.height; y++) {
    for (let x = 0; x < picture.width; x++) {
      cx.fillStyle = picture.pixel(x, y);
      cx.fillRect(x * scale, y * scale, scale, scale);
    }
  }
}
```

{{index "mousedown event", "mousemove event", "button property", "buttons property", "pointerPosition function"}}

Când se apasă butonul stânga al mouseului cu mouseul plasat peste canvas, componenta apelează callback-ul `pointerDown`, transmițându-i poziția pixelului pe care s-a executat click - în coordonate imagine. Aceasta va fi utilizată pentru a implementa interacțiunea cu imaginea folosind mouse-ul. Callback-ul ar putea returna o altă funcție de callback pentru a fi notificat când pointerul a fost mutat pe un alt pixel în timp ce butonul mouse-ului era apăsat.

```{includeCode: true}
PictureCanvas.prototype.mouse = function(downEvent, onDown) {
  if (downEvent.button != 0) return;
  let pos = pointerPosition(downEvent, this.dom);
  let onMove = onDown(pos);
  if (!onMove) return;
  let move = moveEvent => {
    if (moveEvent.buttons == 0) {
      this.dom.removeEventListener("mousemove", move);
    } else {
      let newPos = pointerPosition(moveEvent, this.dom);
      if (newPos.x == pos.x && newPos.y == pos.y) return;
      pos = newPos;
      onMove(newPos);
    }
  };
  this.dom.addEventListener("mousemove", move);
};

function pointerPosition(pos, domNode) {
  let rect = domNode.getBoundingClientRect();
  return {x: Math.floor((pos.clientX - rect.left) / scale),
          y: Math.floor((pos.clientY - rect.top) / scale)};
}
```

{{index "getBoundingClientRect method", "clientX property", "clientY property"}}

Deoarece știm dimensiunea pixelilor și putem apela `getBoundingClientRect` pentru a determina poziția canvasului pe ecran, putem trece din coordonatele evenimentului de mouse (`clientX` și `clientY`) în coordonatele imaginii. Acestea sunt întotdeauna rotunjite în jos pentru a ne referi la un anume pixel.

{{index "touchstart event", "touchmove event", "preventDefault method"}}

Cu evenimentele pentru atingeri trebuie să facem ceva similar, dar utilizând alte evenimente și asigurându-ne că utilizăm `preventDefault` pentru evenimentul `"touchstart"` pentru a preveni panoramarea.

```{includeCode: true}
PictureCanvas.prototype.touch = function(startEvent,
                                         onDown) {
  let pos = pointerPosition(startEvent.touches[0], this.dom);
  let onMove = onDown(pos);
  startEvent.preventDefault();
  if (!onMove) return;
  let move = moveEvent => {
    let newPos = pointerPosition(moveEvent.touches[0],
                                 this.dom);
    if (newPos.x == pos.x && newPos.y == pos.y) return;
    pos = newPos;
    onMove(newPos);
  };
  let end = () => {
    this.dom.removeEventListener("touchmove", move);
    this.dom.removeEventListener("touchend", end);
  };
  this.dom.addEventListener("touchmove", move);
  this.dom.addEventListener("touchend", end);
};
```

{{index "touches property", "clientX property", "clientY property"}}

Pentru evenimentele de atingere, `clientX` și `clientY` nu sunt disponibile direct în obiectul evenimentului, dar putem utiliza coordonatele primului obiect din proprietatea `touches`.

## Aplicația

Pentru a putea construi aplicația bucată cu bucată, vom implementa componenta principală ca o carcasă în jurul canvasului și un set dinamic de instrumente și controale pe care le transmitem constructorului său.

_Controalele_ sunt elementele interfeței care apar sub imagine. Ele vor fi livrate ca un array de constructori pentru componente.

{{index "br (HTML tag)", "flood fill", "select (HTML tag)", "PixelEditor class", dispatch}} 

_Instrumentele_ vor executa operații cum ar fi desenarea pixelilor sau umplerea unei arii. Aplicația afișează setul de instrumente disponibile sub forma unui câmp `<select>`.  Instrumentul curent selectat determină ce se întâmplă când utilizatorul interacționează cu imaginea cu ajutorul unui dispozitiv de tipul mouse-ului. Setul de instrumente disponibile este furnizat ca un obiect care mapează numele care apar în câmpul drop-down cu funcții care implementează instrumentele. Asemenea funcții preiau poziția imaginii, o stare curentă a aplicației, și o funcție `dispatch`, ca și argumente. Ele ar putea returna un handler pentru mișcare care va fi apelat cu o nouă poziție și starea curentă atunci când indicatorul mouseului sau altui dispozitiv de indicare se mută la un alt pixel.

```{includeCode: true}
class PixelEditor {
  constructor(state, config) {
    let {tools, controls, dispatch} = config;
    this.state = state;

    this.canvas = new PictureCanvas(state.picture, pos => {
      let tool = tools[this.state.tool];
      let onMove = tool(pos, this.state, dispatch);
      if (onMove) return pos => onMove(pos, this.state);
    });
    this.controls = controls.map(
      Control => new Control(state, config));
    this.dom = elt("div", {}, this.canvas.dom, elt("br"),
                   ...this.controls.reduce(
                     (a, c) => a.concat(" ", c.dom), []));
  }
  syncState(state) {
    this.state = state;
    this.canvas.syncState(state.picture);
    for (let ctrl of this.controls) ctrl.syncState(state);
  }
}
```

Handlerul pentru pointer transmis către `PictureCanvas` apelează instrumentul selectat cu argumentele corespunzătoare și, dacă acesta returnează un handler petnru mișcare, se adaptează pentru a recepționa și starea.

{{index "reduce method", "map method", [whitespace, "in HTML"], "syncState method"}}

Toate controalele sunt construite și stocate în `this.controls` astfel încât for putea fi actualizate când starea aplicației se modifică. Apelul către `reduce` introduce spații între elementele DOM asociate controalelor. În acest fel, elementele nu vor mai părea atât de înghesuite.

{{index "select (HTML tag)", "change event", "ToolSelect class", "syncState method"}}

Primul control este meniul pentru selectarea instrumentului. El crează un element `<select>` cu câte o opțiune pentru fiecare instrument și setează un handler de eveniment `"change"`, care actualizează starea aplicație de fiecare dată când utilizatorul selectează un instrument diferit.

```{includeCode: true}
class ToolSelect {
  constructor(state, {tools, dispatch}) {
    this.select = elt("select", {
      onchange: () => dispatch({tool: this.select.value})
    }, ...Object.keys(tools).map(name => elt("option", {
      selected: name == state.tool
    }, name)));
    this.dom = elt("label", null, "🖌 Tool: ", this.select);
  }
  syncState(state) { this.select.value = state.tool; }
}
```

{{index "label (HTML tag)"}}

Prin împachetarea textului etichetei și a câmpului într-un element `<label>`, instruim browserul că eticheta aparține unui câmp, astfel încât, de exemplu, puteți efectua click pe etichetă pentru a transfera focusul în acel câmp.

{{index "color field", "input (HTML tag)"}}

We also need to be able to change the color, so let's add a control for that. An HTML `<input>` element with a `type` attribute of `color` gives us a form field that is specialized for selecting colors. Such a field's value is always a CSS color code in `"#RRGGBB"` format (red, green, and blue components, two digits per color). The browser will show a ((color picker)) interface when the user interacts with it.

{{if book

În funcție de browser, selectorul pentru culoare ar putea arăta cam așa:

{{figure {url: "img/color-field.png", alt: "A color field", width: "6cm"}}}

if}}

{{index "ColorSelect class", "syncState method"}}

Acest control crează un asemenea câmp și îl leagă pentru a-l sincroniza cu proprietatea `color` a stării aplicației:

```{includeCode: true}
class ColorSelect {
  constructor(state, {dispatch}) {
    this.input = elt("input", {
      type: "color",
      value: state.color,
      onchange: () => dispatch({color: this.input.value})
    });
    this.dom = elt("label", null, "🎨 Color: ", this.input);
  }
  syncState(state) { this.input.value = state.color; }
}
```

## Instrumente de desenare

Înainte de a putea desena, trebuie să implementăm instrumentele care vor controla funcționalitatea pentru evenimentele de mouse sau de atingere a canvasului.

{{index "draw function"}}

Cel mai elementar instrument este instrumentul de desenare, care modifică orice pixel pe care efectuați click sau tap la culoarea selectată curent. El expediază o acțiune care actualizează imaginea la o versiune în care pixelul indicat primește culoarea selectată curent.

```{includeCode: true}
function draw(pos, state, dispatch) {
  function drawPixel({x, y}, state) {
    let drawn = {x, y, color: state.color};
    dispatch({picture: state.picture.draw([drawn])});
  }
  drawPixel(pos, state);
  return drawPixel;
}
```

Funcția apelează imediat funcția `drawPixel` dar o și returnează, pentru a fi apelată din nou pentru noi pixeli sau când utilizatorul trage sau glisează peste imagine.

{{index "rectangle function"}}

Pentru a desena figuri mai mari, poate fi util să putem crea rapid dreptunghiuri. Instrumentul `rectangle` va desena un dreptunghi din punctul de început al tragerii până în punctul în care terminați tragerea.

```{includeCode: true}
function rectangle(start, state, dispatch) {
  function drawRectangle(pos) {
    let xStart = Math.min(start.x, pos.x);
    let yStart = Math.min(start.y, pos.y);
    let xEnd = Math.max(start.x, pos.x);
    let yEnd = Math.max(start.y, pos.y);
    let drawn = [];
    for (let y = yStart; y <= yEnd; y++) {
      for (let x = xStart; x <= xEnd; x++) {
        drawn.push({x, y, color: state.color});
      }
    }
    dispatch({picture: state.picture.draw(drawn)});
  }
  drawRectangle(start);
  return drawRectangle;
}
```

{{index "persistent data structure", [state, persistence]}}

Un detaliu important în această implementare este că, atunci când tragem, dreptunghiul este redesenat pe imagine, din starea _originală_. In acest fel, puteți să măriți sau să micșorați dreptunghiul în timp ce îl creați, fără ca dreptunghiurile intermediare să rămână pe imaginea finală. Acesta este unul dintre motivele pentru care obiectele imutabile sunt utile - vom vedea un altul puțin mai târziu.

Implementarea umplerii este mai complexă. Acesta este un instrument care umple pixelul de sub pointer și toți pixelii adiacenți ce au aceeași culoare. "Adiacent" înseamnă adiacent pe orizontală sau verticală, nu și direcția diagonală. Imaginea de mai jos ilustrează setul de pixeli colorați când instrumentul de umplere este utilizat pentru pixelul marcat:

{{figure {url: "img/flood-grid.svg", alt: "A pixel grid showing the area filled by a flood fill operation", width: "6cm"}}}

{{index "fill function"}}

Interesant, modul în care realizăm operația seamănă cu codul pentru determinarea rutei din [capitolul ?](robot). În timp ce acel cod căuta o rută într-un graf, acest cod caută într-un grid pentru a determina toți pixelii "conectați". Problema urmăririi unui set ce se poate ramifica de rute posibile este similară.

```{includeCode: true}
const around = [{dx: -1, dy: 0}, {dx: 1, dy: 0},
                {dx: 0, dy: -1}, {dx: 0, dy: 1}];

function fill({x, y}, state, dispatch) {
  let targetColor = state.picture.pixel(x, y);
  let drawn = [{x, y, color: state.color}];
  for (let done = 0; done < drawn.length; done++) {
    for (let {dx, dy} of around) {
      let x = drawn[done].x + dx, y = drawn[done].y + dy;
      if (x >= 0 && x < state.picture.width &&
          y >= 0 && y < state.picture.height &&
          state.picture.pixel(x, y) == targetColor &&
          !drawn.some(p => p.x == x && p.y == y)) {
        drawn.push({x, y, color: state.color});
      }
    }
  }
  dispatch({picture: state.picture.draw(drawn)});
}
```

Array-ul de pixeli desenați se dublează pe măsură ce se execută funcția. Pentru fiecare pixel atins, trebuie să verificăm dacă unul dintre pixelii adiacenți are aceeați culoare și nu a fost redesenat deja. Contorul buclei rămâne în urma lungimii array-ului `drawn` pe măsură ce se adaugă noi pixeli. Orice pixeli anteriori trebuie să fie explorați. Când ajunge să fie egal cu lungimea, nu mai există pixeli care trebuie explorați și funcția se încheie.

{{index "pick function"}}

Ultimul instrument este un selector pentru culori, care permite selecția unei culori din imagine pentru a o utiliza ca și culoare curentă de desenare.

```{includeCode: true}
function pick(pos, state, dispatch) {
  dispatch({color: state.picture.pixel(pos.x, pos.y)});
}
```

{{if interactive

Acum ne putem testa apicația!

```{lang: "text/html"}
<div></div>
<script>
  let state = {
    tool: "draw",
    color: "#000000",
    picture: Picture.empty(60, 30, "#f0f0f0")
  };
  let app = new PixelEditor(state, {
    tools: {draw, fill, rectangle, pick},
    controls: [ToolSelect, ColorSelect],
    dispatch(action) {
      state = updateState(state, action);
      app.syncState(state);
    }
  });
  document.querySelector("div").appendChild(app.dom);
</script>
```

if}}

## Salvarea și încărcarea

{{index "SaveButton class", "drawPicture function", [file, image]}}

După ce ne-am desenat capodopera, o să vrem să o salvăm. Trebuie să adăugăm un buton pentru descărcarea imaginii curente ca și fișier-imagine. Controlul de mai jos crează acest buton:

```{includeCode: true}
class SaveButton {
  constructor(state) {
    this.picture = state.picture;
    this.dom = elt("button", {
      onclick: () => this.save()
    }, "💾 Save");
  }
  save() {
    let canvas = elt("canvas");
    drawPicture(this.picture, canvas, 1);
    let link = elt("a", {
      href: canvas.toDataURL(),
      download: "pixelart.png"
    });
    document.body.appendChild(link);
    link.click();
    link.remove();
  }
  syncState(state) { this.picture = state.picture; }
}
```

{{index "canvas (HTML tag)"}}

Componenta urmărește imaginea curentă astfel încât o poate accesa pentru salvare. Pentru a crea fișierul imagine, utilizează un element `<canvas>` pe care desenează imaginea la scară 1:1 (pixel-la-pixel). 

{{index "toDataURL method", "data URL"}}

Metoda `toDataURL` a elementului canvas crează un URL care începe cu `data:`. Spre deosebire de URL-urile `http:` și `https:`, URL-urile `data` conțin toată resursa în URL. Ele sunt de regulă foarte lungi, dar ne permit să construim legături funcționale către imagini, chiar în browser.

{{index "a (HTML tag)", "download attribute"}}

Pentru descărcarea propriu-zisă a imaginii, creem o legătură care indică spre acel URL și are un atribut `download`. Asemenea legături, când sunt activate prin click, determină browserul să afișeze un dialog pentru salvarea fișierului. Adăugăm acea legătură în document, simulăm un click și apoi o eliminăm.

Puteți realiza o mulțime de lucruri cu tehnologia browserelor dar, uneori, modul în care trebuie să o folosiți este ciudat.

{{index "LoadButton class", control, [file, image]}}

Dar devine și mai ciudat. Vrem să putem să încărcăm o imagine existentă în aplicație. Pentru aceasta, definim încă o componentă pentru un buton.

```{includeCode: true}
class LoadButton {
  constructor(_, {dispatch}) {
    this.dom = elt("button", {
      onclick: () => startLoad(dispatch)
    }, "📁 Load");
  }
  syncState() {}
}

function startLoad(dispatch) {
  let input = elt("input", {
    type: "file",
    onchange: () => finishLoad(input.files[0], dispatch)
  });
  document.body.appendChild(input);
  input.click();
  input.remove();
}
```

{{index [file, access], "input (HTML tag)"}}

Pentru a avea acces la un fișier din computerul utilizatorului, trebuie ca utilizatorul să selecteze mai întâi fișierul cu ajutorul unui câmp pentru fișiere. Dar nu vreau ca butonul de încărcare să arate ca un câmp de încărcare a fișierelor, astfel că voi crea câmpul de încărcare atunci când se execută click pe buton și apopi voi pretinde că acest câmp nou creat a recepționat click-ul.

{{index "FileReader class", "img (HTML tag)", "readAsDataURL method", "Picture class"}}

Când utilizatorul a selectat un fișier, putem utiliza `FileReader` pentru a-i accesa conținutul, din nou ca un URL `data`. Apoi acel URL va putea fi folosit pentru a crea un element `<img>` dar, deoarece nu putem avea acces direc tla pixelii dintr-o asemenea imagine, nu putem crea un obiect `Picture` din aceasta.

```{includeCode: true}
function finishLoad(file, dispatch) {
  if (file == null) return;
  let reader = new FileReader();
  reader.addEventListener("load", () => {
    let image = elt("img", {
      onload: () => dispatch({
        picture: pictureFromImage(image)
      }),
      src: reader.result
    });
  });
  reader.readAsDataURL(file);
}
```

{{index "canvas (HTML tag)", "getImageData method", "pictureFromImage function"}}

Pentru a avea acces la pixeli, mai întâi trebuie să desenăm imaginea pe un element `<canvas>`. Contextul canvasului are o metodă `getImageData` care permite scriptului să citească pixelii. Prin urmare, după ce pictura apare pe canvas, o putem accesa și să construim un obiect `Picture`.

```{includeCode: true}
function pictureFromImage(image) {
  let width = Math.min(100, image.width);
  let height = Math.min(100, image.height);
  let canvas = elt("canvas", {width, height});
  let cx = canvas.getContext("2d");
  cx.drawImage(image, 0, 0);
  let pixels = [];
  let {data} = cx.getImageData(0, 0, width, height);

  function hex(n) {
    return n.toString(16).padStart(2, "0");
  }
  for (let i = 0; i < data.length; i += 4) {
    let [r, g, b] = data.slice(i, i + 3);
    pixels.push("#" + hex(r) + hex(g) + hex(b));
  }
  return new Picture(width, height, pixels);
}
```

Vom limita dimensiunea imaginilor la 100 x 100 pixeli deoarece mai mult de atât va fi _imens_ pentru afișajul nostru și ar putea încetini interfața.

{{index "getImageData method", color, transparency}}

Proprietatea `data` a obiectului returnat de către `getImageData` este un array de componente de culoare. Pentru fiecare pixel din dreptunghiul specificat prin argumente, vor exista 4 valori, care reprezintă componentele roșu, verde, albastru și _alpha_ ale culorii pixelului, ca numere între 0 și 255. Partea _alpha_ reprezintă opacitatea/transparența - valoarea 0 înseamnă că pixelul este complet transparent iar 255 se referă la opacitate completă. Pentru scopul nostru, puttem să o ignorăm.

{{index "hexadecimal number", "toString method"}}

Cele două cifre hexazecimale pe componentă, așa cum le utilizăm în notația noastră pentru culori, corespund exact domeniului 0-255 - două cifre în baza 16 ne permit să reprezentăm 16^2^ = 256 numere diferite. Metoda `toString` pentru numere poate primi o bază ca și argument, astfel încât `n.toString(16)` va produce o reprezentare de tip string în baza 16. Trebuie să ne asigurăm că fiecare număr conține exact două cifre, astfel încât funcția helper `hex` apelează `padStart` pentru a adăuga un zero nesemnfiicativ dacă este nevoie.

Acum putem încărca și salva! Astfel că ne mai rămâne o singură funcționalitate înainte de a termina.

## Istoricul "undo"

Jumătate din procesul de editare înseamnă mici greșeli care apoi trebuie corectate. Astfel că o funcționalitate importantă într-un program de desenare este istoricul "undo".

{{index "persistent data structure", [state, "of application"]}}

Pentru a putea să inversăm modificările, trebuie să stocăm versiunile anterioare ale imaginii. Deoarece aceasta este o valoare imutabilă, este destul de ușor. Dar este necesar un câmp adițional în starea aplicației.

{{index "done property"}}

Vom adăuga un array `done` pentru a stoca versiunile anterioare ale imaginii. Menținerea acestei proprietăți necesită o funcție mai complicată de actualizare a stării care adaugă imaginii în array.

{{index "doneAt property", "historyUpdateState function", "Date.now function"}}

Dar nu vrem să memorăm orice modificare, doar modificările de la anumite momente de timp. Pentru a putea realiza acest lucru, avem nevoie de o a doua proprietate, `doneAt`, care urmărește momentul de timp la care am stocat pentru ultima oară o imagine în istoric.

```{includeCode: true}
function historyUpdateState(state, action) {
  if (action.undo == true) {
    if (state.done.length == 0) return state;
    return Object.assign({}, state, {
      picture: state.done[0],
      done: state.done.slice(1),
      doneAt: 0
    });
  } else if (action.picture &&
             state.doneAt < Date.now() - 1000) {
    return Object.assign({}, state, action, {
      done: [state.picture, ...state.done],
      doneAt: Date.now()
    });
  } else {
    return Object.assign({}, state, action);
  }
}
```

{{index "undo history"}}

Atunci când acțiunea este o acțiune "undo", funcția preia cea mai recentă imagine din istoric și construiește imaginea curentă pe baza ei. Setează `doneAt` la zero astfel încât se garantează că va stoca imaginea în istoric si se poate reveni la ea la un moment ulterior.

Altfel, dacă acțiunea conține o nouă imagine și ultima dată când am stocat ceva este cu mai mult de o seecundă în urmă (1000 milisecunde), proprietățile `done` și `doneAt` sunt actualizate pentru a memora imaginea anterioară.

{{index "UndoButton class", control}}

Butonul "undo" nu face mare lucru. Doar expediază acțiuni "undo" când se realizează un click și se dezactivează dacă nu există nimic de refăcut.

```{includeCode: true}
class UndoButton {
  constructor(state, {dispatch}) {
    this.dom = elt("button", {
      onclick: () => dispatch({undo: true}),
      disabled: state.done.length == 0
    }, "⮪ Undo");
  }
  syncState(state) {
    this.dom.disabled = state.done.length == 0;
  }
}
```

## Hai să desenăm

{{index "PixelEditor class", "startState constant", "baseTools constant", "baseControls constant", "startPixelEditor function"}}

Pentru a seta aplicația, trebuie să creăm o stare, un set de instrumente, un set de controale și o funcție pentru expedierea acțiunilor. Le putem transmite constructorului `PixelEditor` pentru a crea componenta principală. Deoarece va trebui să creem mai multe editoare în exerciții, vom începe prin a defini câteva bindinguri.

```{includeCode: true}
const startState = {
  tool: "draw",
  color: "#000000",
  picture: Picture.empty(60, 30, "#f0f0f0"),
  done: [],
  doneAt: 0
};

const baseTools = {draw, fill, rectangle, pick};

const baseControls = [
  ToolSelect, ColorSelect, SaveButton, LoadButton, UndoButton
];

function startPixelEditor({state = startState,
                           tools = baseTools,
                           controls = baseControls}) {
  let app = new PixelEditor(state, {
    tools,
    controls,
    dispatch(action) {
      state = historyUpdateState(state, action);
      app.syncState(state);
    }
  });
  return app.dom;
}
```

{{index "destructuring binding", "= operator", [property, access]}}

Când destructurăm un obiect sau un array, putem utiliza `=` după numele bindingului pentru a atribui bindingului o valoare implicită, care este utilizată atunci când proprietatea lipsește sau memorează `undefined`. Funcția `startPixelEditor` utilizează această tehnică pentru a accepta un obiect cu câteva proprietăți adiționale ca și argument. Dacă nu transmiteți o proprietate `tools`, de exemplu, `tools` va fi legat de `baseTools`.

Așa putem afișa editorul propriu-zis pe ecran:

```{lang: "text/html", startCode: true}
<div></div>
<script>
  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

## De ce este atat de greu?

Tehnologia browserelor este uimitoare. Ea ne oferă un set puternic de blocuri pentru construcția interfeței, modalități de stilizare și manipulare a acestora, și instrumente pentru a inspecta și repara aplicațiile. Software-ul pe care îl scrieți pentru browser poate fi rulat pe aproape orice computer și smartphone de pe planetă.

În același timp, tehnologia browserelor este ridicolă. Trebuie să învățați un mare număr de trick-uri stupide și fapte obscure pentru a o stăpâni, iar modelul de programare implicit pe care îl expune este atât de problematic încât majoritatea programatorilor preferă să îl împacheteze ăn mai multe straturi de abstractizări, decât să se confrunte cu el direct.

{{index standard, evolution}}

Deși situația se îmbunătățește evident, progresul are loc în principal sub forma adăugării și mai multor elemente pentru a adresa deficiențele - ceea ce crează și mai multă complexitate. O funcționalitate utilizată de un milion de website-uri nu e ușor de înlocuit. Chiar dacă ar fi posibil, ar fi greu de decis cu ce să fie înlocuită.

{{index "social factors", "economic factors", history}}

Tehnologia nu există în vid - suntem constrânși de către instrumentele noastre și de către factorii sociali, economici și istorici care le-au produs. Această situație poate fi deranjantă, dar în general este mai productiv să încercăm să construim o înțelegere bună a modului în care funcționează realitatea tehnică _existentă_ - și de ce lucrurile sunt așa cum sunt - decât să fim vehemenți împotriva ei și săîncercăm să construim o altă realitate.

Noile abstracții _pot_ fi utile. Modelul de componentă și convenția pentru fluxul datelor utilizată în acest capitol sunt o forma brută a acestei abordări. Așa cum am menționat, există librării care încearcă să facă mai plăcută activitatea de programare a interfeței cu utilizatorul. La momentul în care am scris această carte, [React](https://reactjs.org/) și [Angular](https://angular.io/) sunt alegeri populare, dar există o întreagă industrie pentru asemenea frameworkuri. Dacă sunteți interesați în programarea apălicațiilor web, vă recomand să investigați câteva opțiuni pentru a le înțelege cum funcționează și ce beneficii oferă.

## Exerciții

Mai există loc pentru îmbunătățiri în programul nostru. Haideți să mai adăugăm câteva funcționalități ca și exercițiu.

### Legături cu tastatura

{{index "keyboard bindings (exercise)"}}

Adăugați taste de accelerare pentru aplicație. Prima literă a numelui unui instrument selectează instrumentul, iar [control]{keyname}-Z sau [command]{keyname}-Z activează "undo".

{{index "PixelEditor class", "tabindex attribute", "elt function", "keydown event"}}

Pentru aceasta, modificați componenta `PixelEditor`. Adăugați proprietatea `tabIndex` cu valoarea 0 pentru elementul `<div>` astfel încât acesta să poată recepționa focusul de la tastatură. Observați că proprietatea corespunzătoare _atributului_ `tabindex` se numește `tabIndex`, cu I mare, iar funcția noastră `elt` așteaptă nume de proprietăți. Înregistrați handlerele pentru evenimente de tastatură direct pe acest element. Aceasta înseamnă că trebuie să efectuați click, touch sau tab în aplicație înainte de a putea interacționa cu ea prin intermediul tastaturii.

{{index "ctrlKey property", "metaKey property", "control key", "command key"}}

Vă reamintesc că evenimentele de la tastatură au proprietățile `ctrlKey` și `metaKey` (pentru tasta [command]{keyname} pentru Mac) pe care le puteți utiliza pentru a verifica dacă aceste taste sunt apăsate.

{{if interactive

```{test: no, lang: "text/html"}
<div></div>
<script>
  // The original PixelEditor class. Extend the constructor.
  class PixelEditor {
    constructor(state, config) {
      let {tools, controls, dispatch} = config;
      this.state = state;

      this.canvas = new PictureCanvas(state.picture, pos => {
        let tool = tools[this.state.tool];
        let onMove = tool(pos, this.state, dispatch);
        if (onMove) {
          return pos => onMove(pos, this.state, dispatch);
        }
      });
      this.controls = controls.map(
        Control => new Control(state, config));
      this.dom = elt("div", {}, this.canvas.dom, elt("br"),
                     ...this.controls.reduce(
                       (a, c) => a.concat(" ", c.dom), []));
    }
    syncState(state) {
      this.state = state;
      this.canvas.syncState(state.picture);
      for (let ctrl of this.controls) ctrl.syncState(state);
    }
  }

  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

if}}

{{hint

{{index "keyboard bindings (exercise)", "key property", "shift key"}}

Proprietatea `key` a evenimentelor pentru taste-literă este litera mică însăși, daca tasta [shift]{keyname} nu este apăsată. Nu ne interesează evenimentele de tastatură cu tasta [shift]{keyname} în acest caz.

{{index "keydown event"}}

Un handler `"keydown"` își poate inspecta obiectul de eveniment pentru a vedea dacă se potrivește peste vreuna din tastele de accelerare. Puteți obține lista primelor litere din obiectul `tools`, deci nu trebuie să le stocați separat.

{{index "preventDefault method"}}

Când evenimentul de tastatură potrivește o tastă de accelerare, apelați `preventDefault` și apoi expediați acțiunea adecvată.

hint}}

### Desenare eficientă

{{index "efficient drawing (exercise)", "canvas (HTML tag)", efficiency}}

În timp ce desenăm, majoritatea operațiilor din aplicația noastră au loc în `drawPicture`. Crearea unei noi stări și actualizarea DOM nu este foarte costisitoare, dar redesenarea tuturor pixelilor de pe canvas este un efort semnificativ.

{{index "syncState method", "PictureCanvas class"}}

Găsiți o modalitate de a rapidiza metoda `syncState` a `PictureCanvas` prin redesenarea doar a acelor pixeli care s-au modificat.

{{index "drawPicture function", compatibility}}

Atenție, `drawPicture` este utilizată și de către butonul de salvare. Dacă o modificați, fie asigurați-vă că modificările nu strică vechea utilizare fie creați o nouă versiune cu un alt nume.

{{index "width property", "height property"}}

De asemenea, remarcați că schimbarea dimensiunilor elementului `<canvas>` prin setarea proprietăților `width` sau `height` conduce la curățarea acestuia (redevine complet transparent).

{{if interactive

```{test: no, lang: "text/html"}
<div></div>
<script>
  // Change this method
  PictureCanvas.prototype.syncState = function(picture) {
    if (this.picture == picture) return;
    this.picture = picture;
    drawPicture(this.picture, this.dom, scale);
  };

  // You may want to use or change this as well
  function drawPicture(picture, canvas, scale) {
    canvas.width = picture.width * scale;
    canvas.height = picture.height * scale;
    let cx = canvas.getContext("2d");

    for (let y = 0; y < picture.height; y++) {
      for (let x = 0; x < picture.width; x++) {
        cx.fillStyle = picture.pixel(x, y);
        cx.fillRect(x * scale, y * scale, scale, scale);
      }
    }
  }

  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

if}}

{{hint

{{index "efficient drawing (exercise)"}}

Acest exercițiu este un bun exemplu pentru cum poate deveni codul mai rapid cu ajutorul structurilor de date imutabile. Deoarece avem la dispoziția atât imaginea veche cât și cea nouă, le putem compara și să redesenăm doar pixelii care șia-u modificat culoarea, ceea ce ne va permite în majoritatea cazurilor să nu redesenăm mai mult de 99% din imagine.

{{index "drawPicture function"}}

Puteți fie să scrieți o funcție nouă, `updatePicture`, fie să transmiteți un argument suplimentar către `drawPicture` , care poate fi undefined sau imaginea anterioară. Pentru fiecare pixel, funcția verifică dacă o imagine anterioară a fost transmisă și dacă da, iar pixelul are aceeași culoare, atunci nu îl redesenăm.

{{index "width property", "height property", "canvas (HTML tag)"}}

Deoarece canvas va fi curățat când își schimbă dimensiunile, ar fi bine să evitați să îi modificați proprietățile `width` și `height` când vechea imagine și cea nouă au aceeași dimensiune. Dacă sunt diferite, ceea ce se poate întâmpla atunci când se încarcă o nouă imagine, puteți seta bindingul ce reprezintă vechea imagine la valoarea null, după ce ați setat dimensiunea canvasului, deoarece în acest caz totți pixelii vor fi redesenați.

hint}}

### Cercuri

{{index "circles (exercise)", dragging}}

Definiți un tool numit `circle` care desenează un cerc umplut. Centrul cercului va fi plasat în punctul în care începe drag sau touch iar raza va fi determinată de distanța de tragere.

{{if interactive

```{test: no, lang: "text/html"}
<div></div>
<script>
  function circle(pos, state, dispatch) {
    // Your code here
  }

  let dom = startPixelEditor({
    tools: Object.assign({}, baseTools, {circle})
  });
  document.querySelector("div").appendChild(dom);
</script>
```

if}}

{{hint

{{index "circles (exercise)", "rectangle function"}}

Vă puteți inspira din instrumentul `rectangle`. Ca și acel instrument, o să doriți să desenați pe imaginea de start, nu pe imaginea curentă, atunci când pointerul se deplasează.

Pentru a determina ce pixeli să colorați, puteți folosi teorama lui Pitagora. Mai întâi determinați distanța dintre poziția curentă a pointerului și poziția de start, apoi alegeți un pătrat cu dimensiunea cel puțin dublă față de raza cercului, centrat în același punct ca și cercul și colorați pixelii aflați în interiorul cercului.

Asigurați-vă că nu colorați pixeli în afara limitelor imaginii.

hint}}

### Linii corespunzătoare

{{index "proper lines (exercise)", "line drawing"}}

Acesta este un exercițiu mai avansat decât cele două anterioare și va trebui să concepeți o soluție pentru o problemă netrivială. Asigurați-vă că dispuneți de suficient timp și de multă răbdare înainte de a începe să lucrați la acest exercițiu și nu vă lăsați descurajați de eșecurile inițiale.

{{index "draw function", "mousemove event", "touchmove event"}}

În majoritatea browserelor, când selectați instrumentul `draw` și desenați rapid peste imagine, nu veți obține o linie continuă. Mai degrabă veți avea puncte cu spații între ele, deoarece evenimentele `"mousemove"` sau `"touchmove"` nu se declanșează suficient de repede pentru a atinge fiecare pixel.

Îmbunătățiți instrumentul `draw` pentru a desena o linie completă. Aceasta înseamnă că funcția ce gestionează mișcarea ar trebui să rețină poziția anterioară și să o conecteze cu poziția curentă.

Pentru a reuși, deoarece pixelii pot fi la o distanță arbitrară unul de altul, trebuie să scrieți o funcție generală de desenare a unei linii.

O linie între doi pixeli este un lanț conectat de pixeli, cât mai drept posibil, de la început până la sfârșit. Pixelii adiacenți diagonal se consideră conectați. Astfel că o linie oblică ar trebui să arate ca în imaginea din stânga, nu ca în cea din dreapta.

{{figure {url: "img/line-grid.svg", alt: "Two pixelated lines, one light, skipping across pixels diagonally, and one heavy, with all pixels connected horizontally or vertically", width: "6cm"}}}

În final, dacă avem cod care desenează o linie între două puncte oarecare, putem să îl utilizăm pentru a defini un instrument `line`, care desenează o linie între începutul și sfârșitul unei operații de "drag".

{{if interactive

```{test: no, lang: "text/html"}
<div></div>
<script>
  // The old draw tool. Rewrite this.
  function draw(pos, state, dispatch) {
    function drawPixel({x, y}, state) {
      let drawn = {x, y, color: state.color};
      dispatch({picture: state.picture.draw([drawn])});
    }
    drawPixel(pos, state);
    return drawPixel;
  }

  function line(pos, state, dispatch) {
    // Your code here
  }

  let dom = startPixelEditor({
    tools: {draw, line, fill, rectangle, pick}
  });
  document.querySelector("div").appendChild(dom);
</script>
```

if}}

{{hint

{{index "proper lines (exercise)", "line drawing"}}

Problema desenării unei linii din pixeli este că ea ascunde de fapt patru probleme similare dar ușor diferite. Desenarea unei linii orizontale de la stânga la dreapta este ușoară - iterați coordonata x și desenați câte un pixel la fiecare pas. Dacă linia are o pantă ușoară (mai mică de 45 grade), puteți interpola coordonata y cu ajutorul pantei. Aveți nevoie tot de un pixel pe direcția x, iar poziția y a pixelului poate fi determinată cu ajutorul pantei.

Dar dacă panta depășește 45 de grade, trebuie să inversați modul în care folosiți coordonatele. Acum va trebui să alegeți câte un pixel pe direcția y, deoarece linia merge pe verticală mai mult decât pe orizontală. Iar cînd depășiți 135 grade, va trebui să reveniți la iterarea după coordonata x, dar de la dreapta la stânga.

De fapt nu va trebui să scrieți patru bucle. Deoarece desenarea unei linii din punctul _A_ în punctul _B_ este tot una cu desenarea ei din punctul _B_ în punctul _A_, puteți interschimba popzițiile de început și de sfârșit pentru liniile care merg de la dreapta la stânga și să le tratați ca și cum ar merge de la stânga la dreapta.

Înseamnă că aveți nevoie de două bucle diferite. Primul lucru pe care ar trebui să îl facă funcția voastră de desenare a ueni linii ar fi să verifice dacă diferența dintre coordonatele xx este mai mare decât diferența dintre coordonatele y. Dacă da, linia este mai orizontală, dacă nu, linia este mai verticală.

{{index "Math.abs function", "absolute value"}}

Comparați valorile absolute ale diferentelor dintre coordonate, folosind `Math.abs`.

{{index "swapping bindings"}}

După ce ați identificat după care axa trebuie să iterați, puteți verifica dacă punctul de start are o coordonată mai mare pe acea axă decât cel de stop și să le interschimbași dacă este necesar. O modalitate rapidă de interschimbare a valorilor din două bindinguri JavaScript este utilizarea destructurării în atribuire, astfel:

```{test: no}
[start, end] = [end, start];
```

{{index rounding}}

Apoi puteți calcula panta liniei și cu ajutorul ei veți putea calcula poziția corectă pe cealaltă axă, pentru fiecare pas pe care îl faceți pe axa principală. Veți putea rula o buclă de-a lungul axei principale și să calculați poziția pe axa secundară și să desenați pixelii la fiecare iterație. Dar nu uitați să rotunjiți coordonatele deoarece metoda `draw` nu răspunde bine pentru coordonate fracționare.

hint}}
