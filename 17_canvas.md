{{meta {load_files: ["code/chapter/16_game.js", "code/levels.js", "code/chapter/17_canvas.js"], zip: "html include=[\"img/player.png\", \"img/sprites.png\"]"}}}

# Desenarea pe Canvas

{{quote {author: "M.C. Escher", title: "cited by Bruno Ernst in The Magic Mirror of M.C. Escher", chapter: true}

Desenarea este decepție.

quote}}

{{index "Escher, M.C."}}

{{figure {url: "img/chapter_picture_17.jpg", alt: "Picture of a robot arm drawing on paper", chapter: "framed"}}}

{{index CSS, "transform (CSS)", [DOM, graphics]}}

Browserele ne permit mai multe moduri de a afișa imagini. Cel mai simplu mod este de a utiliza reguli de stil pentru a pozționa și colora elemente din DOM. Puteți ajunge destul de departe, așa cum am văzut în [capitolul anterior](game). Adăugând imagini parțial transparente la noduri, le putem afișa exact cum ne dorim. Putem chiar să rotim sau să deformăm cu proprietatea de stil `transform`.

Dar am utiliza DOM pentru un scop pentru care nu a fost conceput. Unele sarcini, cum ar fi desenarea unei linii între două puncte oarecare, sunt extrem de ciudat de realizat cu elemente HTML obișnuite.

{{index SVG, "img (HTML tag)"}}

Avem două alternative. Prima se bazează pe DOM dar utilizează _Scalable Vector Graphics (SVG)_. SVG este un dialect de marcare a documentelor care se concentrează pe figuri geometrice, nu pe text. Puteți încorpora un document SVG direct în orice document HTML sau să îl includeți cu tagul `<img>`.

{{index clearing, [DOM graphics], [interface, canvas]}}

A două alternativă este numită _canvas_. Un canvas este un singur element DOM care încpsulează o pictură. El expune o interfață de programare pentru desenarea figurilor geometrice în spațiul ocupat de către nod. Principala diferență dintre o imagine pe canvas și una in SVG este că descrierea originală în SVG a figurilor geometrice este păstrată astfel încât ele pot fi mutate sau redimensionate fără pierderi de calitate. Un canvas, pe de altă parte, convertește formele la pixeli (puncte colorate pe o grilă) imediat ce au fost desenate și nu își amintește ce reprezintă acești pixeli. Singura modalitate de a muta o figură pe canvas este de a curăța canvasul (sau partea din canvas aflată în jurul figurii) și apoi să redesenăm cu figura în noua poziție.

## SVG

Această carte nu va intra în detalii despre SVG, dar voi explica pe scurt cum funcționează. La sfârșitul [acestui capitol](canvas#graphics_tradeoffs), voi reveni asupra compromisurilor pe care trebuie să le aveți în vedere când decideți care mecanism de desenare este adecvat pentru o anume aplicație.

Acesta este un document HTML cu o imagine SVG simplă în el:

```{lang: "text/html", sandbox: "svg"}
<p>Normal HTML here.</p>
<svg xmlns="http://www.w3.org/2000/svg">
  <circle r="50" cx="50" cy="50" fill="red"/>
  <rect x="120" y="5" width="90" height="90"
        stroke="blue" fill="none"/>
</svg>
```

{{index "circle (SVG tag)", "rect (SVG tag)", "XML namespace", XML, "xmlns attribute"}}

Atributul `xmlns` schimbă un element (și descendenții săi) la un _XML namespace_ diferit. Acest spațiu de nume, identificat printr-un URL, specifică dialectul pe care îl vorbim. Tagurile `<circle>` și `<rect>`, care nu există în HTML, au o semnificație în SVG - ele desenează figuri geometrice utilizând stilul și poziția specificate de atributele lor.

{{if book

Documentul este afișat astfel:

{{figure {url: "img/svg-demo.png", alt: "An embedded SVG image",width: "4.5cm"}}}

if}}

{{index [DOM, graphics]}}

Aceste taguri crează elemente DOM, ca și tagurile HTML, iar scripturile pot interacționa cu ele. De exemplu, codul de mai jos schimbă culoarea de umplere a elementului `<circle>`:

```{sandbox: "svg"}
let circle = document.querySelector("circle");
circle.setAttribute("fill", "cyan");
```

## Elementul canvas

{{index [canvas, size], "canvas (HTML tag)"}}

Imaginile canvas pot fi desenate pe un element `<canvas>`. Pentru un asemenea element, putem seta atributele `width` și `height` pentru a îi specifica dimensiunea în pixeli.

Un canvas nou este gol, adică este complet transparent și astfel, va fi afișat ca un spațiu gol în document.

{{index "2d (canvas context)", "webgl (canvas context)", OpenGL, [canvas, context], dimensions, [interface, canvas]}}

Scopul tag-ului `<canvas>` este de a permite diferite tipuri de desenare. Pentru a avea acces la o interfață de desenare, trebuie mai înâi să creem un _context_, un obiect ale cărui metode oferă interfața de desenare. În prezent sunt suportate pe larg două stiluri de desenare: `"2d"` pentru grafică bidimensională, și `"webgl"` pentru grafică tridimensională prin interfața OpenGL.

{{index rendering, graphics, efficiency}}

În această carte nu vom discuta despre WebGL - vom rămâne în două dimensiuni. Dar dacă sunteți interesați de grafica tridimensională, vă încurajez să studiați despre WebGL. Acesta furnizează o interfață directă către hardware-ul pentru grafică și vă permite să randați chiar și scene mai complicate în mod eficient, utilizând JavaScript.

{{index "getContext method", [canvas, context]}}

Creați un context prin apelarea metodei `getContext` a elementului DOM `<canvas>`.

```{lang: "text/html"}
<p>Before canvas.</p>
<canvas width="120" height="60"></canvas>
<p>After canvas.</p>
<script>
  let canvas = document.querySelector("canvas");
  let context = canvas.getContext("2d");
  context.fillStyle = "red";
  context.fillRect(10, 10, 100, 50);
</script>
```

După ce crează obiectul context, exemplul desenează un dreptunghi roșu cu lățimea de 100 pixeli și înălțimea de 50 pixeli, al cărui colț stânga-sus este în poziția (10,10).

{{if book

{{figure {url: "img/canvas_fill.png", alt: "A canvas with a rectangle",width: "2.5cm"}}}

if}}

{{index SVG, coordinates}}

Ca și în HTML (și SVG), sistemul de coordonate al canvasului plasează (0,0) în colțul stânga-sus iar axa y este orientată în jos. Prin urmare, (10,10) înseamnă 10 pixeli la stânga și 10 pixeli în jos, față de colțul stânga-sus.

{{id fill_stroke}}

## Linii și suprafețe

{{index filling, stroking, drawing, SVG}}

In interfața pentru canvas, o figură poate fi _umplută_, adică suprafața sa să aibă o anumită culoare sau un șablon, sau poate fi _schițată_, ceea ce înseamnă desenarea conturului pe laturile sale. Aceeași terminologie este utilizată în SVG.

{{index "fillRect method", "strokeRect method"}}

Metoda `fillRect` umple un dreptunghi. Primește ca argumente coordonatele x și y ale colțului stânga-sus, apoi lățimea și înălțimea. O metodă similară, `strokeRect` desenează conturul unui dreptunghi.

{{index [state, "of canvas"]}}

Nici una dintre metode nu are parametri suplimentari. Culoarea de umplere, grosimea liniilor și așa mai departe nu sunt determinate de către argumente transmise acestor funcții (așa cum ar fi rezonabil să vă așteptați) ci de către proprietăți ale obiectului context.

{{index filling, "fillStyle property"}}

Proprietatea `fillStyle` controlează modul de umplere a figurilor. Poate fi setată la un string ce definește o culoare, utiliând denumirile utilizate în CSS.

{{index stroking, "line width", "strokeStyle property", "lineWidth property", canvas}}

Proprietatea `strokeStyle` funcționează similar dar determină culoarea utilizată pentru contururi. Lățimea acelei linii este determinată de către proprietatea `lineWidth` property, care poate fi orice număr pozitiv.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.strokeStyle = "blue";
  cx.strokeRect(5, 5, 50, 50);
  cx.lineWidth = 5;
  cx.strokeRect(135, 5, 50, 50);
</script>
```

{{if book

Codul de mai sus desenează două pătrate albastre, utilizând un contur mai gros pentru cel de al doilea.

{{figure {url: "img/canvas_stroke.png", alt: "Two stroked squares",width: "5cm"}}}

if}}

{{index "default value", [canvas, size]}}

Când nu se specifică atributele `width` sau `height`, ca și în exemplu, elementul canvas primește o lățime implicită de 300 pixeli și o înălțime de 150 pixeli.

## Căi

{{index [path, canvas], [interface, design], [canvas, path]}}

O "cale" este o succesiune de linii. Interfața 2D a obiectului canvas are o abordare specifică pentru descrierea sa. Este implementată complet prin efecte secundare. Căile nu sunt valori care pot fi memorate și transmise. În loc de asta, dacă vreți să construiți ceva cu o asemenea linie poligonală, veți executa o secvență de apeluri către metoda pentru a descrie forma.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  for (let y = 10; y < 100; y += 10) {
    cx.moveTo(10, y);
    cx.lineTo(90, y);
  }
  cx.stroke();
</script>
```

{{index canvas, "stroke method", "lineTo method", "moveTo method", shape}}

Exemplul de mai sus crează o cale cu câteva segmente orizontale și apoi îl schițează folosind metoda `stroke`. Fiecare segment creat cu `lineTo` începe de la poziția curentă. Acea poziție este de regulă finalul ultimului segment, dacă nu a fost apelat `moveTo`. În acel caz, următorul segment începe din poziția transmisă către `moveTo`.

{{if book

Calea descrisă de către programul anterior arată cam așa:

{{figure {url: "img/canvas_path.png", alt: "Stroking a number of lines",width: "2.1cm"}}}

if}}

{{index [path, canvas], filling, [path, closing], "fill method"}}

Când umplem o cale (utilizând metoda `fill`), fiecare figură este umplută separat. O cale poate conține mai multe figuri - fiecare deplasare `moveTo` începe cu o nouă figură. Dar calea trebuie să fie _închisă_ (adică poziția de început și cea de sfârșit trebuie să fie identice) pentru a putea fi umplută. Dacă calea nu este închisă, se adaugă o linie de la sfârșitul la începutul său și figura formată de calea închisă este umplută.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(50, 10);
  cx.lineTo(10, 70);
  cx.lineTo(90, 70);
  cx.fill();
</script>
```

Exemplul de mai sus desenează un triunghi umplut. Observați că doar două dintre laturile triunghiului sunt desenate explicit. Cea de a treia este implicită și nu ar fi definită dacă ați utiliza `stroke`.

{{if book

{{figure {url: "img/canvas_triangle.png", alt: "Filling a path",width: "2.2cm"}}}

if}}

{{index "stroke method", "closePath method", [path, closing], canvas}}

Ați putea să apelați metoda `closePath` pentru a închide explicit o cale prin adăugarea unui segment înapoi spre începutul căii. Acest segment ar fi desenat când ați apela metoda `stroke`.

## Curbe

{{index [path, canvas], canvas, drawing}}

O cale poate să conțină și linii curbe. Acestea se desenează puțin mai complicat.

{{index "quadraticCurveTo method"}}

Metoda `quadraticCurveTo` desenează o curbă către un punct. Pentru a determina curbura liniei, metodei i se transmite un punct de control precum și un punct de destinație. Punctul de control _atrage_ linia, determinându-i curbura. Linia curbă nu va trece prin punctul de control, dar direcția sa în punctele de început și sfârșit va indica spre punctul de control. Exemplul de mai jos ilustrează aceasta:

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(10, 90);
  // control=(60,10) goal=(90,90)
  cx.quadraticCurveTo(60, 10, 90, 90);
  cx.lineTo(60, 10);
  cx.closePath();
  cx.stroke();
</script>
```

{{if book

Se va produce o cale care va arăta cam așa:

{{figure {url: "img/canvas_quadraticcurve.png", alt: "A quadratic curve",width: "2.3cm"}}}

if}}

{{index "stroke method"}}

Desenăm o curbă pătratică de la stânga la dreapta și apoi desenăm două segmente de dreaptă care trec prin punctul de control și capetele curbei. Rezultatul se aseamănă cu o insignă _Star Trek_. Puteți observa efectul punctului de control: linia curba pleacă spre punctul de control dar apoi se curbează pentru a se întoarce spre punctul de destinație.

{{index canvas, "bezierCurveTo method"}}

Metoda `bezierCurveTo` desenează o curbă similară. Dar în loc de un singur punct de control, aceasta are două puncte de control, câte unul pentru fiecare capăt al curbei. Exemplul de mai jos ilustrează comportamentul unei astfel de curbe:

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(10, 90);
  // control1=(10,10) control2=(90,10) goal=(50,90)
  cx.bezierCurveTo(10, 10, 90, 10, 50, 90);
  cx.lineTo(90, 10);
  cx.lineTo(10, 10);
  cx.closePath();
  cx.stroke();
</script>
```

Cele două puncte de control definesc direcția curbei la cele două capete. Cu cât sunt mai departe de punctul corespunzător, cu atât mai mult curba se va "bomba" în acea direcție.

{{if book

{{figure {url: "img/canvas_beziercurve.png", alt: "A bezier curve",width: "2.2cm"}}}

if}}

{{index "trial and error"}}

Este greu de lucrat cu asemenea curbe - nu putem ăntotdeauna determina punctele de control care să ne deseneze curba pe care o vrem. Uneori le puteți calcule, alteori le puteți găsi doar printr-un proces de "trial & error".

{{index "arc method", arc}}

Metoda `arc` vă permite să desenați o parte dintr-un cerc. Primește o pereche de coordonate pentru centrul cercului, o rază și unghiurile de început și sfârșit.

{{index pi, "Math.PI constant"}}

Ultimii doi parametri sunt cei care ne permit să desenăm parțial cercul. Unghiurile sunt măsurate în radiani, nu în grade. Un cerc complet are un unghi de 2π, sau `2 * Math.PI`, aproximativ 6.28. Unghiul de 0 grade corespunde unui punct aflat în dreapta centrului cercului și specificarea altor unghiuri se face în sensul acelor de ceasornic. Dacă utilizați unghiul de început 0 și o valoare mai mare decât 2π (sa zicem 7) veți putea desena un cerc complet.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  // center=(50,50) radius=40 angle=0 to 7
  cx.arc(50, 50, 40, 0, 7);
  // center=(150,50) radius=40 angle=0 to ½π
  cx.arc(150, 50, 40, 0, 0.5 * Math.PI);
  cx.stroke();
</script>
```

{{index "moveTo method", "arc method", [path, " canvas"]}}

Imaginea rezultată conține o curbă de la dreapta cercului complet (primul apel către `arc`), până în dreapta sfertului de cerc (al doilea apel). Ca și alte metode de desenare a căilor, o curbă desenată cu `arc` este conectată la segmentul anterior al căii. Puteți apela `moveTo` sau să începeți o nouă cale pentru a evita acest lucru.

{{if book

{{figure {url: "img/canvas_circle.png", alt: "Drawing a circle",width: "4.9cm"}}}

if}}

{{id pie_chart}}

## Desenarea unei hărți-plăcintă (pie-chart)

{{index "pie chart example"}}

Imaginați-vă ca tocmai ați primit un post la EconomiCorp, Inc., și prima voastră sarcină este să desenați un grafic cu rezultatele sondajului cu privire la satisfacția clienților.

Bindingul `results` conține un array de obiecte care reprezintă răspunsurile sondajului.

```{sandbox: "pie", includeCode: true}
const results = [
  {name: "Satisfied", count: 1043, color: "lightblue"},
  {name: "Neutral", count: 563, color: "lightgreen"},
  {name: "Unsatisfied", count: 510, color: "pink"},
  {name: "No comment", count: 175, color: "silver"}
];
```

{{index "pie chart example"}}

Pentru a desena un pie-chart vom desena mai multe felii, fiecare cu ajutorul unui arc și a unei perechi de linii către centrul acelui arc. Putem calcula unghiul  fiecărui arc împărțind cercul complet (2π) la numărul total de răspunsuri (unghiul pentru un singur răspuns) și apoi să multiplicăm această valoare cu numărul total de persoane care au făcut o anumită alegere.

```{lang: "text/html", sandbox: "pie"}
<canvas width="200" height="200"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let total = results
    .reduce((sum, {count}) => sum + count, 0);
  // Start at the top
  let currentAngle = -0.5 * Math.PI;
  for (let result of results) {
    let sliceAngle = (result.count / total) * 2 * Math.PI;
    cx.beginPath();
    // center=100,100, radius=100
    // from current angle, clockwise by slice's angle
    cx.arc(100, 100, 100,
           currentAngle, currentAngle + sliceAngle);
    currentAngle += sliceAngle;
    cx.lineTo(100, 100);
    cx.fillStyle = result.color;
    cx.fill();
  }
</script>
```

{{if book

Exemplul de mai sus desenează următoarea diagramă:

{{figure {url: "img/canvas_pie_chart.png", alt: "A pie chart",width: "5cm"}}}

if}}

Dar o diagramă care nu ne spune ce reprezintă feliile nu este foarte utilă. Avem nevoie de o modalitate de a plasa text pe canvas.

## Text

{{index stroking, filling, "fillStyle property", "fillText method", "strokeText method"}}

Contextul de desenare 2D al unui canvas ne pune la dispoziție metodele `fillText` și `strokeText`. Ceaa de a doua este utilă pentru a desena conturul literelor, dar de regulă prima este cea de care aveți nevoie. Ea va umple literele cu setarea curentă pentru `fillStyle`.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.font = "28px Georgia";
  cx.fillStyle = "fuchsia";
  cx.fillText("I can draw text, too!", 10, 50);
</script>
```

Puteți specifica mărimea, stilul și fontul textului în proprietatea `font`. Exemplul de mai sus setează doar mărimea și familia textului. Puteți adăuga `italic` sau `bold` la începutul stringului pentru a seta un stil.

{{index "fillText method", "strokeText method", "textAlign property", "textBaseline property"}}

Ultimele două argumente transmise către `fillText` și `strokeText` determină poziția de început a linie de bază, care este linia pe care se așează literele. Puteți modifica poziția orizontală prin setarea proprietății `textAlign` la valorile `"end"` sau `"center"` și poziția verticală prin setarea `textBaseline` la `"top"`, `"middle"` sau `"bottom"`.

{{index "pie chart example"}}

Vom reveni asupra diagramei noastre și a problemei de etichetare a feliilor în [exercițiile](canvas#exercise_pie_chart) de la sfârșitul acestui capitol.

## Imagini

{{index "vector graphics", "bitmap graphics"}}

În grafica pe computer, se face de regulă distincție între grafica _vectorială_ și grafica _bitmap_. Prima este ceea ce am făcut până acum în acest capitol - specificarea unei imagini prin furnizarea unei descrieri logice a formei sale. Grafica bitmap, pe de altă parte, nu specifică figurile ci funcționează cu date despre pixeli (rastere de puncte colorate).

{{index "load event", "event handling", "img (HTML tag)", "drawImage method"}}

Metoda `drawImage` ne permite să desenăm pixeli pe canvas. Aceste date despre pixeli pot provine dintr-un element `<img>` sau dintr-un alt canvas_pie_chart. Exemplul de mai jos crează un element `<img>` separat și încarcă un fișier imagine în el. Dar nu poate începe imediat să deseneze pe baza acestei imagini deoarece e posibil ca browserul să nu fi terminat încărcarea ei. Pentru a rezolva problema, înregistrăm un handler pentru evenimentul `"load"` și începem desenarea doar după ce imaginea a fost încărcată.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/hat.png";
  img.addEventListener("load", () => {
    for (let x = 10; x < 200; x += 30) {
      cx.drawImage(img, x, 10);
    }
  });
</script>
```

{{index "drawImage method", scaling}}

Implicit, `drawImage` va desena imaginea la dimensiunea sa originală. Dar putem să transmitem încă două argumente prin care să specificăm o lățime și o înălțime diferite.

Când `drawImage` primește _nouă_ argumente, ea poate fi utilizată pentru a desena doar un fragment din imagine. Argumentele de la al doilea la al cincilea specifică dreptunghiul din imaginea-sursă (x, y, width, height) care urmează să fie copiat, iar argumentele de la al șaselea la al nouălea specifică dreptunghiul din canvas ân care se va efectua copierea.

{{index "player", "pixel art"}}

Această tehnică poate fi utilizată pentru a împacheta mai multe imagini (sprites) într-un singur fișier și apoi să desenați doar partea de care aveți nevoie. De exemplu, iată imaginea unui caracter dintr-un joc în mai multe posturi:

{{figure {url: "img/player_big.png", alt: "Various poses of a game character",width: "6cm"}}}

{{index [animation, "platform game"]}}

Alternând postura pe care o desenăm, putem genera o animație care simulează deplasarea caracterului.

{{index "fillRect method", "clearRect method", clearing}}

Pentru a anima o imagine pe un canvas, metoda `clearRect` se va dovedi utilă. Această metodă se aseamănă cu `fillRect`, dar în loc să coloreze dreptunghiul, șterge pixelii anterior desenați.

{{index "setInterval function", "img (HTML tag)"}}

Știm că fiecare _sprite_, fiecare subimagine, are lățimea de 24 pixeli și înălțimea de 30 de pixeli. Codul de mai jos încarcă imaginea și apoi setează un interval pentru desenarea cadrului următor:

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/player.png";
  let spriteW = 24, spriteH = 30;
  img.addEventListener("load", () => {
    let cycle = 0;
    setInterval(() => {
      cx.clearRect(0, 0, spriteW, spriteH);
      cx.drawImage(img,
                   // source rectangle
                   cycle * spriteW, 0, spriteW, spriteH,
                   // destination rectangle
                   0,               0, spriteW, spriteH);
      cycle = (cycle + 1) % 8;
    }, 120);
  });
</script>
```

{{index "remainder operator", "% operator", [animation, "platform game"]}}

Bindingul `cycle` reține poziția în animație. Pentru fiecare cadru, acesta este incrementat și apoi mutat în intervalul 0-7 prin utilizarea operatorului modulo. Apoi, acest binding este utilizat pentru a calcula coordonata x a imaginii pentru postura curentă din imagine.

## Transformarea

{{index transformation, mirroring}}

{{indexsee flipping, mirroring}}

Dar dacă vrem ca să deplasăm caracterul spre stânga, nu spre dreapta? Am putea desena un alt set de imagini, desigur. Dar am putea modifica exemplul nostru pentru a desena imaginile în ordine inversă.

{{index "scale method", scaling}}

Apelul metodei `scale` determină scalarea a tot ceea ce este desenat ulterior. Această metodă primește doi parametri, unul pentru scala orizontală și celelalt pentru scala verticală.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.scale(3, .5);
  cx.beginPath();
  cx.arc(50, 50, 40, 0, 7);
  cx.lineWidth = 3;
  cx.stroke();
</script>
```

{{if book

Din cauza apelului metodei `scale`, cercul va fi de trei ori mai lat și pe jumătate de înalt.

{{figure {url: "img/canvas_scale.png", alt: "A scaled circle",width: "6.6cm"}}}

if}}

{{index mirroring}}

Scalarea va influența toate aspectele imaginii desenate. Scalarea cu un factor negativ va inversa imaginea. Oglindirea se va efectua în raport cu punctul (0,0), adică se va oglindi și sistemul de coordonate. Când se aplică un factor de scală pe orizontală egal cu -1, o imagine desenată în poziția x egală cu 100 va fi desenată în poziția care era referită de -100.

{{index "drawImage method"}}

Deci, pentru a oglindi o imagine nu putem doar să adăugăm `cx.scale(-1, 1)` înaintea apelului metodei `drawImage` deoarece imaginea noastră va fi mutată în afara canvasului, unde nu va fi vizibilă. Ați putea ajusta coordonatele transmise către `drawImage` pentru a compensa, desenând imaginea la poziția x -50 în loc de zero. O altă soluție, care nu necesită ca codul de desenare să știe despre scalare, este să ajustați axa în raport cu care se realizează scalarea.

{{index "rotate method", "translate method", transformation}}

Există și alte metode, pe lângă `scale` care influențează sistemul de coordonate al canvasului. Puteți roti figurile desenate ulterior prin apelul metodei `rotate` și le puteți muta cu ajutorul metodei `translate`. Faptul interesant - și creator de confuzie - este că aceste transformări sunt _stivuite_, adică fiecare este realizată relativ la transformarea anterioară.

{{index "rotate method", "translate method"}}

Deci, dacă translatăm de două ori pe orizontală cu 10 pixeli, totul va fi desenat cu 20 pixeli mai spre dreapta. Dacă mai întâi mutăm centrul sistemului de coordonate în (50,50) și apoi rotim cu 20 de grade (aproximativ 0.1π radiani), acea rotație va avea loc în jurul punctului (50,50).

{{figure {url: "img/transform.svg", alt: "Stacking transformations",width: "9cm"}}}

{{index coordinates}}

Dar dacă _mai întâi_ rotim cu 20 de grade și _apoi_ translatăm cu (50,50), translația va avea loc în sistemul de coordonate rotit și deci, va produce o orientare diferită. Ordinea în care se aplică transformările este relevantă.

{{index axis, mirroring}}

Pentru a oglindi o imagine relativ la o linie verticală ce trece prin coordonata x putem proceda astfel:

```{includeCode: true}
function flipHorizontally(context, around) {
  context.translate(around, 0);
  context.scale(-1, 1);
  context.translate(-around, 0);
}
```

{{index "flipHorizontally method"}}

Mutăm axa y în poziția în care vrem să facem oglindirea, aplicăm oglindirea apoi mutăm axa y înapoi în poziția inițială, după oglindire. Imaginea de mai jos explică motivu pentru care această abordare funcționează:

{{figure {url: "img/mirror.svg", alt: "Mirroring around a vertical line",width: "8cm"}}}

{{index "translate method", "scale method", transformation, canvas}}

Aceasta prezintă sistemele de coordonate înainte și după oglindirea în raport cu linia centrală. Triunghiurile sunt numerotate pentru a ilustra fiecare pas. Dacă desenăm un triunghi într-o poziție x pozitivă, poziția implicită ar fi cea în care se află triunghiul 1. Un apel către `flipHorizontally` mai întâi efectuează o translatare spre dreapta, prin care obținem triunghiul 2. Apoi se efectuează scalarea, oglindit triunghiul în poziția 3. Aceasta nu este poziția în care ar fi fost plasat dacă am fi oglindit în raport cu linia dată. Al doilea apel către `translate` repară problema - "renunță" la translatarea inițială și triunghiul 4 apare exact în poziția dorită.

Acum putem desena un caracter oglindit în poziția (100,0) prin oglindirea lumii din jurul caracterului, relativ la linia verticală ce trece prin centrul său.

```{lang: "text/html"}
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/player.png";
  let spriteW = 24, spriteH = 30;
  img.addEventListener("load", () => {
    flipHorizontally(cx, 100 + spriteW / 2);
    cx.drawImage(img, 0, 0, spriteW, spriteH,
                 100, 0, spriteW, spriteH);
  });
</script>
```

## Stocarea și curățarea transformărilor

{{index "side effect", canvas, transformation}}

Transformările sunt persistente. Tot ceea ce desenăm după ce am desenat caracterul oglindit va fi de asemenea oglindit. Ceea ce ar putea fi inconvenient.

Putem să salvăm transformarea curentă, apoi să desenăm și să facem alte transformări, iar apoi să restaurăm vechea transformare. De regulă aceasta este abordarea corectă pentru o funcție care trebuie să transforme temporar sistemul de coordonate. Mai întâi, salvăm transformarea pe care o utiliza codul care a apelat funcția. Apoi funcție își execută operațiile, adăugând mai multe transformări peste transformarea curentă. Apoi, resetăm la transformarea inițială.

{{index "save method", "restore method", [state, "of canvas"]}}

Metodele `save` și `restore` ale contextului 2D al canvasului pot fi folosite pentru gestiunea transformărilor. Conceptual, ele gestionează o stivă a stărilor transformărilor. Când apelați `save`, starea curentă este salvată în stivă iar când apelați `restore`, starea din vârful stivei este eliminată și apoi utilizat ca și transformarea curentă a contextului. Puteți apela si `resetTransform` pentru a reseta complet transformarea.

{{index "branching recursion", "fractal example", recursion}}

Funcția `branch` din exemplul următor ilustrează ce puteți realiza cu o funcție care modifică transformarea și apoi apelează o altă funcție (în acest caz pe sine), care continuă desenarea cu transformarea dată.

Această funcție desenează o formă asemănătoare unui arbore, mutând centrul sistemului de coordonate la sfârșitul liniei și apoi se apelează de două ori, mai întâi rotită spre stânga și apoi spre dreapta. Fiecare apel reduce lungimea liniei care va fi desenată și recursivitatea se oprește când lungimea liniei este mai mică de 8.

```{lang: "text/html"}
<canvas width="600" height="300"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  function branch(length, angle, scale) {
    cx.fillRect(0, 0, 1, length);
    if (length < 8) return;
    cx.save();
    cx.translate(0, length);
    cx.rotate(-angle);
    branch(length * scale, angle, scale);
    cx.rotate(2 * angle);
    branch(length * scale, angle, scale);
    cx.restore();
  }
  cx.translate(300, 0);
  branch(60, 0.5, 0.8);
</script>
```

{{if book

Rezultatul este un fractal simplu.

{{figure {url: "img/canvas_tree.png", alt: "A recursive picture",width: "5cm"}}}

if}}

{{index "save method", "restore method", canvas, "rotate method"}}

Dacă apelurile către `save` și `restore` ar fi lipsit, cel de al doilea apel al `branch` ar fi fost făcut cu poziția și rotația primului apel. Nu ar fi fost conectat la ramura curentă ci la cea mai interioară și mai din dreapta desenată de primul apel. Forma rezultată ar fi interesantă, dar nu ar fi un arbore.

{{id canvasdisplay}}

## Înapoi la joc

{{index "drawImage method"}}

Acum știm destule despre desenarea pe canvas ca să începem să lucrăm la un sistem de afișare bazat pe canvas pentru jocul din [capitolul anterior](game). Noul afișaj nu va mai afișa doar dreptunghiuri colorate. În loc de aceasta vom utiliza `drawImage` pentru a desena imagini care reprezintă elementele jocului.

{{index "CanvasDisplay class", "DOMDisplay class", [interface, object]}}

Vom defini un alt tip de obiect pentru afișare, numit `CanvasDisplay` și care va avea aceeași interfață ca și `DOMDisplay` din [capitolul ?](game#domdisplay), adică, va conține metodele `syncState` și `clear`. 

{{index [state, "in objects"]}}

Obiectul stochează puțin mai multă informație decât `DOMDisplay`. În loc să utilizeze poziția de derulare a elementului său DOM, acesta își gestionează propriul viewport, ceea ce ne spune la care parte a nivelului privim la un moment dat. În final, menține o proprietate `flipPlayer` astfel încât atunci când jucătorul se oprește, el este orientat în direcția în care a fost efectuată ultima mișcare.

```{sandbox: "game", includeCode: true}
class CanvasDisplay {
  constructor(parent, level) {
    this.canvas = document.createElement("canvas");
    this.canvas.width = Math.min(600, level.width * scale);
    this.canvas.height = Math.min(450, level.height * scale);
    parent.appendChild(this.canvas);
    this.cx = this.canvas.getContext("2d");

    this.flipPlayer = false;

    this.viewport = {
      left: 0,
      top: 0,
      width: this.canvas.width / scale,
      height: this.canvas.height / scale
    };
  }

  clear() {
    this.canvas.remove();
  }
}
```

Metoda `syncState` mai întâi calculează un nou viewport și apoi desenează scena jocului în poziția corespunzătoare.

```{sandbox: "game", includeCode: true}
CanvasDisplay.prototype.syncState = function(state) {
  this.updateViewport(state);
  this.clearDisplay(state.status);
  this.drawBackground(state.level);
  this.drawActors(state.actors);
};
```

{{index scrolling, clearing}}

Spre deosbire de `DOMDisplay`, acest tip de display _trebuie_ să redeseneze fundalul la fiecare actualizare. Deoarece figurile de pe canvas sunt doar pixeli, după ce îi desenăm nu avem o modalitate efcieintă de a-i muta (sau elimina). Singura modalitate de a actualiza afișarea canvas este de a curăța și redessena scena. Probabil că am și derulat, ceea ce necesită modificarea poziției fundalului.

{{index "CanvasDisplay class"}}

Metoda `updateViewport` este similară metodei `scrollPlayerIntoView` a obiectului `DOMDisplay`. Ea verifică dacă jucătorul este prea aproape de margine și deplasează viewportul dacă da.

```{sandbox: "game", includeCode: true}
CanvasDisplay.prototype.updateViewport = function(state) {
  let view = this.viewport, margin = view.width / 3;
  let player = state.player;
  let center = player.pos.plus(player.size.times(0.5));

  if (center.x < view.left + margin) {
    view.left = Math.max(center.x - margin, 0);
  } else if (center.x > view.left + view.width - margin) {
    view.left = Math.min(center.x + margin - view.width,
                         state.level.width - view.width);
  }
  if (center.y < view.top + margin) {
    view.top = Math.max(center.y - margin, 0);
  } else if (center.y > view.top + view.height - margin) {
    view.top = Math.min(center.y + margin - view.height,
                        state.level.height - view.height);
  }
};
```

{{index boundary, "Math.max function", "Math.min function", clipping}}

Apelurile către `Math.max` și `Math.min` ne asigură că viewportul nu va afișa spațiu în afara nivelului. `Math.max(x, 0)` ne asigură că rezultatul nu este negativ. `Math.min`, în mod simmilar, garantează că valoarea rămâne sub o anumită limită.

Când curățăm afișajul, vom utiliza o culoare ușor diferită, mai luminoasă dacă jocul a fost câștigat sau mai întunecată dacă jocul a fost pierdut.

```{sandbox: "game", includeCode: true}
CanvasDisplay.prototype.clearDisplay = function(status) {
  if (status == "won") {
    this.cx.fillStyle = "rgb(68, 191, 255)";
  } else if (status == "lost") {
    this.cx.fillStyle = "rgb(44, 136, 214)";
  } else {
    this.cx.fillStyle = "rgb(52, 166, 251)";
  }
  this.cx.fillRect(0, 0,
                   this.canvas.width, this.canvas.height);
};
```

{{index "Math.floor function", "Math.ceil function", rounding}}

Pentru a desena fundalul, trecem prin pătratele vizibile în viewport, utilizând aceeași tehnică pe care am utilizat-o în metoda `touches` din [capitolul anterior](game#touches).

```{sandbox: "game", includeCode: true}
let otherSprites = document.createElement("img");
otherSprites.src = "img/sprites.png";

CanvasDisplay.prototype.drawBackground = function(level) {
  let {left, top, width, height} = this.viewport;
  let xStart = Math.floor(left);
  let xEnd = Math.ceil(left + width);
  let yStart = Math.floor(top);
  let yEnd = Math.ceil(top + height);

  for (let y = yStart; y < yEnd; y++) {
    for (let x = xStart; x < xEnd; x++) {
      let tile = level.rows[y][x];
      if (tile == "empty") continue;
      let screenX = (x - left) * scale;
      let screenY = (y - top) * scale;
      let tileX = tile == "lava" ? scale : 0;
      this.cx.drawImage(otherSprites,
                        tileX,         0, scale, scale,
                        screenX, screenY, scale, scale);
    }
  }
};
```

{{index "drawImage method", sprite, tile}}

Pătratele care nu sunt goale for fi desenate cu `drawImage`. Imaginea `otherSprites` conține toate imaginile utilizate pentru elemente, altele decât jucătorul. Ea conține, de la stânga la dreapta, imaginea pentru perete, pentru lava și pentru o monedă.

{{figure {url: "img/sprites_big.png", alt: "Sprites for our game",width: "1.4cm"}}}

{{index scaling}}

Pătratele de fundal au dimensiunea de 20 x 20 pixeli deoarece vom utiliza aceeași scală pe care am utilizat-o pentru `DOMDisplay`. Prin urmare, offset-ul pentru pătratele de lava este 20 (valoarea bindingului `scale` iar offsetul pentru pereți este 0.

{{index drawing, "load event", "drawImage method"}}

Nu ne concentrăm pe a aștepta ca imaginea cu sprite-urile să se încarce. Apelul `drawImage` cu o imagine care nu a fost încă încărcată nu face nimic. Prin urmare, poate că nu reușim să desenăm corespunzător jocul pentru primele câteva cadre, cât timp imaginea încă se încarcă, dar asta nu este o problemă demnă de luat în considerare. Deoarece actualizăm ecranul continuu, scena corectă va apărea imediat ce se termină încărcarea.

{{index "player", [animation, "platform game"], drawing}}

Caracterul în mișcare prezentat anterior va fi utilizat pentru a reprezenta jucătorul. Codul care îl desenează trebuie să aleagă sprite-ul corect și direcția, pe baza mișcării curente a jucătorului. Primele 8 sprite-uri conțin animația deplasării. Când jucătorul se mișcă de-a lungul unei podele, ciclăm printre ele pe baza timpului curent. Vrem să comutăm imaginea la fiecare 60 milisecunde, astfel că împărțim timpul la 60 mai întâi. Când jucătorul stă, desenăm cel de al nouălea sprite. În timpul salturilor, identificate prin faptul că viteza pe verticală este nenulă, utilizăm cel de al zecelea (și cel mai din dreapta sprite).

{{index "flipHorizontally function", "CanvasDisplay class"}}

Deoarece sprite-urile sunt puțin mai late decât obiectul jucător - 24 în loc de 16 pixeli pentru a adăuga spațiu pentru picioare și mâini - metoda trebuie să ajusteze coordonata x cu o cantitate dată (`playerXOverlap`).

```{sandbox: "game", includeCode: true}
let playerSprites = document.createElement("img");
playerSprites.src = "img/player.png";
const playerXOverlap = 4;

CanvasDisplay.prototype.drawPlayer = function(player, x, y,
                                              width, height){
  width += playerXOverlap * 2;
  x -= playerXOverlap;
  if (player.speed.x != 0) {
    this.flipPlayer = player.speed.x < 0;
  }

  let tile = 8;
  if (player.speed.y != 0) {
    tile = 9;
  } else if (player.speed.x != 0) {
    tile = Math.floor(Date.now() / 60) % 8;
  }

  this.cx.save();
  if (this.flipPlayer) {
    flipHorizontally(this.cx, x + width / 2);
  }
  let tileX = tile * width;
  this.cx.drawImage(playerSprites, tileX, 0, width, height,
                                   x,     y, width, height);
  this.cx.restore();
};
```

Metoda `drawPlayer` este apelată din `drawActors`, care are responsabilitatea de a desena toți actorii din joc.

```{sandbox: "game", includeCode: true}
CanvasDisplay.prototype.drawActors = function(actors) {
  for (let actor of actors) {
    let width = actor.size.x * scale;
    let height = actor.size.y * scale;
    let x = (actor.pos.x - this.viewport.left) * scale;
    let y = (actor.pos.y - this.viewport.top) * scale;
    if (actor.type == "player") {
      this.drawPlayer(actor, x, y, width, height);
    } else {
      let tileX = (actor.type == "coin" ? 2 : 1) * scale;
      this.cx.drawImage(otherSprites,
                        tileX, 0, width, height,
                        x,     y, width, height);
    }
  }
};
```

Când desenăm altceva decât jucătorul, consultăm tipul pentru a determina offsetul correct al sprite-ului. Pătratu pentru lava este plasat la offset 20 iar pătratul pentru monedă la 40 (dublul `scale`).

{{index viewport}}

Trebuie să scădem poziția viewportului când calculăm poziția actorului deoarece (0,0) pe canvasul nostru corespunde coordonatelor colțului stânga-sus al viewportului, nu coordonatelor stânga-sus ale nivelului. Am fi putut utiliza și `translate` pentru aceasta. Oricare abordare funcționează.

{{if interactive

Documentul de mai jos conectează nou afișaj în `runGame`:

```{lang: "text/html", sandbox: game, focus: yes, startCode: true}
<body>
  <script>
    runGame(GAME_LEVELS, CanvasDisplay);
  </script>
</body>
```

if}}

{{if book

{{index [game, screenshot], [game, "with canvas"]}}

Cu aceasta am finalizat noul sistem de afișare. Jocul rezultat ar trebui să arate similar cu imaginea de mai jos:

{{figure {url: "img/canvas_game.png", alt: "The game as shown on canvas",width: "8cm"}}}

if}}

{{id graphics_tradeoffs}}

## Alegerea unei itnerfețe grafice

Când aveți de generat grafică în browser, puteți alege între HTML, SVG și canvas. Nu există o abordare unică, cea mai bună în toate situațiile. Fiecare opțiune are propriile puncte tari și puncte slabe.

{{index "text wrapping"}}

HTML are avantajul de a fi simplu. De asemenea, se integrează bine cu textul. Atât SVG cât și canvas vă permit să adăugați text, dar nu vă ajută să poziționați acel text sau să îl scrieți pe mai multe linii. Într-o imagine HTML este mult mai ușor să includeți blocuri de text.

{{index zooming, SVG}}

SVG poate fi utilizat pentru a crea grafică foarte clară, care arată bine la orice nivel de zoom. Spre deosebire de HTML, este conceput pentru desenare și astfel, este mult mai potrivit pentru acest scop.

{{index [DOM, graphics], SVG, "event handling", ["data structure", tree]}}

Atât HTML cât și CSS construiesc o structură de date (DOM) care reprezintă imaginea. Astfel, elementele pot fi modificate după ce au fost desenate. Dacă trebuie să modificați în mod repetat o mică parte dintr-o imagine mai mare, ca și răspuns la acțiunile utilizatorului sau ca și parte a unei animații, utilizarea unui canvas ar putea fi nedorit de costisitoare. DOM permite și înregistrarea unor handlere pentru evenimentele mouseului asupra fiecărui element al imaginii (chiar și pentru formele desenate cu SVG). Acest lucru nu este posibil cu canvas.

{{index performance, optimization}}

Dar abordarea bazată pe pixeli a canvasului poate fi un avantaj atunci când avem de desenat un număr imens de elemente mici. Faptul că nu se construiește o structură de date ci doar se redesenează peste aceeași suprafață de pixeli permite un cost computațional mai scăzut pentru canvas.

{{index "ray tracer"}}

Există și efecte, cum ar fi randarea unei scene pixel cu pixel (utilizând tehnica "ray-tracing") sau post procesarea imaginii în JavaScript (blurare sau distorsionare), care nu sunt gestionate realist decât într-o abordare bazată pe pixeli.

În unele cazuri, ați putea dori să combinați aceste tehnici. De exemplu, ați putea alege să desenați cu SVG  sau canvas, dar să afișați informația textuală prin poziționarea unui element HTML peste figură.

{{index display}}

Pentru aplicațiile simple, nu prea contează care dintre interfețe o alegeți. Afișajul pe care l-am construit pentru jocul nostru în acest capitol putea fi implementat utilizând oricare dintre aceste trei tehnologii de grafică deoarece nu trebuie să deseneze text, să gestioneze interacțiunea cu mouseul sau să lucreze cu un număr extraordinar de mare de elemente.

## Rezumat

În acest capitol am discutat despre tehnici de desenare în browser, cu atenție specială pe elementul `<canvas>`.

Un nod canvas reprezintă o zonă din document în care programul nostru poate desena. Desenarea se realizează cu ajutorul unui obiect pentru contextul desenării, creat de către metoda `getContext`.

Interfața 2D pentru desenare ne permite să umplem sau să conturăm diferite forme. Proprietatea `fillStyle` a contextului determină modul de umplere a formelor. Proprietățile `strokeStyle` și `lineWidth` controlează modul în care sunt desenate liniile.

Dreptunghiurile și bucățile de text pot fi desenate prin apelul unei singure metode. Metodele `fillRect` și `strokeRect` desenează dreptunghiuri, iar metodele `fillText` și `strokeText` desenează text. Pentru a crea forme neobișnuite, mai întâi trebuie să construim o cale.

{{index stroking, filling}}

Apelul metodei `beginPath` începe o nouă cale. Aveți la dispoziție metode pentru a adăuga linii si curbe la calea curentă. Când ați terminat de construit o cale, o puteți umple cu metoda `fill` sau o puteți contura cu metoda `stroke`.

Copierea pixelilor dintr-o imagine sau din alt canvas în canvasul nostru se poate realiza cu metoda `drawImage`. Implicit, această metodă desenează toată imaginea sursă, dar furnizându-i mai mulți parametri, puteți copia doar o anumită porțiune din imagine. Am utlizat această tehnică în jocul nostru prin copierea posturilor individuale ale personajului din joc dintr-o imagine care conținea mai multe asemenea posturi.

Transformările ne permit să desenăm o formă în mai multe orientări. Un context de desenare 2D are o transformare curentă care poate fi modificată cu metodele `translate`, `scale` și `rotate` și restaurată cu metoda `restore`.

Când afișăm o animație pe canvas, metoda `clearRect` poate fi folosită pentru a curăța o parte din canvas înainte de a-l redesena.

## Exerciții

### Figuri geometrice

{{index "shapes (exercise)"}}

Scrieți un program care desenează următoarele figuri geometrice pe un canvas:

{{index rotation}}

1. Un trapez (un dreptunghi cu o latură mai lungă decât cea opusă)

2. Un diamant roșu (un pătrat rotit 45 de grade sau ¼π radians)

3. O linie în zig-zag

4. O spirală formată din 100 de segmente

5. O stea galbenă

{{figure {url: "img/exercise_shapes.png", alt: "The shapes to draw",width: "8cm"}}}

Pentru a le desena pe ultimele două, ați putea avea nevoie să consultați explicația pentru `Math.cos` și `Math.sin` în [capitolul ?](dom#sin_cos), care descrie cum puteți obține coordonatele pe un cerc folosind aceste funcții.

{{index readability, "hard-coding"}}

Vă recomand să creați câte o funcție pentru fiecare figură geometrică. Transmiteți ca parametri poziția și, opțional, alte proprietăți, cum ar fi dimensiunea și numărul de puncte. Alternativa, care înseamnă hard-codarea numerelor peste tot prin cod, tinde să transforme codul în ceva inutil de dificil în a fi citit și modificat.

{{if interactive

```{lang: "text/html", test: no}
<canvas width="600" height="200"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");

  // Your code here.
</script>
```

if}}

{{hint

{{index [path, canvas], "shapes (exercise)"}}

Trapezul (1) este cel mai ușor de desenat folosind o cale. Alegeți un centru corespunzător de coordonate și apoi plasați cele patru colțuri în jurul centrului.

{{index "flipHorizontally function", rotation}}

Diamantul (2) poate fi desenat direct, cu o cale, sau într-un mod interesant, folosind transformarea `rotate`. Pentru a utiliza rotația, va trebui să aplicați o tehnică similară cu cea pe care am utilizat-o în funcția `flipHorizontally`. Deoarece vreți să rotiți în jurul centrului pătratului și nu în jurul punctului (0,0), trebuie mai întâi să tranaslatați centrul de coordonate în punctul ales, folsind funcția `translate`, apoi să aplicați cele patru rotații și apoi să translatați înapoi centrul de coordonate.

Asigurați-vă că resetați transformarea după desenarea fiecărei figuri geometrice care o folosește.

{{index "remainder operator", "% operator"}}

Pentru linia în zxig-zag (3) este nepractic să apelați `lineTo` pentru fiecare segment de dreaptă. Ați putea utiliza o buclă. Pe fiecare iterație ați putea desena două segmente de dreaptă (dreapta și apoi stânga), sau doar unul, caz în care va trebui să folosiți paritatea contorului buclei pentru a determina dacă trebuie să vă deplasați spre stânga sau spre dreapta.

Veți avea nevoie de o buclă și pentru spirală (4). Dacă desenați o serie de puncte, cu fiecare punct deplasându-se mai departe pe un cerc în jurul centrului spiralei, veți obține un cerc. Dacă însă modificați raza cercului pe care plasați punctul curent și vă rotiți de mai multe ori în jurul centrului, rezultatul va fi o spirală.

{{index "quadraticCurveTo method"}}

Steaua (5) ilustrată este construită folosind linii `quadraticCurveTo`. Puteți să o desenați și folosind linii drepte. Împărțiți cercul în 8 felii pentru a desena o stea cu 8 colțuri, apoi desenați linii între aceste puncte care se curbează spre centrul stelei. Cu ajutorul `quadraticCurveTo`, veți putea utiliza centrul ca și punct de control.

hint}}

{{id exercise_pie_chart}}

### Diagrama plăcintă

{{index label, text, "pie chart example"}}

[Anterior](canvas#pie_chart) în acest capitol, am văzut un exemplu în care am desenat o diagrama "pie". Modificați acest program astfel încât numele fiecărei categorii este afișat lângă felia care o reprezintă. Încercați să găsiți o modalitate plăcută de a poziționa automat acest text, care să funcționeze și pentru alte seturi de date. Puteți presupune că felia corespunzătoare fiecărei categorii este suficient de mare pentru a permite loc suficient pentru eticheta categoriei.

Ați putea avea nevoie din nou de `Math.sin` și `Math.cos`, care sunt descrise în [capitolul ?](dom#sin_cos).

{{if interactive

```{lang: "text/html", test: no}
<canvas width="600" height="300"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let total = results
    .reduce((sum, {count}) => sum + count, 0);
  let currentAngle = -0.5 * Math.PI;
  let centerX = 300, centerY = 150;

  // Add code to draw the slice labels in this loop.
  for (let result of results) {
    let sliceAngle = (result.count / total) * 2 * Math.PI;
    cx.beginPath();
    cx.arc(centerX, centerY, 100,
           currentAngle, currentAngle + sliceAngle);
    currentAngle += sliceAngle;
    cx.lineTo(centerX, centerY);
    cx.fillStyle = result.color;
    cx.fill();
  }
</script>
```

if}}

{{hint

{{index "fillText method", "textAlign property", "textBaseline property", "pie chart example"}}

Va trebui să apelați `fillText` și să setați pentru proprietățile contextului `textAlign` și `textBaseline` valori adecvate pentru a plasa textul unde doriți.

O modalitate de poziționare a etichetelor ar fi să plasați textul pe dreapta care trece prin centrul diagramei și mijlocul feliei. Nu cred că ar arăta foarte bine să plasați textul direct pe conturul diagramei ci să îl mutați cu un anumit număr de pixeli spre exterior.

Unghiul corespunzător respectivei linii este `currentAngle + 0.5 * sliceAngle`. Codul de mai jos determină o poziție pe linia anterioară la 120 de pixeli de centrul diagramei:

```{test: no}
let middleAngle = currentAngle + 0.5 * sliceAngle;
let textX = Math.cos(middleAngle) * 120 + centerX;
let textY = Math.sin(middleAngle) * 120 + centerY;
```

Pentru `textBaseline`, valoarea `"middle"` este probabil adecvată când folosiți această abordare. Valoarea pentru `textAlign` depinde de care parte a cercului vă aflați. În stânga, ar trebui să fie `"right"`, iar în dreapta ar trebui să fie `"left"`, astfel încât textul să fie în exteriorul diagramei.

{{index "Math.cos function"}}

Dacă nu sunteți siguri de care parte a cercului este un anume unghi, consultați din nou explicația din [capitolul ?](dom#sin_cos). Cosinusul unui unghi ne spune de fapt de care parte a cercului ne aflăm, în raport cu linia verticală ce trece prin centrul său.

hint}}

### O minge care sare

{{index [animation, "bouncing ball"], "requestAnimationFrame function", bouncing}}

Utilizați tehnica `requestAnimationFrame` din [capitolul ?](dom#animationFrame) și [capitolul ?](game#runAnimation) pentru a desena o cutie cu o minge care sare în interiorul ei. Mingea se va mișca la o viteză constantă și se va reflecta de pe laturile cutiei când le lovește.

{{if interactive

```{lang: "text/html", test: no}
<canvas width="400" height="400"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");

  let lastTime = null;
  function frame(time) {
    if (lastTime != null) {
      updateAnimation(Math.min(100, time - lastTime) / 1000);
    }
    lastTime = time;
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);

  function updateAnimation(step) {
    // Your code here.
  }
</script>
```

if}}

{{hint

{{index "strokeRect method", animation, "arc method"}}

O cutie este ușor de desenat folosind `strokeRect`. Definiți un binding sau două pentru latura cutiei, după cum doriți, pentru a memora dimensiunile ei. Pentru a defini o minge rotundă, definiți o cale și apelați `arc(x, y, radius, 0, 7)`, care va crea un arc ce va depăși un cerc complet și apoi umpleți acea cale.

{{index "collision detection", "Vec class"}}

Pentru a modela poziția și viteza mingii, puteți utiliza clasa `Vec` din [capitolul ?](game#vector)[ (care ete disponibilă pe această pagină]{if interactive}. Dați mingii o viteză inițialăși pentru fiecare cadru, multiplicați acea viteză cu durata de timp scursă. Când mingea se apropie prea mult de un perete vertical, inversați componenta x a vitezei. Similar, inversați componenta y când bila se lovește de un perete orizontal.

{{index "clearRect method", clearing}}

După ce determinați noua poziție și viteză a bilei, utilizați `clearRect` pentru a curăța scena și a o redesena utilizând noua poziție.

hint}}

### Oglindire precalculată

{{index optimization, "bitmap graphics", mirror}}

Un aspect nedorit al transformărilor este acela că ele încetinesc desenarea hărților de pixeli. Poziția și dimensiunea fiecărui pixel trebuie să fie transformată și, deși este posibil ca browserele să devină mai inteligente în viitor relativ la transformări, în prezent ele cauzează o creștere măsurabilă în timpul necesar pentru desenarea pixelilor.

Într-un joc ca al nostru, în care desenăm un singur personaj care se transform, aceasta nu este o problemă. Dar imaginați-vă că trebuie să desenăm sute de personaje sau mii de particule rezultate dintr-o explozie.

Gândiți-vă la o modalitate de a desena caracterul oglindit fără să mai încărcați o imagine și fără să faceți apeluri `drawImage` pentru fiecare cadru.

{{hint

{{index mirror, scaling, "drawImage method"}}

Cheia soluției este faptul că putem utiliza un element canvas ca și imagine sursă când utilizăm `drawImage`. Putem crea un element suplimentar `<canvas>`, fără să îl adăugăm la document și să desenăm imaginea oglindită în el, o singură dată. Când desenăm un cadru, doar copiem imaginea deja inversată în canvasul principal.

{{index "load event"}}

Trebuie să aveți grijă deoarece imaginile nu se încarcă instantaneu. Desenăm imaginea oglindită o singură dată, dar dacă o desenăm înainte ca imaginea să fie încărcată, nu va fi desenat nimic. Un handler pentru evenimentul `"load"` ar fi util în acest caz, pe extra canvas. Astfel, canvasul va putea fi utilizat ca și sursă de desenare imediat (doar că va fi gol până când desenăm personajul pe el).

hint}}

