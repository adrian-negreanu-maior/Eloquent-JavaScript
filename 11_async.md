{{meta {load_files: ["code/crow-tech.js", "code/chapter/11_async.js"]}}}

# Programarea asincronă

{{quote {author: "Laozi", title: "Tao Te Ching", chapter: true}

Cine poate aștepta în liniște în timp ce noroiul se formează?\
Cine poate sta nemișcat până în momentul acțiunii?

quote}}

{{index "Laozi"}}

{{figure {url: "img/chapter_picture_11.jpg", alt: "Două ciori pe o ramură", chapter: framed}}}

Partea centrală a unui computer, partea care îndeplinește pașii din programele noastre, se numește _procesor_. Programele pe care le-am studiat până acum sunt lucruri care țin procesorul ocupat până când își finalizează execuția. Viteza cu care poate fi executat ceva cum ar fi o buclă care procesează numere depinde în cea mai mare parte de viteza procesorului.

{{index [memory, speed], [network, speed]}}

Dar multe programe interacționează cu lucruri care nu țin de procesor. De exemplu, ele ar putea comunica printr-o rețea sau să solicite date de pe harddisk - ceea ce este mult mai lent decât preluarea datelor din memorie.

Când se întâmplă așa ceva, ar fi o rușine să lăsăm procesorul neutilizat - ar putea să execute alte lucruri între timp. Parțial, aceasta este gestionată de către sistemul de operare care va comuta procesorul între mai multe programe care se execută. Dar aceasta nu ajută atunci când vrem ca _un singur_ program să poată să facă progres în timp ce așteaptă răspunsul la o cerere în rețea.

## Asincronicitatea

{{index "synchronous programming"}}

Într-un model de programare sincron, lucrurile se întâmplă pe rând. Când apelați o funcție care execută o acțiune de lungă durată, ea va returna doar la finalului acțiunii. Aceasta suspendă execuția programului pe durata acțiunii.

{{index "asynchronous programming"}}

Un model _asincron_ permite să se întâmple mai multe lucruri în același timp. Când porniți o acțiune, programul continuă să ruleze. Când acțiunea se încheie, programul este informat și are acces la rezultat (de exemplu, datele citite de pe harddisk).

Putem compara programarea sincronă cu cea asincronă folosind un mic exemplu: un program care solicită două resurse din rețea și apoi le combină.

{{index "synchronous programming"}}

Într-un mediu sincron, în care funcția de solicitare returnează doar după ce și-a finalizat munca, cel mai simplu mod de a realiza această sarcină este de a efectua cererile una după alta. Aceasta are dezavantajul că cea de a doua cerere va începe doar după finalizarea primeia. Timpul total de execuție va fi cel puțin suma celor doi timpi de răspuns.

{{index parallelism}}

Soluția la această problemă într-un sistem sincron este de a porni fire de execuție adiționale (threaduri). Un _fir de execuție (thread)_ este un alt program a cărui execuție ar putea fi combinată cu cea a altor programe de către sistemul de operare - deoarece majoritatea computerelor moderne conțin mai multe procesoare, mai multe fire de execuție s-ar putea executa în același timp, pe procesoare diferite. Un al doilea thread ar putea începe cea de a doua cerere și apoi ambele threaduri ar aștepta după rezultate, după care vor fi sincronizate pentru a se combina rezultatele lor.

{{index CPU, blocking, "asynchronous programming", timeline, "callback function"}}

În diagrama care urmează, liniile groase reprezintă timpul petrecut de către program rulând normal, iar cele subțiri reprezintă timpii de așteptare după răspunsul din rețea. Într-un model sincron, timpul consumat de rețea este _parte_ a liniei de timp pentru un fir de execuție dat. În modelul asincron, pornirea unei acțiuni în rețea cauzează conceptual o _divizare_ pe linia temporală. Programul care a cauzat acțiunea continuă să se execute și acțiunea se desfășoară în paralel, programul fiind notificat atunci când acțiunea se încheie.

{{figure {url: "img/control-io.svg", alt: "Fluxul de control în programarea sincronă și asincronă",width: "8cm"}}}

{{index ["control flow", asynchronous], "asynchronous programming", verbosity}}

Un alt mod de a descrie diferența este că așteptarea pentru ca acțiunea să se încheie este _implicită_ în modelul sincron și _explicită_, sub controlul nostru, în modelul asincron.

Asincronicitatea decurge în ambele moduri. Permite exprimarea mai ușoară programelor care nu se potrivesc cu modelul liniar de control, dar poate face exprimarea programelor care se potrivesc cu modelul liniar mai ciudată. Vom vedea câteva modalități de a adresa ciudățeniile mai târziu în acest capitol.

Ambele platforme importante pentru programarea JavaScript - browserele și Node.js - permit implementarea asincronă a operațiilor de durată, în locul utilizării threadurilor. Deoarece utilizarea threadurilor este de o dificultate celebră (înțelegerea efectelor unui program este mult mai dificilă atunci când acesta execută mai multe lucruri în paralel), programarea asincronă este privită ca un lucru bun.

## Crow tech

Majoritatea dintre noi știm că ciorile sunt păsări foarte inteligente. Ele pot utiliza instrumente, își amintesc lucruri și chiar le comunică între ele.

Ceea ce mulți dintre noi nu știm este că ele sunt capabile de multe lucruri pe care le ascund cu grijă de noi. Un reputat expert în corvide (oarecum excentric) mi-a spus că tehnologia ciorilor nu este mult în spatele tehnologie umane și se apropie.

De exemplu, ciorile pot construi dispozitive computaționale. Acestea nu sunt dispozitive electronice, ca și ale noastre, ci operează cu ajutorul unor mici insecte, o specie înrudită cu termitele, care a dezvoltat o relație simbiotică cu ciorile. Păsările le oferă mâncare iar insectele își construiesc și își operează coloniile lor complexe, care cu ajutorul indivizilor efectuează acțiuni computaționale.

Asemenea colonii se află de regulă în cuiburi mari și utilizate de mult timp. Păsările și insectele conlucrează pentru a construi o rețea de structuri din argilă, ascunse printre ramurile cuibului, în care insectele trăiesc și lucrează.

Pentru comunicarea cu alte dispozitive, aceste mașini utilizează semnale luminoase. Ciorile încorporează bucăți de material reflectiv în poziții speciale iar insectele le utilizează pentru a reflecta lumina la un alt cuib, codificând datele ca o secvență de flash-uri rapide. Prin urmare, doar cuiburile care au o linie vizuală directă pot comunica.

Prietenul nostru expert în corvide a realizat harta rețelei de cuiburi de ciori din Hières-sur-Amby, pe malurile râului Rhône. Această hartă prezintă cuiburile și conexiunile lor:

{{figure {url: "img/Hieres-sur-Amby.png", alt: "O rețea de cuiburi de ciori într-un mic sat"}}}

Într-un uimitor exemplu de evoluție convergentă, computerele ciorilor rulează JavaScript. În acest capitol vom scrie câteva funcții de bază pentru rețeaua lor.

## Callbacks

{{indexsee [function, callback], "callback function"}}

O abordare în programarea asincronă este de a furniza un extra argument funcțiilor care realizează o acțiune lentă, o _funcție callback_. Acțiunea începe, iar la încheiere este apelată funcția callback cu rezultatul.

{{index "setTimeout function", waiting}}

Ca exemplu, funcția `setTimeout`, disponibilă atât în browsere cât și în Node.js, așteaptă un număr dat de milisecunde și apoi apelează o funcție.

```{test: no}
setTimeout(() => console.log("Tick"), 500);
```

Așteptarea nu este de regulă un tip de activitate foarte importantă, dar poate fi utilă pentru operații cum ar fi actualizarea unei animații sau verificarea dacă ceva durează mai mult decât o anumită durata.

Executarea mai multor acțiuni asincrone utilizînd callback-uri înseamnă că trebuie să continuați să transmiteți noi funcții pentru a gestiona continuarea computațiilor după fiecare acțiune.

{{index "hard disk"}}

Majoritatea computerelor din cuiburile ciorilor au o structură de stocare pe termen lung, unde bucăți de informație sunt gravate pe ramuri astfel încât să poată fi returnate ulterior. Gravarea și găsirea ulterioară a unei bucăți de date are o durata neneglijabilă, astfel încât interfața cu memoria de stocare pe termen lung este asincronă și utilizează funcții callback.

Structurile de stocare memorează bucăți JSON - data codificabile cu nume. O cioară ar putea stoca informația despre locuri în care a ascuns mâncare sub numele `"food caches"`, care ar putea memora un array de nume care se referă la alte bucăți de date, descriind valoarea stocată. Pentru a căuta un `food cache` în structurile de stocare ale cuibului _Big Oak_ o cioară ar putea rula cod ca și acesta:

{{index "readStorage function"}}

```{includeCode: "top_lines: 1"}
import {bigOak} from "./crow-tech";

bigOak.readStorage("food caches", caches => {
  let firstCache = caches[0];
  bigOak.readStorage(firstCache, info => {
    console.log(info);
  });
});
```

(Toate numele bindingurilor și stringurile au fost translatate din limbajul ciorilor în limba engleză.)

Acest stil de programare poate fi utilizat, dar nivelul de indentare crește cu fiecare acțiune asincronă deoarece intrăm într-o nouă funcție. Pentru a realiza lucruri mai complicate, cum ar fi rularea mai multor acțiuni în același timp, codul ar putea fi puțin ciudat.

Computerele din cuiburile ciorilor sunt construite să comunice utilizând perechi "cerere-răspuns". Aceasta înseamnă că un cuib trimite un mesaj unui alt cuib, care apoi trimite imediat un mesaj înapoi, confirmând primirea și eventual incluzând o replică la o întrebare din mesaj.

Fiecare mesaj este marcat cu un _tip_, care determină modul în care va fi gestionat, Codul nostru definește handler-e pentru tipuri specifice de cereri și atunci când este primită o asemenea solicitare, se apelează handler-ul pentru a produce un răspuns.

{{index "crow-tech module", "send method"}}

Interfața exportată de către modulul  `"./crow-tech"` publică funcții callback-based pentru comunicare. Cuiburile au o metodă `send` care trimite o cerere. Aceasta așteaptă numele cuibului destinație, tipul cererii și conținutul cererii ca și primele trei argumente și mai așteaptă o funcție ce va fi apelată atunci când se recepționează răspunsul, ca și al patrulea și ultimul argument.

```
bigOak.send("Cow Pasture", "note", "Let's caw loudly at 7PM",
            () => console.log("Note delivered."));
```

Dar pentru ca aceste cuiburi să fie capabile să recepționeze cererea, trebuie să definim mai întâi un tip de cerere, numit `"note"`. Codul care manevrează cererile trebuie să ruleze nu doar pe un singur computer ci pe toate computerele din toate cuiburile care pot primi mesaje de acest tip. Presupunem că o cioară a zburat în toate cuiburile și a instalat codul nostru în toate cuiburile.

{{index "defineRequestType function"}}

```{includeCode: true}
import {defineRequestType} from "./crow-tech";

defineRequestType("note", (nest, content, source, done) => {
  console.log(`${nest.name} received note: ${content}`);
  done();
});
```

Funcția `defineRequestType` definește un nou tip de cerere. Exemplul adaugă suport pentru cereri de tip `"note"`, care doar trimit o notă către un cuib dat. Implementarea noastră apelează `console.log` astfel că putem verifica dacă cererea a ajuns la destinație. Cuiburile au o proprietate `name` care memorează numele lor.

{{index "asynchronous programming"}}

Cel de-al patrulea argument al handlerului, `done` este o funcție callback care trebuie să fie apelată când cererea a fost rezolvată. Dacă am fi utilizat valoarea de retur a handlerului ca si valoare de răspuns, nu ar fi fost posibil ca handlerul să efectueze asincron acțiuni. O funcție ce se execută asincron de regulă va returna înainte de finalizarea lucrului, după ce a aranjat ca un callback să fie apelat când se va termina acțiunea. Deci, avem nevoie de un mecanism asincron - în acest caz o altă funcție de callback - pentru a semnala când răspunsul este disponibil.

Am putea spune că _asincronicitatea_ este _contagioasă_. Orice funcție care apelează o funcție care funcționează asincron trebuie să fie la rândul ei asincronă, utilizând un callback sau un mecanism similar pentru a livra rezultatul. Apelarea unui callback este întrucâtva mai elaborată și mai expusă erorilor decît simpla returnare a unei valori astfel că necesitatea de a structura părți mari din program în acest mod nu este entuziasmantă.

## Promises

Manevrarea conceptelor abstracte este adesea mai ușoară când aceste concepte pot fi reprezentate prin valori. În cazul acțiunilor asincrone, ați putea, în loc să aranjați ca o funcție să fie apelată la un moment dat în viitor, să returnați un obiect ce reprezintă acest eveniment ulterior.

{{index "Promise class", "asynchronous programming"}}

Acesta este scopul clasei standard `Promise`. Un _promise_ este o acțiune asincronă care s-ar putea finaliza la un moment dat, producând o valoare. Este capabil să notifice pe toți cei interesați când o valoare este disponibilă.

{{index "Promise.resolve function", "resolving (a promise)"}}

Cel mai simplu mod de a crea un obiect "promise" este prin a apela `Promise.resolve`. Această funcție asigură că valoarea pe care i-o transmiteți este împachetată într-o promisiune. Dacă este deja o promisiune, va fi doar returnată direct - altfel, veți primi o nouă promisiune care se încheie imediat cu valoarea transmisă ca și rezultat.

```
let fifteen = Promise.resolve(15);
fifteen.then(value => console.log(`Got ${value}`));
// → Got 15
```

{{index "then method"}}

Pentru a obține rezultatul unui _promise_ îi putem apela metoda `then`. Aceasta înregistrează o funcție callback care va fi apelată atunci când promisiunea este rezolvată și produce o valoare. Puteți adăuga mai multe callbackuri la un singur promise si ele vor fi apelate, chiar dacă le adăugați după ce promisiunea a fost rezolvată (finalizată).

Dar nu doar aceasta este acțiunea efectuată de metoda `then`. Ea va returna o altă promisiune, care va fi rezolvată la valoarea pe care funcția handler o returnează sau, dacă returnează o promisiune, așteaptă și apoi o rezolvă la rezultatul său.

Este util să vă gândiți la promisiuni ca la un fel de dispozitiv de transfer al valorilor într-o realitate asincronă. O valoare normală este pur și simplu prezentă. O valoare promisă este o valoare care ar putea fi deja prezentă sau care ar putea să apară la un moment dat. Computațiile definite ca și promisiuni acționează asupra unor asemenea valori împachetate și se execută asincron pe măsură ce valorile sunt disponibile.

{{index "Promise class"}}

Pentru a crea o promisiune, veți utiliza constructorul `Promise`. Acesta are o interfață oarecum ciudată - constructorul așteaptă o funcție ca și argument, pe care o apelează imediat, transmițându-i o funcție pe care o poate utiliza pentru a rezolva promisiunea. Funcționează astfel, în loc de a utiliza o metodă `resolve` de exemplu, astfel încât doar codul care crează promisiunea o poate rezolva.

{{index "storage function"}}

Iată cum ați putea crea o interfață bazată pe promisiuni pentru funcția `readStorage`:

```{includeCode: "top_lines: 5"}
function storage(nest, name) {
  return new Promise(resolve => {
    nest.readStorage(name, result => resolve(result));
  });
}

storage(bigOak, "enemies")
  .then(value => console.log("Got", value));
```

Această funcție asincronă returnează o valoare semnificativă. Acesta este principalul avantaj al promisiunilor - ele simplifică utilizarea funcțiilor asincrone. În loc să fim nevoiți să utilizăm callbackuri, funcțiile bazate pe promisiuni sunt similare cu cele regulate: primesc intrarea ca și argumente și returnează rezultatele. Singura diferență este că rezultatul ar putea să nu fie imediat disponibil.

## Eșecul

{{index "exception handling"}}

Computațiile JavaScript regulate pot eșua prin aruncarea unei excepții. Computațiile asincrone au adesea nevoie de ceva similar. O cerere prin rețea ar putea eșua, sau o parte de cod ce aparține unei computații asincrone ar putea arunca o excepție.

{{index "callback function", error}}

Una dintre cele mai presante probleme legate de stilul bazat pe callback-uri în programarea asincronă este dificultatea extremă cu care ne confruntăm pentru a ne asigura că erorile sunt raportate corespunzător către callback-uri.

O convenție utilizată pe larg este ca primul argument al callback-ului să fie utilizat pentru a indica eșecul acțiunii iar cel de-al doilea să conțină valoarea produsă de acțiune atunci când s-a finalizat cu succes. Aceste funcții callback trebuie să verifice întotdeauna dacă au recepționat o excepție și să se asigure că, indiferent de probleme cauzate, inclusiv excepțiile aruncate de către funcțiile pe care le apelează, sunt prinse și transmise către funcția adecvată.

{{index "rejecting (a promise)", "resolving (a promise)", "then method"}}

Promisiunile ușurează această muncă. Ele pot să fie rezolvate (acțiunea s-a finalizat cu succes) sau respinse (a eșuat). Handlerele pentru rezolvare (înregistrate cu `then`) sunt apelate doar când acțiunea se încheie cu succes iar respingerile sunt automat propagate către următoarea promisiune returnată de către `then`. Atunci când un handler aruncă o excepție, acesta cauzează automat respingerea promisiunii produse de apelul `then`. Astfel, dacă oricare element al unui lanț de acțiuni asincrone eșuează, rezultatul întregului lanț este marcat ca și respins și nici un handler pentru succes nu va mai fi apelat dincolo de locul eșecului.

{{index "Promise.reject function", "Promise class"}}

Așa cum rezolvarea unei promisiuni furnizează o valoare, respingerea ei de asemenea va furniza o valoare, de regulă denumită _motivul_ respingerii. Când o excepție într-o funcție handler cauzează o respingere, valoarea excepție este utilizată ca și motiv. Similar, când un handler returnează o promisiune care este respinsă, respingerea continuă spre următoarea promisiune. Există metoda `Promise.reject` care crează o promisiune nouă pe care o respinge imediat.

{{index "catch method"}}

Pentru a gestiona explicit aceste respingeri, promisiunile au o metoda `catch` care înregistrarea un handler ce va fi apelat când promisiunea este respinsă, similar cu modul în care handlerele `then` gestionează rezolvarea normală. De asemenea, `catch` returnează o nouă promisiune, care este rezolvată cu valoarea promisiunii originale dacă a fost rezolvată normal și cu rezultatul handlerului `catch` în caz contrar. Dacă un handler `catch` aruncă o excepție, noua promisiune va fi și ea respinsă.

{{index "then method"}}

Ca și prescurtare, `then` acceptă un handler pentru respingere ca și al doilea argument, astfel că putem preciza ambele tipuri de handlere într-un singur apel.

O funcție transmisă constructorului `Promise` primește un al doilea argument, pe lângă funcția pentru rezolvare, care poate fi utilizat pentru a respinge promisiunea.

Succesiunea de promisiuni create de către apelurile către `then` și `catch` poate fi privită ca o conductă prin care valorile asincrone sau eșecurile se deplasează. Deoarece asemenea lanțuri sunt create prin înregistrarea handlerelor, fiecare legătură are un handler pentru succes sau unul pentru respingere (sau amândouă) asociate cu ea. Handlerele care nu potrivesc tipul de ieșire (succes sau eșec) sunt ignorate. Dar cele care îl potrivesc sunt apelate și rezultatul lor determină ce fel de valoare urmează - succes când returnează o valoare care nu este o promisiune, respingere când este aruncată o excepție și rezultatul unei promisiuni când este returnată una.

```{test: no}
new Promise((_, reject) => reject(new Error("Fail")))
  .then(value => console.log("Handler 1"))
  .catch(reason => {
    console.log("Caught failure " + reason);
    return "nothing";
  })
  .then(value => console.log("Handler 2", value));
// → Caught failure Error: Fail
// → Handler 2 nothing
```

{{index "uncaught exception", "exception handling"}}

Tot așa cum o excepție netratată este gestionată de către mediu, mediile JavaScript pot detecta dacă respingerea unei promisiuni nu este gestionată și vor raporta aceasta ca fiind o eroare.

## Rețelele sunt dificile

{{index [network, reliability]}}

Ocazional, lumina nu este suficientă în sistemul de oglinzi al ciorilor pentru a transmite un semnal, sau ceva blochează calea semnalului. Este posibil ca un semnal să fie trimis, dar nu și recepționat.

{{index "send method", error, timeout}}

Prin urmare, callbackul metodei `send` nu va fi niciodată apelat, ceea ce probabil va cauza oprirea programului fără ca măcar să atenționeze că există o problemă. Ar fi frumos ca, după scurgerea unei anumite perioade de timp fără a recepționa un răspuns, cererea să expire și să raporteze un eșec.

Adesea, erorile de transmisie sunt accidente întâmplătoare, cum ar fi interferența luminii farurilor cu semnalul luminos și retransmiterea cererii ar putea conduce la succes. Haideți să modificăm funcția de cerere ca să reîncerce să trimită automat cererea de câteva ori înainte de a renunța.

{{index "Promise class", "callback function", [interface, object]}}

Și, deoarece am stabilit deja că promisiunile sunt un lucru bun, vom modifica funcția de cerere ca să returneze o promisiune. Din punct de vedere al ceea ce pot exprima, callbackurile și promisiunile sunt echivalente. Funcțiile bazate pe callback-uri pot fi împachetate ca să expună o interfață bazată pe promisiuni și invers.

Chiar și atunci când o cerere și răspunsul său au fost livrate cu succes, răspunsul ar putea indica un eșec - de exemplu, dacă cererea încearcă să utilizeze un tip de cerere care nu a fost definit sau dacă handlerul aruncă o excepție. Pentru a suporta aceasta, `send` și `defineRequestType` urmează convenția menționată anterior, în care primul argument transmis către callback este motivul eșecului, dacă există, iar cel de-al doilea este rezultatul propriu-zis.

Acestea pot fi translatate în rezolvarea sau respingerea unei promisiuni.

{{index "Timeout class", "request function", retry}}

```{includeCode: true}
class Timeout extends Error {}

function request(nest, target, type, content) {
  return new Promise((resolve, reject) => {
    let done = false;
    function attempt(n) {
      nest.send(target, type, content, (failed, value) => {
        done = true;
        if (failed) reject(failed);
        else resolve(value);
      });
      setTimeout(() => {
        if (done) return;
        else if (n < 3) attempt(n + 1);
        else reject(new Timeout("Timed out"));
      }, 250);
    }
    attempt(1);
  });
}
```

{{index "Promise class", "resolving (a promise)", "rejecting (a promise)"}}

Deoarece promisiunile pot fi rezovlate sau respinse doar o dată, acest cod va funcționa. La primul apel al `resolve` sau `reject` se determină rezultatul promisunii iar apeluri ulterioare cauzate de o cerere ce revine după ce o altă cerere a fost rezolvată sunt ignorate.

{{index recursion}}

Pentru a construi o buclă asincronă pentru reîncercări, trebuie să folosim o funcție recursivă - o buclă simplă nu ne permite să ne oprim și să așteptăm după o acțiune asincronă. Funcția `attempt` face o singură încercare de a trimite o cerere. De asemenea, setează un timp de expirare. Dacă nu a venit un răspuns după 250 milisecunde, fie încearcă din nou fie, dacă a fost cea de a treia încercare, respinge promisiunea cu o instanță a `Timeout` ca și motiv.

{{index idempotence}}

Reîncercarea după fiecare sfert de secundă și renunțarea după ce trei încercări nu au primit răspuns este evident arbitrară. Este chiar posibil ca , dacă răspunsul a venit dar handlerul a luat puțin mai mult timp, cererea să fie livrată de mai multe ori. Vom scrie handlerele ținând cont de această problemă - mesajele duplicate nu ar trebui să dăuneze.

În general, nu vom construi o rețea foarte robustă. Dar e în regulă, ciorile nu au încă așteptări foarte mari când vine vorba despre computații.

{{index "defineRequestType function", "requestType function"}}

Pentru a ne izola de callbackuri, vom defini un wrapper pentru `defineRequestType` care permite ca funcția handler să returneze o promisune sau o valoare obișnuită care va fi legată apoi de callback.

```{includeCode: true}
function requestType(name, handler) {
  defineRequestType(name, (nest, content, source,
                           callback) => {
    try {
      Promise.resolve(handler(nest, content, source))
        .then(response => callback(null, response),
              failure => callback(failure));
    } catch (exception) {
      callback(exception);
    }
  });
}
```

{{index "Promise.resolve function"}}

`Promise.resolve` este utilizat pentru a converti valoarea returnată de către `handler` într-o promisiune, dacă nu este deja.

{{index "try keyword", "callback function"}}

Observați că apelul către `handler` a fost inclus într-un bloc `try` pentru a ne asigura că orice excepție pe care o ridică direct este dată callback-ului. Aceasta ilustrează dificultatea de a gestiona corespunzător erorile cu callback-uri brute - este uțor să uitați să rutați corespunzător excepțiile și, dacă nu o faceți, eșecurile nu vor fi raportate către callbackul corect. Promisiunile fac asta automat și deci, mai puțin expus erorilor.

## Colecții de promisiuni

{{index "neighbors property", "ping request"}}

Computerul din fiecare cuib memorează un array al cuiburilor care se află in raza de transmisie, în proprietatea `neighhbors`. Pentru a verifica dacă un cuib poate fi atins, putem scrie o funcție care să transmită o cerere `"ping"` (o cerere care doar solicită un răspuns) către fiecare dintre cuiburi și să verifice care dintre ele se întoarce.

{{index "Promise.all function"}}

Când folosim colecții de promisiuni ce rulează simultan, metoda `Promise.all` poate fi folositoare. Ea returnează o promisiune care așteaptă ca toate promisiunile din array să se rezolve și apoi se rezolvă la un array de valori produse de acele promisiuni (în aceeați ordine ca și array-ul original). Dacă orice promisune este respinsă, rezultatul `Promise.all` va fi respins.

```{includeCode: true}
requestType("ping", () => "pong");

function availableNeighbors(nest) {
  let requests = nest.neighbors.map(neighbor => {
    return request(nest, neighbor, "ping")
      .then(() => true, () => false);
  });
  return Promise.all(requests).then(result => {
    return nest.neighbors.filter((_, i) => result[i]);
  });
}
```

{{index "then method"}}

Când un vecin nu este disponibil, nu vrem ca promisiunea combinată să eșueze deoarece tot nu am avea nici o informație. Astfel că funcția care este mapată peste setul de vecini pentru a-i transforma in promisiuni de cereri atașează handlere care vor produce `true` pentru cererile reușite și `false` pentru cele respinse.

{{index "filter method", "map method", "some method"}}

În handlerul pentru promisiunea combinată, metoda `filter` se utilizează pentru a elimina din arrayul `neighbors` elementele pentru care valoarea corespunzătoare este false. Aceasta utilizează faptul că metoda `filter` transmite indexul elementului curent ca și cel de-al doilea argument al funcției de filtrare (`map`, `some` și alte câteva metode similare de oridn superior procedează la fel).

## Inundarea rețelei

Faptul că oricare cuib poate comunica doar cu vecinii săi limitează mult utilitatea acestei rețele.

Pentru a transmite informația în întreaga rețea, o soluție ar fi să construim un tip de cerere care este transmis automat către toți vecinii. Apoi vecinii ar transmite către vecinii lor, până când toată rețeaua ar primi mesajul.

{{index "sendGossip function"}}

```{includeCode: true}
import {everywhere} from "./crow-tech";

everywhere(nest => {
  nest.state.gossip = [];
});

function sendGossip(nest, message, exceptFor = null) {
  nest.state.gossip.push(message);
  for (let neighbor of nest.neighbors) {
    if (neighbor == exceptFor) continue;
    request(nest, neighbor, "gossip", message);
  }
}

requestType("gossip", (nest, message, source) => {
  if (nest.state.gossip.includes(message)) return;
  console.log(`${nest.name} received gossip '${
               message}' from ${source}`);
  sendGossip(nest, message, source);
});
```

{{index "everywhere function", "gossip property"}}

Pentru a evita transmiterea infinită a aceluiași mesaj prin rețea, fiecare cuib memorează un array cu mesajele pe care le-a recepționat deja. Pentru a defini acest array folosim funcția `everywhere` care rulează cod în fiecare cuib pentru a adăuga o proprietate la obiectul `state` al cuibului, ce reține starea locală a cuibului.

Când un cuib primește un mesaj duplicat, ceea ce este foarte probabil să se întâmple de vreme ce toată lumea le transmite orbește, îl ignoră. Dar când primește un nou mesaj, îl retransmite tuturor vecinilor, cu excepția celui de la care l-a primit.

În acest fel, un mesaj se va răspândi în rețea ca o picătură de cerneală în apă. Chiar și atunci când unele conexiuni nu funcționează temporar, dacă există o rută alternativă către un cuib, mesajul va ajunge acolo.

{{index "flooding"}}

Acest mod de comunicare în rețea se numește _inundare (flooding)_ - rețeaua este inundată de către o bucată de informație până când toate nodurile sunt atinse.

{{if interactive

Putem apela `sendGossip` pentru a vedea cum se propagă un mesaj prin rețea.

```
sendGossip(bigOak, "Kids with airgun in the park");
```

if}}

## Rutarea mesajelor

{{index efficiency}}

Dacă un nod dat vrea să comunice cu un singur alt nod, inundarea nu este o abordare prea eficientă. Mai ales atunci când rețeaua este mare, aceasta va conduce la multe transferuri de date inutile.

{{index "routing"}}

O abordare alternativă ar fi setarea unui mod prin care mesajul să sară din nod în nod până ajunge la destinație. Dificultatea este legată de faptul că este necesară cunoașterea aspectului rețelei. Pentru a transmite o cerere în direcția unui nod îndepărtat, trebuie să știm care dintre noduri ne va apropia de destinație. Trimiterea mesajului în direcția greșită nu va fi de mare folos.

Deoarece fiecare nod cunoaște doar lista vecinilor săi direcți, nu are informație suficientă pentru a calcula o rută. Cumva ar trebui să răspândim informația despre aceste conexiuni către toate cuiburile, preferabil într-un mod care să permită modificarea în timp, când cuiburile sunt abandonate sau sunt create noi cuiburi.

{{index flooding}}

Putem utiliza din nou inundarea, dar în loc să verificăm dacă un mesaj dat a fost primit deja, verificăm dacă un nou set de vecini pentru un cuib dat potrivește setul curent pe care îl avem.

{{index "broadcastConnections function", "connections binding"}}

```{includeCode: true}
requestType("connections", (nest, {name, neighbors},
                            source) => {
  let connections = nest.state.connections;
  if (JSON.stringify(connections.get(name)) ==
      JSON.stringify(neighbors)) return;
  connections.set(name, neighbors);
  broadcastConnections(nest, name, source);
});

function broadcastConnections(nest, name, exceptFor = null) {
  for (let neighbor of nest.neighbors) {
    if (neighbor == exceptFor) continue;
    request(nest, neighbor, "connections", {
      name,
      neighbors: nest.state.connections.get(name)
    });
  }
}

everywhere(nest => {
  nest.state.connections = new Map();
  nest.state.connections.set(nest.name, nest.neighbors);
  broadcastConnections(nest, nest.name);
});
```

{{index JSON, "== operator"}}

Comparația utilizează `JSON.stringify` deoarece `==`, pe obiecte și arrayuri, va returna true doar dacă cei doi operanzi sunt exact aceeași valoare, ceea ce nu ne este util în acest caz. Compararea stringurilor JSON este o metodă brutală dar eficientă pentru a le compara conținutul.

Nodurile vor începe imediat să își transmită conexiunile, ceea ce ar trebui, dacă nu cumva câteva cuiburi sunt de neatins, să dea fiecărui cuib o harta a grafului rețelei curente.

{{index pathfinding}}

Un lucru pe care îl putem face într-un graf este să găsim rute, așa cum am văzut în [capitolul ?](robot). Dacă avem o rută către destinatarul unui mesaj, atunci știm în ce direcție să trimitem mesajul.

{{index "findRoute function"}}

Funcția `findRoute` de mai jos, care seamănă foarte bine cu funcția `findRoute` din [capitolul ?](robot#findRoute), caută o modalitate de a atinge un nod dat din rețea. Dar, în loc să returneze toată ruta, returnează doar pasul următor. Următorul cuib va utiliza informația sa curentă despre rețea pentru a decide unde va trimite mesajul.

```{includeCode: true}
function findRoute(from, to, connections) {
  let work = [{at: from, via: null}];
  for (let i = 0; i < work.length; i++) {
    let {at, via} = work[i];
    for (let next of connections.get(at) || []) {
      if (next == to) return via;
      if (!work.some(w => w.at == next)) {
        work.push({at: next, via: via || next});
      }
    }
  }
  return null;
}
```

Acum putem construi o funcție care va trimite mesaje la mare distanță. Dacă mesajul este adresat unui vecin direct, este livrat ca de obicei. Dacă nu, este împachetat într-un obiect și transmis unui vecin care este mai aproape de destinatar, utilizând tipul de cerere `"route"` care va determina vecinul să procedeze la fel.

{{index "routeRequest function"}}

```{includeCode: true}
function routeRequest(nest, target, type, content) {
  if (nest.neighbors.includes(target)) {
    return request(nest, target, type, content);
  } else {
    let via = findRoute(nest.name, target,
                        nest.state.connections);
    if (!via) throw new Error(`No route to ${target}`);
    return request(nest, via, "route",
                   {target, type, content});
  }
}

requestType("route", (nest, {target, type, content}) => {
  return routeRequest(nest, target, type, content);
});
```

{{if interactive

Acum putem trimite un mesaj către cuibul din turnul bisericii care se află la patru pași distanță:

```
routeRequest(bigOak, "Church Tower", "note",
             "Incoming jackdaws!");
```

if}}

{{index [network, abstraction], layering}}

Am construit mai multe straturi de funcționalitate în vârful unui sistem de comunicație primitiv pentru a-l putea utiliza convenabil. Acesta este un model simpatic(deși simplificat) al funcționării rețelelor de computere.

{{index error}}

A poprietate distinctivă a rețelelor de computere este faptul că nu sunt de încredere - abstracțiile construite în vârful lor pot ajuta, dar u puteți abstractiza erorile rețelei. Astfel încât programarea rețelelor este în mare parte despre anticiparea și tratarea eșecurilor.

## Funcții `async`

Ciorile sunt cunoscute pentru duplicarea informațiilor importante între cuiburi. Astfel, când un șoim distruge un cuib, infromația nu se pierde.

Pentru a retturna o informație dată pe care nu o are în propriul spațiu de stocare, computerul dintr-un cuib ar putea consulta la întâmplare alte cuiburi până când o găsește.

{{index "findInStorage function", "network function"}}

```{includeCode: true}
requestType("storage", (nest, name) => storage(nest, name));

function findInStorage(nest, name) {
  return storage(nest, name).then(found => {
    if (found != null) return found;
    else return findInRemoteStorage(nest, name);
  });
}

function network(nest) {
  return Array.from(nest.state.connections.keys());
}

function findInRemoteStorage(nest, name) {
  let sources = network(nest).filter(n => n != nest.name);
  function next() {
    if (sources.length == 0) {
      return Promise.reject(new Error("Not found"));
    } else {
      let source = sources[Math.floor(Math.random() *
                                      sources.length)];
      sources = sources.filter(n => n != source);
      return routeRequest(nest, source, "storage", name)
        .then(value => value != null ? value : next(),
              next);
    }
  }
  return next();
}
```

{{index "Map class", "Object.keys function", "Array.from function"}}

Deoarece `connections` este un `Map`, nu putem folosi `Object.keys`. Are o _metodă_ `keys` dar aceasta returnează un iterator nu un array. Un iterator (valoare iterabilă) poate fi convertit într-un array cu funcția `Array.from`.

{{index "Promise class", recursion}}

Chiar și cu promisiuni, codul ar fi ciudat. Mai multe acțiuni asincrone sunt înlănțuite în moduri nu prea evidente. Din nou avem nevoie de o funcție recursiva (`next`) pentru a modela parcurgerea cuiburilor.

{{index "synchronous programming", "asynchronous programming"}}

Iar ceea ce face de fapt codul este complet liniar - întotdeauna așteaptă pentru acțiunea anterioară ca să se încheie înainte de a trece la următoarea acțiune. Într-un model de programare sincron, ar fi mai simplu de exprimat.

{{index "async function", "await keyword"}}

Veștile bune sunt că JavaScript permite scrierea de cod pseudo-asincron pentru a descrie computațiile asincrone. O funcție `async` este o funcție care returnează implicit o promisiune și care poate, în corpul său să aștepte (`await`) alte promisiuni într-un mod care _pare_ sincron.

{{index "findInStorage function"}}

Putem rescrie `findInStorage` astfel:

```
async function findInStorage(nest, name) {
  let local = await storage(nest, name);
  if (local != null) return local;

  let sources = network(nest).filter(n => n != nest.name);
  while (sources.length > 0) {
    let source = sources[Math.floor(Math.random() *
                                    sources.length)];
    sources = sources.filter(n => n != source);
    try {
      let found = await routeRequest(nest, source, "storage",
                                     name);
      if (found != null) return found;
    } catch (_) {}
  }
  throw new Error("Not found");
}
```

{{index "async function", "return keyword", "exception handling"}}

O funcție `async` este marcată prin cuvântul cheie `async` utilizat înainte de cuvântul `function`. Metodele pot și ele să fie scrise `async` prin utilizarea acestui cuvânt în fața numelui lor. Când se apelează o asemenea funcție sau metodă, ea va returna o promisiune. Imediat ce corpul său returnează ceva, acea promisiune este rezolvată. Dacă aruncă o excepție, promisiunea este respinsă.

{{if interactive

```{startCode: true}
findInStorage(bigOak, "events on 2017-12-21")
  .then(console.log);
```

if}}

{{index "await keyword", ["control flow", asynchronous]}}

În interiorul unei funcții `async`, cuvântul `await` poate fi folosit în fața unei expresii pentru a aștepta ca o promisiune să fie rezolvată și doar după aceea să continue execuția funcției.

Spre deosebire de o funcție obișnuită, o  asemenea funcție nu se va mai executa de la început la sfârșit într-un singur pas. De fapt, ar putea să _înghețe_ în orice punct unde are un `await` și apoi să își reia execuția la un moment ulterior.

Pentru codul asincron netrivial, această notație este de regulă mai convenabilă decât utilizarea directă a promisiunilor. Chiar dacă trebuie să faceți o operație care nu se potrivește modelului sincron, cum ar fi să executați mai multe operații simultan, este ușor să combinați `await` cu utilizarea directă a promisiunilor.
such as perform multiple actions at the same time, it is easy to combine `await` with the direct use of promises.

## Generatori

{{index "async function"}}

Această abilitate a funcțiilor de a fi suspendate și apoi reluate nu este exclusivă funcțiilor `async`. JavaScript are și o funcționalitate numită _funcții generator_. Acestea funcționează asemănător dar fără promisiuni.

Când definiți o funcție cu `function*` ea devine un generator. Când apelați un generator, el va returna un iterator, ceea ce am folosit deja în [capitolul ?](object).

```
function* powers(n) {
  for (let current = n;; current *= n) {
    yield current;
  }
}

for (let power of powers(3)) {
  if (power > 50) break;
  console.log(power);
}
// → 3
// → 9
// → 27
```

{{index "next method", "yield keyword"}}

Inițial, când apelați `powers` funcția _îngheață_ la începutul său. De fiecare dată când apelați `next` pe iterator, funcția rulează până când atinge expresia `yield`, care o suspendă și face ca valoarea produsă să devină următoarea valoare produsă de către iterator. Când funcția returnează (ce din exemplu nu o face niciodată), iteratorul își finalizează activitatea.

Scrierea iteratorilor este adesea mai simplă decât scrierea funcțiilor generator. Iteratorul pentru clasa `Group` (din exercițiul din [capitolul ?](object#group_iterator)) poate fi scris cu acest generator:

{{index "Group class"}}

```
Group.prototype[Symbol.iterator] = function*() {
  for (let i = 0; i < this.members.length; i++) {
    yield this.members[i];
  }
};
```

```{hidden: true, includeCode: true}
class Group {
  constructor() { this.members = []; }
  add(m) { this.members.add(m); }
}
```

{{index [state, in iterator]}}

Nu mai este nevoie să creem un obiect care să rețină starea iterației - generatorii își salvează automat starea locală de fiecare dată când produc.

Asemenea expresii `yield` pot apărea doar direct în funcția generator și nu într-o funcție interioară pe care o definiți. Starea generatorului salvează, când produce, doar mediul său _local_ și poziția în care a produs.

{{index "await keyword"}}

O funcție `async` este un tip special de generator. Ea produce o promisiune când este apelată, care este rezolvată când returnează (se încheie) sau respinsă când aruncă o excepție. Întotdeauna când produce (așteaptă) o promisune, rezultatul acelei promisiuni (valoare sau excepție aruncată) este rezultatul expresiei `await`.

## Bucla de evenimente

{{index "asynchronous programming", scheduling, "event loop", timeline}}



Asynchronous programs are executed piece by piece. Each piece may start some actions and schedule code to be executed when the action finishes or fails. In between these pieces, the program sits idle, waiting for the next action.

{{index "setTimeout function"}}

So callbacks are not directly called by the code that scheduled them. If I call `setTimeout` from within a function, that function will have returned by the time the callback function is called. And when the callback returns, control does not go back to the function that scheduled it.

{{index "Promise class", "catch keyword", "exception handling"}}

Asynchronous behavior happens on its own empty function ((call stack)). This is one of the reasons that, without promises, managing exceptions across asynchronous code is hard. Since each callback starts with a mostly empty stack, your `catch` handlers won't be on the stack when they throw an exception.

```
try {
  setTimeout(() => {
    throw new Error("Woosh");
  }, 20);
} catch (_) {
  // This will not run
  console.log("Caught!");
}
```

{{index thread, queue}}

No matter how closely together events—such as timeouts or incoming requests—happen, a JavaScript environment will run only one program at a time. You can think of this as it running a big loop _around_ your program, called the _event loop_. When there's nothing to be done, that loop is stopped. But as events come in, they are added to a queue, and their code is executed one after the other. Because no two things run at the same time, slow-running code might delay the handling of other events.

This example sets a timeout but then dallies until after the timeout's intended point of time, causing the timeout to be late.

```
let start = Date.now();
setTimeout(() => {
  console.log("Timeout ran at", Date.now() - start);
}, 20);
while (Date.now() < start + 50) {}
console.log("Wasted time until", Date.now() - start);
// → Wasted time until 50
// → Timeout ran at 55
```

{{index "resolving (a promise)", "rejecting (a promise)", "Promise class"}}

Promises always resolve or reject as a new event. Even if a promise is already resolved, waiting for it will cause your callback to run after the current script finishes, rather than right away.

```
Promise.resolve("Done").then(console.log);
console.log("Me first!");
// → Me first!
// → Done
```

In later chapters we'll see various other types of events that run on the event loop.

## Asynchronous bugs

{{index "asynchronous programming", [state, transitions]}}

When your program runs synchronously, in a single go, there are no state changes happening except those that the program itself makes. For asynchronous programs this is different—they may have _gaps_ in their execution during which other code can run.

Let's look at an example. One of the hobbies of our crows is to count the number of chicks that hatch throughout the village every year. Nests store this count in their storage bulbs. The following code tries to enumerate the counts from all the nests for a given year:

{{index "anyStorage function", "chicks function"}}

```{includeCode: true}
function anyStorage(nest, source, name) {
  if (source == nest.name) return storage(nest, name);
  else return routeRequest(nest, source, "storage", name);
}

async function chicks(nest, year) {
  let list = "";
  await Promise.all(network(nest).map(async name => {
    list += `${name}: ${
      await anyStorage(nest, name, `chicks in ${year}`)
    }\n`;
  }));
  return list;
}
```

{{index "async function"}}

The `async name =>` part shows that ((arrow function))s can also be made `async` by putting the word `async` in front of them.

{{index "Promise.all function"}}

The code doesn't immediately look suspicious...it maps the `async` arrow function over the set of nests, creating an array of promises, and then uses `Promise.all` to wait for all of these before returning the list they build up.

But it is seriously broken. It'll always return only a single line of output, listing the nest that was slowest to respond.

{{if interactive

```
chicks(bigOak, 2017).then(console.log);
```

if}}

Can you work out why?

{{index "+= operator"}}

The problem lies in the `+=` operator, which takes the _current_ value of `list` at the time where the statement starts executing and then, when the `await` finishes, sets the `list` binding to be that value plus the added string.

{{index "await keyword"}}

But between the time where the statement starts executing and the time where it finishes there's an asynchronous gap. The `map` expression runs before anything has been added to the list, so each of the `+=` operators starts from an empty string and ends up, when its storage retrieval finishes, setting `list` to a single-line list—the result of adding its line to the empty string.

{{index "side effect"}}

This could have easily been avoided by returning the lines from the mapped promises and calling `join` on the result of `Promise.all`, instead of building up the list by changing a binding. As usual, computing new values is less error-prone than changing existing values.

{{index "chicks function"}}

```
async function chicks(nest, year) {
  let lines = network(nest).map(async name => {
    return name + ": " +
      await anyStorage(nest, name, `chicks in ${year}`);
  });
  return (await Promise.all(lines)).join("\n");
}
```

Mistakes like this are easy to make, especially when using `await`, and you should be aware of where the gaps in your code occur. An advantage of JavaScript's _explicit_ asynchronicity (whether through callbacks, promises, or `await`) is that spotting these gaps is relatively easy.

## Summary

Asynchronous programming makes it possible to express waiting for long-running actions without freezing the program during these actions. JavaScript environments typically implement this style of programming using callbacks, functions that are called when the actions complete. An event loop schedules such callbacks to be called when appropriate, one after the other, so that their execution does not overlap.

Programming asynchronously is made easier by promises, objects that represent actions that might complete in the future, and `async` functions, which allow you to write an asynchronous program as if it were synchronous.

## Exercises

### Tracking the scalpel

{{index "scalpel (exercise)"}}

The village crows own an old scalpel that they occasionally use on special missions—say, to cut through screen doors or packaging. To be able to quickly track it down, every time the scalpel is moved to another nest, an entry is added to the storage of both the nest that ad it and the nest that took it, under the name `"scalpel"`, with its new location as the value.

This means that finding the scalpel is a matter of following the breadcrumb trail of storage entries, until you find a nest where that points at the nest itself.

{{index "anyStorage function", "async function"}}

Write an `async` function `locateScalpel` that does this, starting at the nest on which it runs. You can use the `anyStorage` function defined earlier to access storage in arbitrary nests. The scalpel has been going around long enough that you may assume that every nest has a `"scalpel"` entry in its data storage.

Next, write the same function again without using `async` and `await`.

{{index "exception handling"}}

Do request failures properly show up as rejections of the returned promise in both versions? How?

{{if interactive

```{test: no}
async function locateScalpel(nest) {
  // Your code here.
}

function locateScalpel2(nest) {
  // Your code here.
}

locateScalpel(bigOak).then(console.log);
// → Butcher Shop
```

if}}

{{hint

{{index "scalpel (exercise)"}}

This can be done with a single loop that searches through the nests, moving forward to the next when it finds a value that doesn't match the current nest's name and returning the name when it finds a matching value. In the `async` function, a regular `for` or `while` loop can be used.

{{index recursion}}

To do the same in a plain function, you will have to build your loop using a recursive function. The easiest way to do this is to have that function return a promise by calling `then` on the promise that retrieves the storage value. Depending on whether that value matches the name of the current nest, the handler returns that value or a further promise created by calling the loop function again.

Don't forget to start the loop by calling the recursive function once from the main function.

{{index "exception handling"}}

In the `async` function, rejected promises are converted to exceptions by `await`. When an `async` function throws an exception, its promise is rejected. So that works.

If you implemented the non-`async` function as outlined earlier, the way `then` works also automatically causes a failure to end up in the returned promise. If a request fails, the handler passed to `then` isn't called, and the promise it returns is rejected with the same reason.

hint}}

### Building Promise.all

{{index "Promise class", "Promise.all function", "building Promise.all (exercise)"}}

Given an array of ((promise))s, `Promise.all` returns a promise that waits for all of the promises in the array to finish. It then succeeds, yielding an array of result values. If a promise in the array fails, the promise returned by `all` fails too, with the failure reason from the failing promise.

Implement something like this yourself as a regular function called `Promise_all`.

Remember that after a promise has succeeded or failed, it can't succeed or fail again, and further calls to the functions that resolve it are ignored. This can simplify the way you handle failure of your promise.

{{if interactive

```{test: no}
function Promise_all(promises) {
  return new Promise((resolve, reject) => {
    // Your code here.
  });
}

// Test code.
Promise_all([]).then(array => {
  console.log("This should be []:", array);
});
function soon(val) {
  return new Promise(resolve => {
    setTimeout(() => resolve(val), Math.random() * 500);
  });
}
Promise_all([soon(1), soon(2), soon(3)]).then(array => {
  console.log("This should be [1, 2, 3]:", array);
});
Promise_all([soon(1), Promise.reject("X"), soon(3)])
  .then(array => {
    console.log("We should not get here");
  })
  .catch(error => {
    if (error != "X") {
      console.log("Unexpected failure:", error);
    }
  });
```

if}}

{{hint

{{index "Promise.all function", "Promise class", "then method", "building Promise.all (exercise)"}}

The function passed to the `Promise` constructor will have to call `then` on each of the promises in the given array. When one of them succeeds, two things need to happen. The resulting value needs to be stored in the correct position of a result array, and we must check whether this was the last pending ((promise)) and finish our own promise if it was.

{{index "counter variable"}}

The latter can be done with a counter that is initialized to the length of the input array and from which we subtract 1 every time a promise succeeds. When it reaches 0, we are done. Make sure you take into account the situation where the input array is empty (and thus no promise will ever resolve).

Handling failure requires some thought but turns out to be extremely simple. Just pass the `reject` function of the wrapping promise to each of the promises in the array as a `catch` handler or as a second argument to `then` so that a failure in one of them triggers the rejection of the whole wrapper promise.

hint}}
