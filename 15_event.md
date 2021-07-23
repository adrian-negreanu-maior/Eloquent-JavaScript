# Gestiunea evenimentelor

{{quote {author: "Marcus Aurelius", title: Meditations, chapter: true}

Poți să îți influențezi mintea - nu evenimentele exterioare. Realizează aceasta și îți vei găsi puterea.

quote}}

{{index stoicism, "Marcus Aurelius", input, timeline}}

{{figure {url: "img/chapter_picture_15.jpg", alt: "Picture a Rube Goldberg machine", chapter: "framed"}}}

Unele programe funcționează direct cu intrarea de la utilizator, cum ar fi acțiunile cu mouse-ul sau tastatura. Acest tip de intrare nu este disponibil sub forma unei structuri de date bine organizate - el apare bucată cu bucată, în timp real și așteptările sunt ca programul să răspundă tot în timp real.

## Event handler-e 

{{index polling, button, "real-time"}}

Imaginați-vă o interfață în care singura modalitate de a afla dacă o tastă a fost apăsată este de a citi starea curentă a acelei chei. Pentru a putea reacționa la apăsările de taste, ar trebui să citiți constant starea tastelor astfel încât să interceptați înainte de eliberarea tastei. Ar fi periculos să efectuați și alte operații consumatoare de timp deoarece ați putea rata o apăsare de tastă.

Unele mașini primitive gestionau astfel intrarea. Un pas înainte ar fi ca harware-ul sau sistemul de operare să notifice apăsarea tastei și să o plaseze într-o coadă. Un program ar putea apoi verifica periodic acea coadă pentru apariția unor noi evenimente și apoi să reacționeze la ceea ce găsește acolo.

{{index responsiveness, "user experience"}}

Desigur, ar trebui să își amintească să consulte coada și să o facă destul de frecvent, deoarece orice întârziere între apăsarea tastei și observarea de către program a acelui eveniment va determina ca sotware-ul să pară ne-reactiv. Această abordare se numește _polling_ și mulți programatori preferă să o evite.

{{index "callback function", "event handling"}}

Un mecanism mai bun presupune ca sistemul să notifice activ codul nostru atunci când apare un eveniment. Browserele fac asta prin a ne permite să înregistrăm funcții ca și _handler-e_ pentru evvenimente specifice.

```{lang: "text/html"}
<p>Click this document to activate the handler.</p>
<script>
  window.addEventListener("click", () => {
    console.log("You knocked?");
  });
</script>
```

{{index "click event", "addEventListener method", "window object", [browser, window]}}

Bindingul `window` este un obiect implicit furnizat de către browser. El reprezintă fereastra browserului care conține documentul. Apelând metoda `addEventListener` putem înregistra cod ce se va executa ori de câte ori evenimentul descris de primul argument apare.

## Evenimente și noduri DOM

{{index "addEventListener method", "event handling", "window object", browser, [DOM, events]}}

Fiecare handler pentru evenimente este înregistrat într-un anumit context. În exemplul anterior, am apelat `addEventListener` pentru obiectul `window` cu scopul de a înregistra un handler pentru întreaga fereastră. O metodă similară este disponibilă și pentru elementele DOM și alte câteva tipuri de obiecte. Codul pentru tratarea evenimentului (event listener) este apelat doar când evenimentul pentru care a fost înregistrat are loc, în contextul obiectului pentru care a fost înregistrat.

```{lang: "text/html"}
<button>Click me</button>
<p>No handler here.</p>
<script>
  let button = document.querySelector("button");
  button.addEventListener("click", () => {
    console.log("Button clicked.");
  });
</script>
```

{{index "click event", "button (HTML tag)"}}

Exemplul de mai sus atașează un handler la nodul ce reprezintă butonul. Click-urile pe buton vor conduce la executarea handlerului, dar clickurile în restul documentului nu vor provoca reacție.

{{index "onclick attribute", encapsulation}}

Utilizarea atributului `onclick` al nodului duce la același efect. Aceasta poate fi utilizată pentru majoritatea tipurilor de evenimente - puteți atașa un handler prin atributul al cărui nume este numele evenimentului, prefixat de `on`.

Dar un nod poate avea un singur atribut `onclick`, ceea ce înseamnă că veți putea înregistra un singur handler. Metoda `addEventListener` vă permite să înregistrați oricâte handlere, astfel că puteți adăuga un nou handler la un element chiar dacă deja a fost înregistrat un handler.

{{index "removeEventListener method"}}

Metoda `removeEventListener`, apelată similar cu `addEventListener`, elimină un handler.

```{lang: "text/html"}
<button>Act-once button</button>
<script>
  let button = document.querySelector("button");
  function once() {
    console.log("Done.");
    button.removeEventListener("click", once);
  }
  button.addEventListener("click", once);
</script>
```

{{index [function, "as value"]}}

Funcția dată pentru `removeEventListener` trebuie să fie aceeași valoare de tip funcție pe care ați transmis-o anterior către `addEventListener`. Deci, pentru a elimina un handler, trebuie ca funcția să aibă un nume (`once` - în exemplu) pentru a putea transmite aceeași valoare către ambele metode.

## Obiectul evenimentului

{{index "button property", "event handling"}}

Deși am ignorat până acum, funcțiile pentru gestiunea evenimentelor primesc un argument: _obiectul evenimentului_. Acest obiect reține informație suplimentară despre eveniment. De exemplu, dacă vrem să știm _care_ dintre butoanele mouse-ului a fost apăsat, putem consulta proprietatea `button` a obiectului evenimentului.

```{lang: "text/html"}
<button>Click me any way you want</button>
<script>
  let button = document.querySelector("button");
  button.addEventListener("mousedown", event => {
    if (event.button == 0) {
      console.log("Left button");
    } else if (event.button == 1) {
      console.log("Middle button");
    } else if (event.button == 2) {
      console.log("Right button");
    }
  });
</script>
```

{{index "event type", "type property"}}

Informația stocată în obiectul evenimentului diferă în funcție de tipul evenimentului. Vom discuta despre aceasta mai târziu în capitolul curent. Proprietatea `type` a obiectului memorează întodeauna un string ce identifică evenimentul (cum ar fi `"click"` sau `"mousedown"`).

## Propagarea

{{index "event propagation", "parent node"}}

{{indexsee bubbling, "event propagation"}}

{{indexsee propagation, "event propagation"}}

Pentru majoritatea tipurilor de evenimente, handlerele înregistrate pe noduri ce au descendenți vor fi notificate și pentru evenimente ce au loc la nivelul descendenților. Dacă se efectuează un click pe un buton din interiorul unui paragraf, event handlerele înregistrate pentru paragraf vor "vedea" și ele evenimentul de click.

{{index "event handling"}}

Dar dacă atât paragraful cât și butonul au un handler, cel mai specific dintre ele (cel pentru buton) va fi primul apelat. Spunem că evenimentul se _propagă_ spre exterior, de la nodul în care a avut loc spre nodul părinte, până la rădăcina documentului. În final, după ce toate handlerele înregistrate pe noduri s-au executat, handlerele înregistrate pentru întreaga fereastră vor avea șansa să răspundă la eveniment.

{{index "stopPropagation method", "click event"}}

În orice moment, un handler de evenimente poate apela metoda `stopPropagation` asupra obiectului evenimentului pentru a nu mai fi notificate handlerele ulterioare cu privire la producerea evenimentului. Aceasta este utilă atunci când, de exemplu, aveți un buton în interiorul altui element pe care se poate da click și nu doriți ca un click pe buton să declanșeze și reacția la click a elementului părinte.

{{index "mousedown event", "pointer event"}}

În exemplul de mai jos înregistrăm handlere pentru evenimentul `"mousedown"` atât pentru buton cât și pentru paragraful care îl conține. Când se execută un click dreapta, handlerul pentru buton apelează `stopPropagation` care previn execuția handlerului pentru paragraf. Când se execută click cu alt buton al mouse-ului, se vor executa ambele handlere.

```{lang: "text/html"}
<p>A paragraph with a <button>button</button>.</p>
<script>
  let para = document.querySelector("p");
  let button = document.querySelector("button");
  para.addEventListener("mousedown", () => {
    console.log("Handler for paragraph.");
  });
  button.addEventListener("mousedown", event => {
    console.log("Handler for button.");
    if (event.button == 2) event.stopPropagation();
  });
</script>
```

{{index "event propagation", "target property"}}

Majoritatea obiectelor pentu evenimente au o proprietate `target` care se referă la nodul de origine. Puteți utiliza această proprietate pentru a vă asigura că nu tratați accidental un eveniment propagat într-un nod în care nu ați dori să îl tratați.

De asemenea, puteți utiliza proprietatea `target` pentru a distribui unei rețele un anumit tip de eveniment. De exemplu, dacă aveți un nod ce conține o listă lungă de butoane, ar putea fi mai convenabil să înregistrați un singur handler pentru click pentru nodul părinte și să utilizați proprietatea `target` pentru a determina dacă un anume buton a fost apăsat, decât să înregistrați handlere la nivelul fiecărui buton în parte.

```{lang: "text/html"}
<button>A</button>
<button>B</button>
<button>C</button>
<script>
  document.body.addEventListener("click", event => {
    if (event.target.nodeName == "BUTTON") {
      console.log("Clicked", event.target.textContent);
    }
  });
</script>
```

## Acțiuni implicite

{{index scrolling, "default behavior", "event handling"}}

Multe evenimente au o acțiune implicită asociată cu ele. Dacă executați click pe un link, veți naviga la adresa linkului. Dacă apăsați săgeata-jos, veți naviga în jos în pagină. Dacă executați click-dreapta vi se afișează un meniu contextual, șamd.

{{index "preventDefault method"}}

Pentru majoritatea tipurilor de evenimente, handlere de eveniment înregistrate se execută _înainte_ de a se executa comportamentul implicit. Dacă handlerul nu dorește să permită execuția acestui comportament predefinit, de regulă deoarece deja s-a tratat evenimentul, se poate apela metoda `preventDefault` asupra obiectului evenimentului.

{{index expectation}}

De exemplu, puteți utiliza aceasta pentru a crea propriile combinații de taste de accelerare sau meniuri contextuale. Dar se poate utiliza și pentru a surprinde neplăcut utilizatorul. De exemplu, linkul de mai jos nu poate fi navigat:

```{lang: "text/html"}
<a href="https://developer.mozilla.org/">MDN</a>
<script>
  let link = document.querySelector("a");
  link.addEventListener("click", event => {
    console.log("Nope.");
    event.preventDefault();
  });
</script>
```

{{index usability}}

Încercați să nu folosiți asemenea cod, dacă nu aveți un motiv foarte bun. Va fi neplăcut pentru utilizator ca pagina să nu se comporte așa cum se așteaptă.

În funcție de browser, anumite evenimente nu pot fi interceptate. De exemplu, în Chrome, combinația de taste pentru închiderea tab-ului curent ([control]{keyname}-W sau [command]{keyname}-W) nu poate fi tratată în JavaScript.

## Evenimente de la tastatură

{{index keyboard, "keydown event", "keyup event", "event handling"}}

Când se apasă o tastă, browserul generează un eveniment `"keydown"`. La eliberarea tastei, se va genera un eveniment `"keyup"`.

```{lang: "text/html", focus: true}
<p>This page turns violet when you hold the V key.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == "v") {
      document.body.style.background = "violet";
    }
  });
  window.addEventListener("keyup", event => {
    if (event.key == "v") {
      document.body.style.background = "";
    }
  });
</script>
```

{{index "repeating key"}}

În pofida numelui, `"keydown"` este generat nu doar când tasta este apăsată fizic. Când mențineți apăsată tasta, evenimentul este generat pentru fiecare _repetiție_ a tastei. Uneori trebuie să fiți atenți la acest aspect. De exemplu, dacă adăugați un buton la DOM când tasta este apăsată și îl eliminați când tasta este eliberată, s-ar putea să adăugați o mulțime de butoane când tasta este menținută apăsată pentru mai mult timp.

{{index "key property"}}

În exemplu am consultat proprietatea `key` a obiectului pentru eveniment pentru a determina despre ce tasta este vorba. Această proprietate memorează un string ce corespunde caracterului pe care l-ar genera apăsarea tastei, pentru majoritatea tastelor. Pentru tastele speciale, cum ar fi [enter]{keyname}, memorează numele tastei (`"Enter"`, în acest caz). Dacă apăsați [shift]{keyname} împreună cu o tastă, numele cheii ar putea fi influențat - `"v"` devine `"V"`, și `"1"` ar putea fi `"!"`, dacă asta produce apăsarea [shift]{keyname}-1 pe tastatură.

{{index "modifier key", "shift key", "control key", "alt key", "meta key", "command key", "ctrlKey property", "shiftKey property", "altKey property", "metaKey property"}}

Cheile modificatoare, cum sunt [shift]{keyname}, [control]{keyname}, [alt]{keyname} și [meta]{keyname} ([command]{keyname} pe Mac) generează evenimente ca și cheile normale. Dar când verificați combinațiile de taste, puteți consulta proprietățile `shiftKey`, `ctrlKey`, `altKey` și `metaKey` ale evenimentelor pentru tastatură și mouse.

```{lang: "text/html", focus: true}
<p>Press Control-Space to continue.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == " " && event.ctrlKey) {
      console.log("Continuing!");
    }
  });
</script>
```

{{index "button (HTML tag)", "tabindex attribute", [DOM, events]}}

Nodul DOM de origine al unui eveniment de tastatură depinde de elementul care are _focus-ul_ atunci când tasta este apăsată. Majoritatea elementelor nu pot deține focusul decât dacă le asociați un atribut `tabindex`, dar linkurile, butoanele și câmpurile formularelor au această posibilitate. Vom reveni la câmpurile din formulare în [capitolul ?](http#forms). Când nici un element nu are focusul,  `document.body` este nodul țintă pentru evenimentele tastaturii.

Când utilizatorul introduce text, utilizarea evenimentelor pentru taste cu scopul de a identifica ce se introduce este problematică. Unele platforme, cum ar fi tastatura virtuală de pe telefoanele Android, nu generează evenimente de tastatură. Dar chiar și atunci când dispuneți de o tastatură fizică, unele tipuri de text nu se potrivesc cu tastele apăsate, cum ar fi cazul programelor IME (Input Method Editor) utilizate pentru scripturi care nu au o tastatură și pentru care trebuie apăsate mai multe taste pentru generarea caracterelor.

Pentru a identifica introducerea unui text, elementele în care puteți introduce textul, cum ar fi tagurile `<input>` și `<textarea>`, generează evenimente `"input"` de fiecare dată când utiolizatorul le modifică textul conținut. Pentru a obține valoarea introdusă, este cel mai bine să citiți direct din câmpul care are focusul. [Capitolul ?](http#forms) vă va arăta cum să faceți asta.

## Evenimente de pointer

Sunt două metode de a indica lucruri pe ecran, frecvent utilizate: mouși (inclusiv dispozitive care acționează similar moușilor, cum ar fi track-pad-uri și track-ball-uri) și ecrane tactile. Acestea produc tipuri diferite de evenimente.

### Clickuri de mouse

{{index "mousedown event", "mouseup event", "mouse cursor"}}

Apăsarea butonului unui mouse declanșează mai multe evenimente. Evenimentele `"mousedown"` și `"mouseup"` sunt similare cu `"keydown"` și `"keyup"` și se declanșează când butonul este apăsat și eliberat. Aceasta au loc asupra nodurilor DOM aflate imediat sub indicatorul mouseului atunci când se declanșează evenimentul.

{{index "click event"}}

După evenimentul `"mouseup"` se declanșează un eveniment `"click"` asupra celui mai specific nod care conține atât apăsarea cât și eliberarea butonului. De exemplu, dacă apăs butonul mouse-ului pe un paragraf și apoi mut pointerul peste un alt paragraf, după care eliberez butonul, evenimentul `"click"` se va declanșa asupra elementului care conține ambele paragrafe.

{{index "dblclick event", "double click"}}

Dacă două click-uri au loc la scurt timp, unul după altul, se va declanșa un eveniment `"dblclick"` (double-click) după al doilea eveniment de click.

{{index pixel, "clientX property", "clientY property", "pageX property", "pageY property", "event object"}}

Pentru a obține informații precise despre locul în care a fost declanșat evenimentul de mouse, putem consulta proprietățile `clientX` și `clientY`, care conțion coordonatele evenimentului (în pixeli) relativ la colțul stânga-sus al ferestrei, sau `pageX` și `pageY`, care sunt relative la colțul stânga-sus al întregului document (valori care ar putea diferite, dacă s-a derulat pagina).

{{index "border-radius (CSS)", "absolute positioning", "drawing program example"}}

{{id mouse_drawing}}

În exemplul de mai jos construim un program primitiv de desenare. De fiecare dată când executați click pe document, este adăugat un punct sub pointerul mouseului. Pentru un program mai puțin primitiv, consultați [capitolul ?](paint).

```{lang: "text/html"}
<style>
  body {
    height: 200px;
    background: beige;
  }
  .dot {
    height: 8px; width: 8px;
    border-radius: 4px; /* rounds corners */
    background: blue;
    position: absolute;
  }
</style>
<script>
  window.addEventListener("click", event => {
    let dot = document.createElement("div");
    dot.className = "dot";
    dot.style.left = (event.pageX - 4) + "px";
    dot.style.top = (event.pageY - 4) + "px";
    document.body.appendChild(dot);
  });
</script>
```

### Mișcarea mouse-ului

{{index "mousemove event"}}

De fiecare dată când pointerul mouse-ului se mișcă, este declanșat un eveniment `"mousemove"`. Acest eveniment poate fi folosit pentru a urmări poziția mouse-ului. Acest lucru poate fi util pentru situațiile în care implementăm funcționalitate pentru tragerea obiectelor cu mouse-ul.

{{index "draggable bar example"}}

Ca și exemplu, programul de mai jos afișează o bară și setează handlere pentru evenimente astfel încât tragerea de laturile stânga sau dreapta măresc sau micșorează bara:

```{lang: "text/html", startCode: true}
<p>Drag the bar to change its width:</p>
<div style="background: orange; width: 60px; height: 20px">
</div>
<script>
  let lastX; // Tracks the last observed mouse X position
  let bar = document.querySelector("div");
  bar.addEventListener("mousedown", event => {
    if (event.button == 0) {
      lastX = event.clientX;
      window.addEventListener("mousemove", moved);
      event.preventDefault(); // Prevent selection
    }
  });

  function moved(event) {
    if (event.buttons == 0) {
      window.removeEventListener("mousemove", moved);
    } else {
      let dist = event.clientX - lastX;
      let newWidth = Math.max(10, bar.offsetWidth + dist);
      bar.style.width = newWidth + "px";
      lastX = event.clientX;
    }
  }
</script>
```

{{if book

Pagina rezultată arată astfel:

{{figure {url: "img/drag-bar.png", alt: "A draggable bar",width: "5.3cm"}}}

if}}

{{index "mouseup event", "mousemove event"}}

Observați că handlerul `"mousemove"` este înregistrat la nivelul întregii ferestre. Chiar dacă pointerul mouseului părăsește bara în timpul redimensionării, cât timp este menținut apăsat butonul, vrem să actualizăm dimensiunea barei.

{{index "buttons property", "button property", "bitfield"}}

Trebuie să oprim redimensionarea barei la eliberarea butonului mouse-ului. Pentru aceasta, putem utiliza proprietatea `buttons` (atenție, la plural), care ne spune ce butoane sunt apăsate la un moment dat. Dacă aceasta are valoarea 0, atunci nu este apăsat nici un buton. Când butoanele sunt apăsate, valoarea acestei proprietăți este suma codurilor pentru fiecare dintre butoane - butonul stânga are codul 1, cel din dreapta codul 2 iar cel din mijloc codul 4. Dacă sunt menținute apăsate butoanele stânga și dreapta simultan, valoarea pentru proprietatea `buttons` va fi 3.

Remarcați că ordinea acestor coduri este diferită de cea utilizată pentru `button`, unde butonul din mijloc avea cod mai mic decât cel din dreapta. Așa cum am mai menționat, consistența nu este chiar un punct forte al interfeței de programare a browserului.

### Evenimente pentru touch

{{index touch, "mousedown event", "mouseup event", "click event"}}

Tipul de browser grafic pe care îl utilizăm a fost conceput cu interfețele de tip mouse în minte, într-o perioadă când ecranele tactile erau rare. Pentru a face ca Web-ul să funcționeze pentru primele telefoane cu ecrane tactile, browserele de pe acele dispozitive pretindeau, până la un punct, că evenimentele de touch erau evenimente de mouse. Dacă atingeați ecranul, se declanșau evenimente `"mousedown"`, `"mouseup"` și `"click"`.

Dar această iluzie nu este foarte robustă. Un ecran tactil funcționează diferit de un mouse: nu are mai multe butoane, nu puteți urmări degetul dacă acesta nu este pe ecran (pentru a simula `mousemove`) și permite ca mai multe degete să fie pe ecran în același timp.

Evenimentele de mouse acoperă interacțiunea tactilă doar pentru situațiile evidente - dacă adăugați un handler pentru `"click"` la un buton, utilizatorii ecranelor tactile îl vor putea folosi. Dar o funcționalitate similară cu redimensionarea barei din exemplul anterior nu va funcționa.

{{index "touchstart event", "touchmove event", "touchend event"}}

Există evenimente specifice declanșate de interacțiunea prin intermediul ecranelor tactile. Când un deget începe să atingă ecranul, se va declanșa un eveniment `"touchstart"`. Când mișcăm degetul în timp ce atingem ecranul, se declanșează evenimentul `"touchmove"`. În sfârșit, când ridicăm degetul de pe ecran, va apărea evenimentul `"touchend"`.

{{index "touches property", "clientX property", "clientY property", "pageX property", "pageY property"}}

Deoarece multe ecrane tactile pot detecta mai multe atingeri simultane, aceste evenimente nu au un singur set de coordonate asociate cu ele. În loc de asta, obiectul evenimentului pentru ele are o proprietate `touches`, care reține un obiect similar unui array, cu punctele de atingere și fiecare punct are propriile proprietăți `clientX`, `clientY`, `pageX` și `pageY`.

Ați putea proceda ca și în exemplul de mai jos, pentru a afișa cercuri roșii în jurul fiecărui punct de atingere:

```{lang: "text/html"}
<style>
  dot { position: absolute; display: block;
        border: 2px solid red; border-radius: 50px;
        height: 100px; width: 100px; }
</style>
<p>Touch this page</p>
<script>
  function update(event) {
    for (let dot; dot = document.querySelector("dot");) {
      dot.remove();
    }
    for (let i = 0; i < event.touches.length; i++) {
      let {pageX, pageY} = event.touches[i];
      let dot = document.createElement("dot");
      dot.style.left = (pageX - 50) + "px";
      dot.style.top = (pageY - 50) + "px";
      document.body.appendChild(dot);
    }
  }
  window.addEventListener("touchstart", update);
  window.addEventListener("touchmove", update);
  window.addEventListener("touchend", update);
</script>
```

{{index "preventDefault method"}}

Foarte frecvent veți avea nevoie să apelați `preventDefault` în evenimentele tactile, pentru a suprascrie comportamentul implicit al browserului și pentru a preveni declanșarea evenimentelor de mouse, care ar putea să aibă și ele asociate handlere.

## Evenimente de scroll (derularea paginilor sau a elementelor)

{{index scrolling, "scroll event", "event handling"}}

Atunci când se derulează un element, se va declanșa un eveniment `"scroll"` pentru acesta. Acesta are diverse utilizări, cum ar fi să identificăm ce privește utilizatorul (pentru a dezactiva animații pe ecran, de exemplu) sau pentru a afișa un indicator de progres.

Exemplul de mai jos desenează o bară de progres în partea de sus a unui document și o actualizează pe măsură ce derulați în jos pagina:

```{lang: "text/html"}
<style>
  #progress {
    border-bottom: 2px solid blue;
    width: 0;
    position: fixed;
    top: 0; left: 0;
  }
</style>
<div id="progress"></div>
<script>
  // Create some content
  document.body.appendChild(document.createTextNode(
    "supercalifragilisticexpialidocious ".repeat(1000)));

  let bar = document.querySelector("#progress");
  window.addEventListener("scroll", () => {
    let max = document.body.scrollHeight - innerHeight;
    bar.style.width = `${(pageYOffset / max) * 100}%`;
  });
</script>
```

{{index "unit (CSS)", scrolling, "position (CSS)", "fixed positioning", "absolute positioning", percentage, "repeat method"}}

Atunci când un element are `position` setat la `fixed`, efectul este similar cu setarea poziției `absolute` dar elementul va rămâne vizibil în poziția sa chiar dacă se derulează pagina. În acest fel, bara de progres rămâne tot timpul vizibilă. Pentru a indica progresul, modificăm proprietatea `width`. Utilizăm `%` în loc de pixeli pentru a îi seta dimensiunea relativ la lățimea paginii.

{{index "innerHeight property", "innerWidth property", "pageYOffset property"}}

Bindingul global `innerHeight` ne returnează înălțimea ferestrei, pe care trebuie să o scădem din înălțimea totală de derulare - nu mai putem derula când ajungem la sfârșitul documentului. Există și o proprietate `innerWidth` pentru lățimea ferestrei. Împărțind `pageYOffset`, poziția curentă în derulare, cu poziția maximă de derulare și înmulțind cu 100, obținem procentul necesar pentru lățimea barei de progress.

{{index "preventDefault method"}}

Apelând `preventDefault` intr-un handler pentru evenimentul de derulare, nu va preveni derularea. De fapt, handlerul de eveniment va fi apelat doar _după_ ce a avut loc derularea.

## Evenimente pentru focus

{{index "event handling", "focus event", "blur event"}}

Atunci când un element are primește focusul, browserul declanșează un eveniment `"focus"` asupra sa. Când elementul pierde focusul, acesta primește un eveniment `"blur"`.

{{index "event propagation"}}

Spre deosebire de evenimentele discutate anterior, aceste două evenimente nu se propagă. Un handler de pe elementul părinte nu va fi notificat atunci când un element descendent câștigă sau pierde focusul.

{{index "input (HTML tag)", "help text example"}}

Exemplul de mai jos afișează un text explicativ pentru câmpul de text care are focusul:

```{lang: "text/html"}
<p>Name: <input type="text" data-help="Your full name"></p>
<p>Age: <input type="text" data-help="Your age in years"></p>
<p id="help"></p>

<script>
  let help = document.querySelector("#help");
  let fields = document.querySelectorAll("input");
  for (let field of Array.from(fields)) {
    field.addEventListener("focus", event => {
      let text = event.target.getAttribute("data-help");
      help.textContent = text;
    });
    field.addEventListener("blur", event => {
      help.textContent = "";
    });
  }
</script>
```

{{if book

Imaginea de mai jos ilustrează textul explicativ pentru câmpul de vârstă.

{{figure {url: "img/help-field.png", alt: "Providing help when a field is focused",width: "4.4cm"}}}

if}}

{{index "focus event", "blur event"}}

Obiectul `window` va recepționa evenimentele `"focus"` și `"blur"` atunci când userul schimbă tab-ul sau fereastra în care documentul este afișat.
## Evenimentul "load"

{{index "script (HTML tag)", "load event"}}


Când se termină încărcarea unei pagini, se va declanșa evenimentul `"load"` pentru fereastră și obiectele din corpul documentului. Acesta este adesea utilizat pentru a programa acțiuni care necesită ca întregul document să fie construit. Amintiți-vă că, imediat ce se întâlnește un tag `<script>`, conținutul acestuia este executat. Ar putea fi prea devreme, de exemplu atunci când scriptul trebuie să acționeze asupra unor elemente din document care apar după tag-ul `<script>`.

{{index "event propagation", "img (HTML tag)"}}

Elementele cum ar fi tagurile pentru imagini și script-uri care încarcă un fișier extern, au asociat un eveniment `"load"` care indică finalizarea încărcării fișierelor respective. Ca și evenimentele legate de focus, aceste evenimente nu se propagă.

{{index "beforeunload event", "page reload", "preventDefault method"}}

Când o pagină este închisă sau părăsită ( prin urmarea unei hiperlegături, de exemplu), se declanșează un eveniment `"beforeunload"`. Principala utilizare a acestui eveniment este de a preveni pierderea accidentală de date atunci când utilizatorul a făcut modificări într-un document pe care vrea să îl închidă. Dacă preveniți comportamentul implicit al acestui eveniment _și_ setați proprietatea `returnValue` a obiectului evenimentului la un string, browserul va afișa un dialog în care va întreba utilizatorul dacă intr-adevăr dorește să părăsească pagina. Acest dialog ar putea conține stringul vostru dar, deoarece unele site-uri malițioase încearcă să utilizeze aceste dialoguri pentru a introduce confuzie utilizatorului, majoritatea browserelor nu le mai afișează.

{{id timeline}}

## Evenimente și bucla de evenimente

{{index "requestAnimationFrame function", "event handling", timeline, "script (HTML tag)"}}

În contextul buclei de evenimente, așa cum am discutat în [capitolul ?](async), handlerele petnru evenimente ale browserului se comportă ca și celelalte notificări asincrone. Ele sunt programate când se produc evenimentele dar trebui să aștepte pentru alte scripturi să își termine execuția înainte de a avea șansa să se execute.

Faptul că evenimentele pot fi procesate doar când nimic altceva nu se execută înseamnă că, dacă bucla de evenimente este legată cu alte lucruri, orice interacțiune cu pagina (care se întâmplă prin evenimente) va fi întârziată până în momentul în care poate fi executată. Astfel, dacă veți programa prea multă muncă, fie prin handlere care au durată mare de execuție, fie prin multe handlere scurte, pagina va deveni lentă și greu de utilizat.

Pentru cazurile în care vreți să executați un cod consumator de timp în fundal, fără să înghețați pagina, browserele vă pun la dispoziție _web worker-i_. Un "worker" este un proces JavaScript care rulează în paralel cu scriptul principal, pe propria sa linie temporală.

Imaginați-vă că ridicarea la pătrat a unui număr este un calcul de durată pe care vrem să îl executăm pe un thread separat. Am putea scrie codul într-un fișier numit `code/squareworker.js` care răspunde la mesaje prin calcularea pătratului și returnarea unui mesaj.

```
addEventListener("message", event => {
  postMessage(event.data * event.data);
});
```

Pentru a evita problema de a avea mai multe thread-uri ce modifică aceleași date, worker-ii nu își partajează domeniul de vizibilitate global sau orice alte date cu mediul scriptului principal. Comunicarea cu ei se realizează prin mesaje.

Codul de mai jos crează un worker ce va rula acel script, îi trimite mai multe mesaje și afișează răspunsurile.

```{test: no}
let squareWorker = new Worker("code/squareworker.js");
squareWorker.addEventListener("message", event => {
  console.log("The worker responded:", event.data);
});
squareWorker.postMessage(10);
squareWorker.postMessage(24);
```

{{index "postMessage method", "message event"}}

Funcția `postMessage` trimite un mesaj, ceea ce va declanșa un eveniment `"message"` la destinatar. Scriptul care a creat worker-ul trimite și primește mesaje prin intermediul obiectului `Worker`, în timp ce workerul discută cu scriptul care l-a creat prin observarea directă a propriului domeniu de vizibilitate global. Doar valorile care pot fi reprezentate în JSON pot fi trimise ca și mesaje - cealaltă parte va primi o _copie_, nu valoarea însăși.

## Timere

{{index timeout, "setTimeout function"}}

Am lucrat cu funcția `setTimeout` în [capitolul ?](async). Ea programează apelul ulterior al unei funcții, după un număr dat de milisecunde.

{{index "clearTimeout function"}}

Uneori trebuie să renunțați la execuția unei funcții anterior programate. Puteți realiza aceasta cu ajutorul valorii returnate de către `setTimeout` și să apelați `clearTimeout` asupra ei.

```
let bombTimer = setTimeout(() => {
  console.log("BOOM!");
}, 500);

if (Math.random() < 0.5) { // 50% chance
  console.log("Defused.");
  clearTimeout(bombTimer);
}
```

{{index "cancelAnimationFrame function", "requestAnimationFrame function"}}

Funcția `cancelAnimationFrame` se execută în același mod ca și `clearTimeout` - apelarea ei asupra rezultatului returnat de  `requestAnimationFrame` va renunța la acel cadru (presupunând că nu a fost deja apelat).

{{index "setInterval function", "clearInterval function", repetition}}

Uns et similar de funcții, `setInterval` și `clearInterval`, sunt utilizate pentru a seta timere care se vor _repeta_ după fiecare _X_ milisecunde.

```
let ticks = 0;
let clock = setInterval(() => {
  console.log("tick", ticks++);
  if (ticks == 10) {
    clearInterval(clock);
    console.log("stop.");
  }
}, 200);
```

## Dezactivarea (debouncing)

{{index optimization, "mousemove event", "scroll event", blocking}}

Unele tipuri de evenimente au potențialul de a se declanșa rapid, de mai mutle ori (cum ar fi `"mousemove"` și `"scroll"`). Când gestionăm asemenea evenimente, trebuie să avem grijă să nu executăm cod consumator de timp. În caz contrar, handlerul va consuma atât de mult timp încât interacțiunea cu documentul va fi încetinită.

{{index "setTimeout function"}}

Dacă trebuie să executați o operație non-trivială într-un asemenea handler, puteți utiliza `setTimeout` pentru a vă asigura că nu o executați prea frecvent. Aceasta înseamnă _deazctivarea (debouncing)_ evenimentului. Există câteva abordări ușor diferite pentru a realiza această sarcină.

{{index "textarea (HTML tag)", "clearTimeout function", "keydown event"}}

În primul exemplu, vrem să reacționăm la inputul utilizatorului, dar nu vrem să reacționăm imediat la fiecare eveniment de intrare. Când utilizatorul tastează rapid, vrem să așteptăm până când apare o pauză. În loc să executăm imediat o acțiune în handler, setăm un timeout. De asemenea, renunțăm la timeout-ul anterior (dacă există) astfel încât, dacă două evenimente apar aproape unul de altul (mai aproape decât întârzierea setată în timeout), timeout-ul din evenimentul anterior va fi eliminat.

```{lang: "text/html"}
<textarea>Type something here...</textarea>
<script>
  let textarea = document.querySelector("textarea");
  let timeout;
  textarea.addEventListener("input", () => {
    clearTimeout(timeout);
    timeout = setTimeout(() => console.log("Typed!"), 500);
  });
</script>
```

{{index "sloppy programming"}}

Transmiterea unei valori nedefinite către `clearTimeout` sau apelul acestei funcții asupra unui timeout care deja a fost declanșat, nu are nici un efect. Astfel, nu trebuie să ne preocupe apelul și doar îl executăm pentru fiecare eveniment.

{{index "mousemove event"}}

Putem folosi un șablon diferit dacă vrem să spațiem răspunsurile astfel încât ele să fie separate de cel puțin un anumit interval de timp dar vrem să le declanșăm _pe parcursul_ unei serii de evenimente, nu doar la sfârșitul ei. De exemplu, vrem să răspundem la evenimentele  `"mousemove"` prin afișarea coordonatelor curente ale mouseului, dar doar la fiecare 250 milisecunde.

```{lang: "text/html"}
<script>
  let scheduled = null;
  window.addEventListener("mousemove", event => {
    if (!scheduled) {
      setTimeout(() => {
        document.body.textContent =
          `Mouse at ${scheduled.pageX}, ${scheduled.pageY}`;
        scheduled = null;
      }, 250);
    }
    scheduled = event;
  });
</script>
```

## Rezumat

Handlerele pentru evenimente ne permit detectarea și reacția la evenimente ce au loc în pagina noastră web. Metoda `addEventListener` este utilizată pentru a înregistra un asemenea handler.

Fiecare eveniment are un tip (`"keydown"`, `"focus"` etc), care îl identifică. Majoritatea evenimentelor sunt declanșate asupra unui element specific din DOM și apoi se propagă la ancestorii acestuia, permițând handlere-lor asociate cu acele elemente să le trateze.

Când este apelat un handler de eveniment, i se transmite un obiect de eveniment cu informații suplimentare despre eveniment. Acest obiect are și metode care ne permit să stopăm propagarea (`stopPropagation`) sau să prevenim tratarea implicită a evenimentului de către browser (`preventDefault`).

Apăsarea unei taste declanșează evenimentele `"keydown"` și `"keyup"`. Apăsarea unui buton al mouse-ului declanșează evenimentele  `"mousedown"`, `"mouseup"` și `"click"`. Mișcarea mouseului declanșează evenimente `"mousemove"`. Interacțiunea cu un ecran tactil va declanșa evenimente `"touchstart"`, `"touchmove"` și `"touchend"`.

Derularea poate fi detectată prin intermdiul evenimentului `"scroll"` iar schimbările de focus pot fi detectate prin evenimentele  `"focus"` și `"blur"`. Când se finalizează încărcarea documentului, este declanșat un eveniment `"load"` în fereastră.

## Exerciții

### Balonul

{{index "balloon (exercise)", "arrow key"}}

Scrieți o pagină care afișează un balon (folosind emoji-ul 🎈). Când apăsați săgeata-sus trebuie să se umfle cu 10 procente iar când apăsați săgeata în jos să se dezumfle cu 10 procente.

{{index "font-size (CSS)"}}

Puteți controla dimensiunea textului (emoji sunt text) prin setarea proprietății CSS `font-size` (`style.fontSize` în JavaScript) asupra elementului părinte. Includeți si o unitate pentru dimensiune, de exemplu pixeli (`10px`). 

Numele tastelor pentru cele două taste menționate sunt `"ArrowUp"` și `"ArrowDown"`. Cheile trebuie să schimbe doar dimensiunea balonului, fără a derula pagina.

După ce reușiți să implementați cele de mai sus, adăugați o funcționalitate în care, dacă umflați balonul peste o anumită dimensiune, acesta va exploda, adică v-a fi înlocuit de un emoji 💥, și handler-ul pentru eveniment va fi eliminat (nu veți putea umfla sau dezumfla explozia).

{{if interactive

```{test: no, lang: "text/html", focus: yes}
<p>🎈</p>

<script>
  // Your code here
</script>
```

if}}

{{hint

{{index "keydown event", "key property", "balloon (exercise)"}}

Va trebui să înregistrați un handler pentru evenimentul `"keydown"` și să consultați `event.key` pentru a determina dacă au fost apăsate tastele săgeata-sus sau săgeata-jos.

Dimensiunea curentă poate fi stocată într-un binding, astfel încât se va putea determina noua dimensiune. Ar fi util să definiți o funcție care actualizează dimensiunea - atât bindingul cât și stilul balonului în DOM - pe care o veți putea apela din handlerul pentru eveniment și poate încă o dată la început, pentru a seta dimensiunea inițială.

{{index "replaceChild method", "textContent property"}}

Puteți schimba balonul într-o explozie prin înlocuirea nodului text cu un alt nod (utilizând `replaceChild`) sau setând proprietatea `textContent` a nodului său părinte la o nouă valoare de tip string.

hint}}

### Urma mouse-ului

{{index animation, "mouse trail (exercise)"}}

La începuturile JavaScript, o perioadă cu multe imagini animate, programatorii au descoperit multe moduri creative de a utiliza limbajul.

Una dintre acestea era _urma mouseului_ - o serie de elemente care ar urma pointerul mouseului în mișcarea acestuia în pagină.

{{index "absolute positioning", "background (CSS)"}}

În acest exercițiu, aș vrea să implementați ceva similar. Utilizați elemente `<div>` poziționate absolut, cu dimensiune fixă și o anumită culoare de fundal (consultați [codul](event#mouse_drawing) din secțiunea "Mouse Clicks" pentru un exemplu). Creați mai multe astfel de elemente, iar când mouse-ul se mișcă, afișați-le în poziția pointerului.

{{index "mousemove event"}}

Puteți folosi mai multe abordări. Puteți construi soluția cât de complexă doriți. O soluție simplă, pentru a începe, este să rețineți un număr fix de elemente de urmărire și să ciclați printre ele, mutându-l pe următorul în poziția curentă a mouse-ului de fiecare dată când este declanșat un eveniment `"mousemove"`.

{{if interactive

```{lang: "text/html", test: no}
<style>
  .trail { /* className for the trail elements */
    position: absolute;
    height: 6px; width: 6px;
    border-radius: 3px;
    background: teal;
  }
  body {
    height: 300px;
  }
</style>

<script>
  // Your code here.
</script>
```

if}}

{{hint

{{index "mouse trail (exercise)"}}

Elementele se pot crea într-o buclă. Apoi le adăugați la document pentru ca acestea să apară. Pentru a le putea accesa ulterior și a le modifica pooziția, probabil că ați vrea să le memorați într-un array.

{{index "mousemove event", [array, indexing], "remainder operator", "% operator"}}

Ciclarea printre ele se poate realiza cu ajutorul unei variabile cu rol de contor și incrementarea acesteia la fiecare declanșare a unui eveniment `"mousemove"`. Operatorul pentru calculul restului (`% elements.length`) poate fi utilizat apoi pentru a obține un index valid pe array pentru a selecta elementul pe care să îl deplasați la un eveniment dat.

{{index simulation, "requestAnimationFrame function"}}

Un alt efect interesant poate fi obținut dacă modelați un sistem simplu. Utilizați evenimentul `"mousemove"` doar petnru a actualiza o pereche de bindinguri care urmăresc poziția mouseului. Apoi utilizați `requestAnimationFrame` pentru a simula că elementele sunt atrase spre poziția pointerului mouseului. În fiecare pas al animației, actualizați-le poziția pe baza locației relativ la pointerul mouseului (eventual având și o viteză memorată pentru fiecare element). Cum să realizați acest lucru este complet la latiudinea voastră.

hint}}

### Tab-uri

{{index "tabbed interface (exercise)"}}

Panourile tabulare sunt utilizate pe larg în interfețele utilizator. Ele permit selectarea unui panou prin alegerea dintre un set tab-uri ce sunt plasate deasupra unui element.

{{index "button (HTML tag)", "display (CSS)", "hidden element", "data attribute"}}

În acest exercițiu trebuie să impleemntați o interfață tabulară simplă. Scrieți o funcție `asTabs`, care primește un nod DOM și crează o interfață tabulară ce afișează copiii acelui nod. Trebuie să insereze o listă de elemente `<button>` la începutul nodului, câte unul pentru fiecare element copil, conținând textul din atributul `data-tabname` al copilului. Toate elementele copil originale, cu excepția primului trebuie să fie ascunse (stilul pentru `display` să fie `none`). Elementul vizibil trebuie să poată fi selectat prin click-uri pe butoane.

Când reușiți, continuați cu stilizarea diferită a butonului pentru tabul selectat astfel încât să fie clar ce tab este selectat.

{{if interactive

```{lang: "text/html", test: no}
<tab-panel>
  <div data-tabname="one">Tab one</div>
  <div data-tabname="two">Tab two</div>
  <div data-tabname="three">Tab three</div>
</tab-panel>
<script>
  function asTabs(node) {
    // Your code here.
  }
  asTabs(document.querySelector("tab-panel"));
</script>
```

if}}

{{hint

{{index "text node", "childNodes property", "live data structure", "tabbed interface (exercise)", [whitespace, "in HTML"]}}

Nu veți putea utiliza direct proprietatea `childNodes` a nodului ca și colecție pentru nodurile tabulare. Atunci când veți adăuga butoanele, acestea vor fi tot noduri copil și vor fi adăugate la acest obiect deoarece este o structură de date "live". Pe de altă parte, nodurile text create pentru spațiile albe dintre noduri fac și ele parte din `childNodes` dar nu trebuie să li se creeze un tab. Puteți utiliza `children` în loc de `childNodes` pentru a ignora nodurile text.

{{index "TEXT_NODE code", "nodeType property"}}

Puteți începe prin a construi un array de taburi, astfel încât să aveți ușor acces la ele. Pentru a implementa stilizarea butoanelor, puteți stoca obiecte care conțin atât panoul tabului cât și butonul.

Vă recomand să scrieți o funcție separată pentru comutarea între taburi. Puteți fie să stocați tabul selectat anterior și să modificați doar stilurile necesare pentru a-l ascunde și a afișa unul nou, sau să actualizați stilurile tuturor taburilor de fiecare dată când este selectat unul nou.

Probabil că ar trebui să apelați această funcție imediat pentru ca interfața să se afișeze cu primul tab vizibil.

hint}}
