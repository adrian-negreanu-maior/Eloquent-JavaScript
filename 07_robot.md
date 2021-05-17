{{meta {load_files: ["code/chapter/07_robot.js", "code/animatevillage.js"], zip: html}}}

# Proiect: Un Robot

{{quote {author: "Edsger Dijkstra", title: "The Threats to Computing Science", chapter: true}

[...] întrebarea dacă mașinile pot gândi [...] este cam la fel de relevantă ca și întrebarea dacă submarinele pot înota.

quote}}

{{index "artificial intelligence", "Dijkstra, Edsger"}}

{{figure {url: "img/chapter_picture_7.jpg", alt: "Picture of a package-delivery robot", chapter: framed}}}

{{index "project chapter", "reading code", "writing code"}}

În capitolele de proiecte, ne oprim puțin din a parcurge teorie și vom lucra împreună la un program. Teoria este necesară pentru a învăța să programați, dar este la fel de important să puteți citi și înțelege programe.

Proiectul din acest capitol este despre a construi un automat, un mic program care realizeazî o sarcină într-o lume virtuală. Automatul nostru este un robot pentru livrarea corespondenței care va ridica și va lăsa colete.

## Meadowfield

{{index "roads array"}}

Orașul Meadowfield nu este foarte mare. El constă din 11 locuri cu 14 drumuri între ele. Poate fi descris ca un array de drumuri:

```{includeCode: true}
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

{{figure {url: "img/village2x.png", alt: "The village of Meadowfield"}}}

Rețeaua de drumuri a orașului formează un graf. Un graf este o colecție de puncte (locuri din oraș) cu linii între ele (drumurile). Acest graf este lumea în care robotul nostru se va deplasa.

{{index "roadGraph object"}}

Nu este foarte ușor să lucrăm cu array-ul de stringuri. Ceea ce ne interesează sunt destinațiile în care putem ajunge dintr-un loc dat. Haideți să convertim lista de drumuri într-o structură de date care, pentru fiecare loc, ne precizează ce destinații putem atinge.

```{includeCode: true}
function buildGraph(edges) {
  let graph = Object.create(null);
  function addEdge(from, to) {
    if (graph[from] == null) {
      graph[from] = [to];
    } else {
      graph[from].push(to);
    }
  }
  for (let [from, to] of edges.map(r => r.split("-"))) {
    addEdge(from, to);
    addEdge(to, from);
  }
  return graph;
}

const roadGraph = buildGraph(roads);
```

Dat fiind un array de muchii, `buildGraph` crează un map care, pentru fiecare nod, memorează array-ul de noduri conectate.

{{index "split method"}}

Mai întâi, utilizăm metoda `split` pentru a trece de la stringuri, care au forma `"Start-End"`, la array-uri de două elemente ce conțin `Start` și `End` ca și stringuri separate.

## Sarcina

Robotul nostru se va deplasa prin oraș. Există colete în diferite locuri, fiecare destinat unei alte locații. Robotul ridică acele colete când ajunge la ele și le livrează atunci când ajunge la destinația lor.

Automatul trebuie să decidă, în fiecare nod, care este următorul nod în care se va deplasa. Își va termina sarcina în momentul în care toate pachetele au fost livrate.

{{index simulation, "virtual world"}}

Pentru a putea simula acest proces, trebuie să definim o lume virtuală care îl descrie. Acest model ne va spune unde este robotul și unde sunt coletele. Când robotul a decis să se deplaseze într-o nouă locație, trebuie să actualizăm modelul pentru a reflecta noua situație.

{{index [state, in objects]}}

Dacă gândiți in termenii programării orientate obiect, primul impuls ar fi să începeți prin a defini obiecte pentru diferitele elemente ale lumii virtuale: o clasă pentru robot, una pentru colet și poate una pentru destinații. Acestea ar putea să conțină proprietăți care descriu starea lor curentă, cum ar fi stiva de colete dintr-o locație pe care le-am putea modifica atunci când actualizăm lumea virtuală.

Dar această abordare este greșită.

Cel puțin este _de regulă_ greșită. Faptul că o anumită entitate pare a fi un obiect nu înseamnă automat că trebuie să fie un obiect în program. Scrierea din reflex a unor clase pentru fiecare concept din aplicație tinde să vă conducă la o colecție de obiecte interconectate care au fiecare propria stare interna, schimbătoare. Asemenea programe sunt adesea greu de înțeles și ușor de stricat.

{{index [state, in objects]}}

haideți să condensăm mai bine starea orașului la un set minimal de valori care o definesc. Există locația curentă a robotului și colecția de pachete încă nelivrate, fiecare dintre ele având o locație curentă și o adresă de destinație. Atât!

{{index "VillageState class", "persistent data structure"}}

Haideți să procedăm astfel încât să nu _modificăm_ starea când robotul se mișcă, ci să calculăm o _nouă_ stare pentru situația de după deplasarea robotului.

```{includeCode: true}
class VillageState {
  constructor(place, parcels) {
    this.place = place;
    this.parcels = parcels;
  }

  move(destination) {
    if (!roadGraph[this.place].includes(destination)) {
      return this;
    } else {
      let parcels = this.parcels.map(p => {
        if (p.place != this.place) return p;
        return {place: destination, address: p.address};
      }).filter(p => p.place != p.address);
      return new VillageState(destination, parcels);
    }
  }
}
```

Acțiunea are loc în metoda `move`. Mai întâi se verifică dacă există un drum de la locația curentă până la destinație și, dacă nu, returnează vechea stare deoarece această mutare nu ar fi validă.

{{index "map method", "filter method"}}

Apoi se crează o nouă stare cu destinația ca și noua poziție a robotului. Dar va trebui și să creeze un nou set de colete, coletele pe care robotul le transportă (cele care se vor afla în noua locație a robotului) trebuie să fie mutate în noua locație. Iar coletele care sunt adresate noii locație trebuie să fie livrate - adică, să fie eliminate din setul de pachete nelivrate. Apelul funcției `map` rezolvă problema mișcării, iar apelul funcției `filter` se va ocupa de livrare.

Obiectele pentru colete nu sunt modificate când sunt transportate ci sunt recreate. Metoda `move` ne dă o nouă stare a orașului, dar lasă vechea stare intactă.

```
let first = new VillageState(
  "Post Office",
  [{place: "Post Office", address: "Alice's House"}]
);
let next = first.move("Alice's House");

console.log(next.place);
// → Alice's House
console.log(next.parcels);
// → []
console.log(first.place);
// → Post Office
```

Mișcarea cauzează livrarea coletului și aceasta se reflectă în starea următoare. Dar starea inițială încă descrie situația în care robotul este la oficiul poștal și coletul nu este livrat.

## Date persistente

{{index "persistent data structure", mutability, ["data structure", immutable]}}

Structurile de date care nu se modifică se numesc _imutabile_ sau _persistente_. Ele se comportă asemănător stringurilor și numerelor în sensul că sunt ceea ce sunt și rămân așa, în loc să conțină date diferite la momente diferite.

În JavaScript, aproape orice poate fi modificat astfel încât lucrul cu valori care se presupune că sunt persistente necesită câteva restricții. Există o funcție numită  `Object.freeze` care modifică un obiect astfel încât orice scriere în proprietățile sale este ignorată. Puteți folosi această metodă pentru a vă asigura că obiectele nu sunt modificate. Înghețarea cere extra efort din partea computerului și ignorarea actualizărilor ar putea crea cam la fel de multă confuzie ca și actualizarea greșită. Așa că ar fi de preferat să informăm că un anume obiect nu ar trebui modificat și să sperăm că ceiallți programatori care vor interacționa cu obiectul își vor aminti.

```
let object = Object.freeze({value: 5});
object.value = 10;
console.log(object.value);
// → 5
```

De ce evit să modific obiecte când este evident că limbajul se așteaptă să fac asta?

Deoarece mă ajută să înțeleg programele. Este vorba din nou despre gestiunea complexității. Când obiectele din sistemul meu sunt fixe, stabile, pot considera operațiile asupra lor izolate - deplasarea la casa lui Alice dintr-o stare de start va produce întotdeauna aceeași stare. Când obiectele se modifică în timp, acest comporttament adaugă o dimensiune complet nouă a complexității acestui mod de a gândi.

Pentru un sistem de mici dimensiuni, ca și cel pe care îl construim în acest capitol, am putea gestiona complexitatea adăugată. Dar cea mai importantă limitare cu privire la tipul de sisteme pe care le putem construi este dată de capacitatea noastră de a înțelege. Orice face codul mai ușor de înțeles va face posibil să construim un sistem mai ambițios.

Din păcate, deși înțelegerea unui sistem bazat pe structuri de date persistente este mai ușoară, _conceperea_ lui, mai ales atunci  când limbajul de programare nu este de prea mare ajutor, poate fi mai dificilă. Vom căuta oportunități de a utiliza structuri de date persistente pe parcursul acestei cărți, însă vom utiliza și structuri ce se modifică în timp.

## Simularea

{{index simulation, "virtual world"}}

Un robot de livrare analizează lumea virtuală și decide în ce direcție să se deplaseze. Prin urmare, putem spune că robotul este o funcție care primește obiectul `VillageState` și returnează numele unei locații învecinate.

{{index "runRobot function"}}

Deoarece vrem ca robotul să fie înzestrat cu memorie, astfel încât să poată crea și executa planuri, trebuie să îi transmitem și memoria și să îi permitem să returneze o memorie nouă. Astfel, robotul va returna un obiect ce va conține atât destinația de deplasare cât și valoarea memoriei care îi va fi transmisă din nou atunci când va fi apelat data viitoare.

```{includeCode: true}
function runRobot(state, robot, memory) {
  for (let turn = 0;; turn++) {
    if (state.parcels.length == 0) {
      console.log(`Done in ${turn} turns`);
      break;
    }
    let action = robot(state, memory);
    state = state.move(action.direction);
    memory = action.memory;
    console.log(`Moved to ${action.direction}`);
  }
}
```

Sa vedem ce trebuie să facă un robot pentru a "rezolva" o stare dată. Trebuie să culeagă toate coletele prin vizitarea fiecărei locații care are cel puțin un colet și să le livreze prin vizitarea oricărei locații căreia îi este adresat cel puțin un colet, dar numai după ce a ridicat coletul.

Care ar fi cea mai stupidă strategie care ar putea funcționa? Robotul ar putea să se deplaseze la întâmplare în fiecare mișcare. Adică, cu mare probabilitate, va ridica la un moment dat toate pachetele și la un moment ulterior ar ajunge în locația de livrare a fiecărui pachet.

{{index "randomPick function", "randomRobot function"}}

O asemenea abordare ar putea arăta cam așa:

```{includeCode: true}
function randomPick(array) {
  let choice = Math.floor(Math.random() * array.length);
  return array[choice];
}

function randomRobot(state) {
  return {direction: randomPick(roadGraph[state.place])};
}
```

{{index "Math.random function", "Math.floor function", [array, "random element"]}}

`Math.random()` va returna întotdeauna un număr real din intervalul [0,1). Multiplicând acest număr cu lungimea unui array și apoi apelând `Math.floor`, obținem un index aleator al unui element al array-ului.

Deoarece acest robot nu trebuie să își amintească nimic, el poate ignora cel de-al doilea argument (funcțiile JavaScript pot fi apelate cu argumente asemănătoare fără a produce efecte) și omite proprietatea `memory` din obiectul returnat.

Pentru a pune acest robot sofisticat la lucru, mai întâi avem nevoie de o modalitate de a crea o nouă stare cu câteva colete. O metodă statică (scrisă în codul de mai jos prin adăugarea directă a unei proprietăți la constructor) este un loc bun pentru a adăuga acestă funcționalitate.

```{includeCode: true}
VillageState.random = function(parcelCount = 5) {
  let parcels = [];
  for (let i = 0; i < parcelCount; i++) {
    let address = randomPick(Object.keys(roadGraph));
    let place;
    do {
      place = randomPick(Object.keys(roadGraph));
    } while (place == address);
    parcels.push({place, address});
  }
  return new VillageState("Post Office", parcels);
};
```

{{index "do loop"}}

Nu ne interesează coletele care sunt trimise la aceeași locație ca și cea în care se află. Din acest motiv, bucla `do` analizează noi locații când primește o locație care este egală cu adresa.

Să pornim o lume virtuală.

```{test: no}
runRobot(VillageState.random(), randomRobot);
// → Moved to Marketplace
// → Moved to Town Hall
// → …
// → Done in 63 turns
```

Robotul va face multe mutări pentru a livra coletele deoarece nu planifică grozav de bine. Vom rezolva această problemă în curând.

{{if interactive

Pentru o perspectivă mai plăcută asupra simulării, puteți folosi funcția `runRobotAnimation` disponibilă în [mediul de programare al acestui capitol](https://eloquentjavascript.net/code/#7). Aceasta rulează simularea dar, în loc să afișeze text, afișează robotul ce se deplasează pe harta orașului.

```{test: no}
runRobotAnimation(VillageState.random(), randomRobot);
```

Modul în care am implementat `runRobotAnimation` va rămâne deocamdată un mister, dar după ce veți parcurge [capitolele următoare](dom) ale acestei cărți, care discută integrarea JavaScript în browserele web, o să vă dați seama cum funcționează.

if}}

## Ruta camionului de poștă

{{index "mailRoute array"}}

Ar trebui să ne descurcăm mult mai bine decât robotul aleator. O îmbunătățire ușor de implementat ar fi să ne inspirăm din modul în care funcționează livrarea corespondenței în lumea reală. Dacă găsim o rută care trece prin toate locațiile din oraș, robotul ar putea parcurge acea rută de două ori, ceea ce ar garanta finalizarea sarcinii. Mai jos vă prezint o asemenea rută (începând de la oficiul poștal):

```{includeCode: true}
const mailRoute = [
  "Alice's House", "Cabin", "Alice's House", "Bob's House",
  "Town Hall", "Daria's House", "Ernie's House",
  "Grete's House", "Shop", "Grete's House", "Farm",
  "Marketplace", "Post Office"
];
```

{{index "routeRobot function"}}

Pentru a implementa robotul care urmărește o rută, va trebui să folosim memoria robotului. Robotul va memora restul rutei în memorie și la fiecare deplasare va elimina primul element al rutei.

```{includeCode: true}
function routeRobot(state, memory) {
  if (memory.length == 0) {
    memory = mailRoute;
  }
  return {direction: memory[0], memory: memory.slice(1)};
}
```

Acest robot va fi mult mai rapid. Va avea nevoie de cel mult 26 de mutări (de ouă ori pe ruta de 13 pași), dar de regulă va face chiar mai puține mișcări.

{{if interactive

```{test: no}
runRobotAnimation(VillageState.random(), routeRobot, []);
```

if}}

## Determinarea drumului

Nu aș considera că urmarea unei rute fixe este un comportament inteligent. Robotul ar putea lucra mai eficient dacă și-ar ajuta comportamentul la munca efectivă pe care trebuie să o efectueze.

{{index pathfinding}}

Pentru aceasta, ar trebui să fie capabil să se miște deliberat către un colet sau către o locație în care coletul ar trebui să fie livrat. Astfel, chiar dacă scopul este la mai mult de o mutare, va fi necesară o funcție pentru detectarea rutei.

Problema determinării unei rute într-un graf este o problemă clasică de _căutare_. Putem spune dacă o soluție dată (o rută) este validă dar nu o putem calcula direct, așa cum procedăm pentru 2 + 2, de exemplu. De fapt, va trebui să creem soluții potențiale, până când găsim una care funcționează.

Numărul de rute posibile într-un graf poate fi foarte mare. Dar atunci când căutăm o rută de la _A_ la _B_ ne interesează doar rutele care pornesc din _A_. De asemenea, nu ne interesează rutele care vizitează același loc de mai multe ori - cu siguranță acelea nu sunt cele mai eficiente rute. Astfel reducem numărul de rute care trebuie să fie considerate.

De fapt, ne interesează _cea mai scurtă_ rută. Deci vrem să considerăm rute scurte înainte de a considera rute mai lungi. O abordare bună ar fi să "creștem" rutele din punctul de plecare explorând fiecare locație ce poate fi atinsă dar nu a fost încă considerată, până când ruta își atinge destinația. Astfel, vom explora doar rutele care sunt potențial interesante și vom determina cea mai scurtă rută către destinație (sau una dintre cele mai scurte rute, dacă există mai multe rute de aceeași lungime).

{{index "findRoute function"}}

{{id findRoute}}

Iată cum ar putea arăta funcția care realizează aceasta:

```{includeCode: true}
function findRoute(graph, from, to) {
  let work = [{at: from, route: []}];
  for (let i = 0; i < work.length; i++) {
    let {at, route} = work[i];
    for (let place of graph[at]) {
      if (place == to) return route.concat(place);
      if (!work.some(w => w.at == place)) {
        work.push({at: place, route: route.concat(place)});
      }
    }
  }
}
```

Explorarea trebuie să se realizeze în ordinea corectă - locațiile atinse mai întâi trebuie să fie primele care sunt explorate. Nu putem explora imediat o locație deoarece asta ar însemna că și locațiile care pot fi atinse direct din această locație vor fi explorate imediat, chiar dacă ar putea exista o altă cale mai scurtă care nu a fost încă explorată.

Prin urmare, funcția va reține o _listă de lucru_. Acesta este un array cu locațiile care urmează să fie explorate, împreună cu ruta care ne-a adus în această locație. Vom începe din poziția de start cu o rută goală.

Căutarea este realizată prin considerarea următorului item din listă și explorarea sa, ceea ce înseamnă că toate drumurile ce pleacă din acea locație sunt considerate. Dacă unul dintre ele este obiectivul, putem returna o rută. În caz contrar, dacă nu am atins această locație anterior, vom adăuga un nou item la listă. Dacă locația a fost atinsă anterior, deoarece ne interesează doar rutele cele mai scurte, fie am găsit o rută mai lungă decât cea anterioară, fie o rută de aceeași lungime și nu va fi necesar să o explorăm.

Vă puteți imagina această abordarea ca pe o rețea de rute cunoscute care crește din locația inițială, uniform în toate direcțiile (dar fără bucle de revenire). Imediat ce un fir ajunge în locația de destinație, acest fir este urmărit până la locația de start și astfel obținem ruta de interes.

{{index "connected graph"}}

Codul nostru nu tratează situația în care nu mai putem continua explorarea deoarce știm că graful nostru este _conex_, adică orice locație poate fi atinsă din orice altă locație. Întotdeauna vom putea determina o rută între două locații și căutarea nu va putea eșua.

```{includeCode: true}
function goalOrientedRobot({place, parcels}, route) {
  if (route.length == 0) {
    let parcel = parcels[0];
    if (parcel.place != place) {
      route = findRoute(roadGraph, place, parcel.place);
    } else {
      route = findRoute(roadGraph, place, parcel.address);
    }
  }
  return {direction: route[0], memory: route.slice(1)};
}
```

{{index "goalOrientedRobot function"}}

Acest robot își va utiliza valoarea memoriei ca o listă de direcții în care să se miște, ca și robotul ce urmărea o cale fixată. Când acea listă se golește, va trebui să determine ce urmează să facă. Robotul va considera primul colet nelivrat și, dacă acel pachet nu a fost încă ridicat, determină o rută către el. După ce coletul a fost ridicat, va trebui să fie livrat, astfel încât robotul va calcula o rută către adresa de livrare.

{{if interactive

Haideți să vedem ce se întâmplă.

```{test: no, startCode: true}
runRobotAnimation(VillageState.random(),
                  goalOrientedRobot, []);
```

if}}

Acest robot își finalizează sarcina de a livra 5 colete în aproximativ 16 mutări. Ceea ce e puțin mai bine decît in cazul `routeRobot` dar nu optimal.

## Exerciții

### Măsurarea unui robot

{{index "measuring a robot (exercise)", testing, automation, "compareRobots function"}}

Este greu să comparăm roboții doar lăsându-i să rezolve câteva scenarii. Poate că un robot a primit o sarcină mai ușoară sau genul de sarcină pe care este specializat să o rezolve eficient.

Scrieți o funcție `compareRobots` care primește doi roboți și memoriile lor inițiale. Funcția va genera 100 de sarcini și va lăsa fiecare robot să le rezolve pe toate. La final, va afișa numărul mediu de pași necesar fiecărui robot pentru a rezolva toate sarcinile.

Pentru echitate, fiecare task trebuie să fie rezolvat de ambii roboți, în loc să se genereze sarcini diferite pentru fiecare robot.

{{if interactive

```{test: no}
function compareRobots(robot1, memory1, robot2, memory2) {
  // Your code here
}

compareRobots(routeRobot, [], goalOrientedRobot, []);
```
if}}

{{hint

{{index "measuring a robot (exercise)", "runRobot function"}}

Va trebui să scrieți o variantă a funcției `runRobot` care, în loc să afișeze evenimentele în consolă, să returneze numărul de pași necesari robotului pentru completarea sarcinii.

Apoi, funcția de măsurare, într-o buclă, va putea să genereze noi stări și să numere pașii pe care îi face fiecare robot. După ce a făcut destule măsurători, funcția va utiliza `console.log` pentru a afișa media pentru fiecare robot, care se va calcula ca raportul dintre numărul total de pași efectuați și numărul total de măsurători.

hint}}

### Eficiența robotului

{{index "robot efficiency (exercise)"}}

Puteți scrie un robot care finalizează livrarea mai repede decât `goalOrientedRobot`? Dacă observați comportamentul acelui robot, ce manevre stupide face acesta? Cum l-ați putea îmbunătăți?

Dacă ați rezolvat excercițiul, utilizați funcția `compareRobots` pentru a verifica că ați îmbunătățit într-adevăr robotul.

{{if interactive

```{test: no}
// Your code here

runRobotAnimation(VillageState.random(), yourRobot, memory);
```

if}}

{{hint

{{index "robot efficiency (exercise)"}}

Cea mai mare limitare a `goalOrientedRobot` este că ia în calcul întotdeuna un singur colet. Se va deplasa adesea de-a lungul orașului deoarece coletul pe care îl va ridica va trebui să îl livreze în partea opusă a orașului, chiar dacă alte obiecte ar putea fi mult mai apropiate.

O soluție posibilă ar fi să calculăm rutele pentru toate pachetele și să o alegem pe cea mai scurtă. Putem obține rezultate și mai bune dacă, în cazul în care există mai multe rute scurte, le preferăm pe cele care ridică pachete în locul celor care livrează pachete.

hint}}

### Grup persistent

{{index "persistent group (exercise)", "persistent data structure", "Set class", "set (data structure)", "Group class", "PGroup class"}}

Majoritatea structurilor de date disponibile într-un mediu JavaScript standard nu sunt foarte potrivite pentru utilizarea persistentă. Array-urile au metodele `slice` și `concat` care ne permit să creem ușor array-uri noi fără să modificăm array-ul original. Dar `Set` nu are metode pentru crearea unui nou set cu un item adăugat sau eliminat.

Scrieți o nouă clasă, `PGroup`, similară clasei `Group` din [capitolul ?](object#groups), care memorează un set de valori. Ca și `Group`, va avea metodele `add`, `delete` și `has`.


Metoda `add` va returna o _nouă instanță_ `PGroup` cu noul membru adăugat, lăsând neschimbată instanța originală. Similar, `delete` va crea o nouă instanță, fără membrul primit ca și argument. 

Clasa va trebui să funcționeze pentru valori de orice tip, nu doar pentru stringuri. Nu este necesar să fie eficientă atunci când lucrează cu un volum mare de valori.

{{index [interface, object]}}

Constructorul nu trebuie să facă parte din interfața clasei (deși probabil că o să îl utilizați intern). În locul lui, trebuie să existe o instanță goală, `PGroup.empty`, pe care o veți utiliza ca și valoare de pornire.

{{index singleton}}

De ce ați dori să aveți o valoare `PGroup.empty` în locul unei funcții care crează un set gol nou de fiecare dată?

{{if interactive

```{test: no}
class PGroup {
  // Your code here
}

let a = PGroup.empty.add("a");
let ab = a.add("b");
let b = ab.delete("a");

console.log(b.has("b"));
// → true
console.log(a.has("b"));
// → false
console.log(b.has("a"));
// → false
```

if}}

{{hint

{{index "persistent map (exercise)", "Set class", [array, creation], "PGroup class"}}

Cel mai convenabil mod de a reprezenta setul de valori membre este tot un array, deoarece array-urile sunt ușor de copiat.

{{index "concat method", "filter method"}}

Când o valoare se adaugă la un grup, puteți crea un nou grup cu o copie a array-ului original care conține valoarea adăugată (utrilizând de exemplu `concat`). Când o valoare este eliminată, o veți filtra din array.

Constructorul clasei poate primi un asemenea array ca și argument și îl va memora ca și singura proprietate a instanței. Acest array nu va fi actualizat niciodată.

{{index "static method"}}

Pentru a adăuga o proprietate (`empty`) la un constructor, dar care nu este o metodă, trebuie să o adăugați consttructorului după definiția clasei, ca și proprietate obișnuită.

Aveți nevoie doar de o singură instanță `empty` deoarece toate grupurile goale sunt identice și instanțele clasei nu vor fi modificate. Puteți crea oricâte grupuri din acel grup gol, fără a-l afecta.

hint}}
