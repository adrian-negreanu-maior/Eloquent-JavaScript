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

One of the most pressing problems with the callback style of asynchronous programming is that it makes it extremely difficult to make sure failures are properly reported to the callbacks.

A widely used convention is that the first argument to the callback is used to indicate that the action failed, and the second contains the value produced by the action when it was successful. Such callback functions must always check whether they received an exception and make sure that any problems they cause, including exceptions thrown by functions they call, are caught and given to the right function.

{{index "rejecting (a promise)", "resolving (a promise)", "then method"}}

Promises make this easier. They can be either resolved (the action finished successfully) or rejected (it failed). Resolve handlers (as registered with `then`) are called only when the action is successful, and rejections are automatically propagated to the new promise that is returned by `then`. And when a handler throws an exception, this automatically causes the promise produced by its `then` call to be rejected. So if any element in a chain of asynchronous actions fails, the outcome of the whole chain is marked as rejected, and no success handlers are called beyond the point where it failed.

{{index "Promise.reject function", "Promise class"}}

Much like resolving a promise provides a value, rejecting one also provides one, usually called the _reason_ of the rejection. When an exception in a handler function causes the rejection, the exception value is used as the reason. Similarly, when a handler returns a promise that is rejected, that rejection flows into the next promise. There's a `Promise.reject` function that creates a new, immediately rejected promise.

{{index "catch method"}}

To explicitly handle such rejections, promises have a `catch` method that registers a handler to be called when the promise is rejected, similar to how `then` handlers handle normal resolution. It's also very much like `then` in that it returns a new promise, which resolves to the original promise's value if it resolves normally and to the result of the `catch` handler otherwise. If a `catch` handler throws an error, the new promise is also rejected.

{{index "then method"}}

As a shorthand, `then` also accepts a rejection handler as a second argument, so you can install both types of handlers in a single method call.

A function passed to the `Promise` constructor receives a second argument, alongside the resolve function, which it can use to reject the new promise.

The chains of promise values created by calls to `then` and `catch` can be seen as a pipeline through which asynchronous values or failures move. Since such chains are created by registering handlers, each link has a success handler or a rejection handler (or both) associated with it. Handlers that don't match the type of outcome (success or failure) are ignored. But those that do match are called, and their outcome determines what kind of value comes next—success when it returns a non-promise value, rejection when it throws an exception, and the outcome of a promise when it returns one of those.

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

Much like an uncaught exception is handled by the environment, JavaScript environments can detect when a promise rejection isn't handled and will report this as an error.

## Networks are hard

{{index [network, reliability]}}

Occasionally, there isn't enough light for the ((crow))s' mirror systems to transmit a signal or something is blocking the path of the signal. It is possible for a signal to be sent but never received. 

{{index "send method", error, timeout}}

As it is, that will just cause the callback given to `send` to never be called, which will probably cause the program to stop without even noticing there is a problem. It would be nice if, after a given period of not getting a response, a request would _time out_ and report failure.

Often, transmission failures are random accidents, like a car's headlight interfering with the light signals, and simply retrying the request may cause it to succeed. So while we're at it, let's make our request function automatically retry the sending of the request a few times before it gives up.

{{index "Promise class", "callback function", [interface, object]}}

And, since we've established that promises are a good thing, we'll also make our request function return a promise. In terms of what they can express, callbacks and promises are equivalent. Callback-based functions can be wrapped to expose a promise-based interface, and vice versa.

Even when a ((request)) and its ((response)) are successfully delivered, the response may indicate failure—for example, if the request tries to use a request type that hasn't been defined or the handler throws an error. To support this, `send` and `defineRequestType` follow the convention mentioned before, where the first argument passed to callbacks is the failure reason, if any, and the second is the actual result.

These can be translated to promise resolution and rejection by our wrapper.

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

Because promises can be resolved (or rejected) only once, this will work. The first time `resolve` or `reject` is called determines the outcome of the promise, and further calls caused by a request coming back after another request finished are ignored.

{{index recursion}}

To build an asynchronous ((loop)), for the retries, we need to use a recursive function—a regular loop doesn't allow us to stop and wait for an asynchronous action. The `attempt` function makes a single attempt to send a request. It also sets a timeout that, if no response has come back after 250 milliseconds, either starts the next attempt or, if this was the third attempt, rejects the promise with an instance of `Timeout` as the reason.

{{index idempotence}}

Retrying every quarter-second and giving up when no response has come in after three-quarter second is definitely somewhat arbitrary. It is even possible, if the request did come through but the handler is just taking a bit longer, for requests to be delivered multiple times. We'll write our handlers with that problem in mind—duplicate messages should be harmless.

In general, we will not be building a world-class, robust network today. But that's okay—crows don't have very high expectations yet when it comes to computing.

{{index "defineRequestType function", "requestType function"}}

To isolate ourselves from callbacks altogether, we'll go ahead and also define a wrapper for `defineRequestType` that allows the handler function to return a promise or plain value and wires that up to the callback for us.

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

`Promise.resolve` is used to convert the value returned by `handler` to a promise if it isn't already.

{{index "try keyword", "callback function"}}

Note that the call to `handler` had to be wrapped in a `try` block to make sure any exception it raises directly is given to the callback. This nicely illustrates the difficulty of properly handling errors with raw callbacks—it is easy to forget to properly route exceptions like that, and if you don't do it, failures won't get reported to the right callback. Promises make this mostly automatic and thus less error-prone.

## Collections of promises

{{index "neighbors property", "ping request"}}

Each nest computer keeps an array of other nests within transmission distance in its `neighbors` property. To check which of those are currently reachable, you could write a function that tries to send a `"ping"` request (a request that simply asks for a response) to each of them and see which ones come back.

{{index "Promise.all function"}}

When working with collections of promises running at the same time, the `Promise.all` function can be useful. It returns a promise that waits for all of the promises in the array to resolve and then resolves to an array of the values that these promises produced (in the same order as the original array). If any promise is rejected, the result of `Promise.all` is itself rejected.

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

When a neighbor isn't available, we don't want the entire combined promise to fail since then we still wouldn't know anything. So the function that is mapped over the set of neighbors to turn them into request promises attaches handlers that make successful requests produce `true` and rejected ones produce `false`.

{{index "filter method", "map method", "some method"}}

In the handler for the combined promise, `filter` is used to remove those elements from the `neighbors` array whose corresponding value is false. This makes use of the fact that `filter` passes the array index of the current element as a second argument to its filtering function (`map`, `some`, and similar higher-order array methods do the same).

## Network flooding

The fact that nests can talk only to their neighbors greatly inhibits the usefulness of this network.

For broadcasting information to the whole network, one solution is to set up a type of request that is automatically forwarded to neighbors. These neighbors then in turn forward it to their neighbors, until the whole network has received the message.

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

To avoid sending the same message around the network forever, each nest keeps an array of gossip strings that it has already seen. To define this array, we use the `everywhere` function—which runs code on every nest—to add a property to the nest's `state` object, which is where we'll keep nest-local state.

When a nest receives a duplicate gossip message, which is very likely to happen with everybody blindly resending them, it ignores it. But when it receives a new message, it excitedly tells all its neighbors except for the one who sent it the message.

This will cause a new piece of gossip to spread through the network like an ink stain in water. Even when some connections aren't currently working, if there is an alternative route to a given nest, the gossip will reach it through there.

{{index "flooding"}}

This style of network communication is called _flooding_—it floods the network with a piece of information until all nodes have it.

{{if interactive

We can call `sendGossip` to see a message flow through the village.

```
sendGossip(bigOak, "Kids with airgun in the park");
```

if}}

## Message routing

{{index efficiency}}

If a given node wants to talk to a single other node, flooding is not a very efficient approach. Especially when the network is big, that would lead to a lot of useless data transfers.

{{index "routing"}}

An alternative approach is to set up a way for messages to hop from node to node until they reach their destination. The difficulty with that is it requires knowledge about the layout of the network. To send a request in the direction of a faraway nest, it is necessary to know which neighboring nest gets it closer to its destination. Sending it in the wrong direction will not do much good.

Since each nest knows only about its direct neighbors, it doesn't have the information it needs to compute a route. We must somehow spread the information about these connections to all nests, preferably in a way that allows it to change over time, when nests are abandoned or new nests are built.

{{index flooding}}

We can use flooding again, but instead of checking whether a given message has already been received, we now check whether the new set of neighbors for a given nest matches the current set we have for it.

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

The comparison uses `JSON.stringify` because `==`, on objects or arrays, will return true only when the two are the exact same value, which is not what we need here. Comparing the JSON strings is a crude but effective way to compare their content.

The nodes immediately start broadcasting their connections, which should, unless some nests are completely unreachable, quickly give every nest a map of the current network ((graph)).

{{index pathfinding}}

A thing you can do with graphs is find routes in them, as we saw in [Chapter ?](robot). If we have a route toward a message's destination, we know which direction to send it in.

{{index "findRoute function"}}

This `findRoute` function, which greatly resembles the `findRoute` from [Chapter ?](robot#findRoute), searches for a way to reach a given node in the network. But instead of returning the whole route, it just returns the next step. That next nest will itself, using its current information about the network, decide where _it_ sends the message.

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

Now we can build a function that can send long-distance messages. If the message is addressed to a direct neighbor, it is delivered asusual. If  not, it is packaged in an object and sent to a neighbor that is closer to the target, using the `"route"` request type, which will cause that neighbor to repeat the same behavior.

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

We can now send a message to the nest in the church tower, which is four network hops removed.

```
routeRequest(bigOak, "Church Tower", "note",
             "Incoming jackdaws!");
```

if}}

{{index [network, abstraction], layering}}

We've constructed several layers of functionality on top of a primitive communication system to make it convenient to use. This is a nice (though simplified) model of how real computer networks work.

{{index error}}

A distinguishing property of computer networks is that they aren't reliable—abstractions built on top of them can help, but you can't abstract away network failure. So network programming is typically very much about anticipating and dealing with failures.

## Async functions

To store important information, ((crow))s are known to duplicate it across nests. That way, when a hawk destroys a nest, the information isn't lost.

To retrieve a given piece of information that it doesn't have in its own storage bulb, a nest computer might consult random other nests in the network until it finds one that has it.

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

Because `connections` is a `Map`, `Object.keys` doesn't work on it. It has a `keys` _method_, but that returns an iterator rather than an array. An iterator (or iterable value) can be converted to an array with the `Array.from` function.

{{index "Promise class", recursion}}

Even with promises this is some rather awkward code. Multiple asynchronous actions are chained together in non-obvious ways. We again need a recursive function (`next`) to model looping through the nests.

{{index "synchronous programming", "asynchronous programming"}}

And the thing the code actually does is completely linear—it always waits for the previous action to complete before starting the next one. In a synchronous programming model, it'd be simpler to express.

{{index "async function", "await keyword"}}

The good news is that JavaScript allows you to write pseudo-synchronous code to describe asynchronous computation. An `async` function is a function that implicitly returns a promise and that can, in its body, `await` other promises in a way that _looks_ synchronous.

{{index "findInStorage function"}}

We can rewrite `findInStorage` like this:

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

An `async` function is marked by the word `async` before the `function` keyword. Methods can also be made `async` by writing `async` before their name. When such a function or method is called, it returns a promise. As soon as the body returns something, that promise is resolved. If it throws an exception, the promise is rejected.

{{if interactive

```{startCode: true}
findInStorage(bigOak, "events on 2017-12-21")
  .then(console.log);
```

if}}

{{index "await keyword", ["control flow", asynchronous]}}

Inside an `async` function, the word `await` can be put in front of an expression to wait for a promise to resolve and only then continue the execution of the function.

Such a function no longer, like a regular JavaScript function, runs from start to completion in one go. Instead, it can be _frozen_ at any point that has an `await`, and can be resumed at a later time.

For non-trivial asynchronous code, this notation is usually more convenient than directly using promises. Even if you need to do something that doesn't fit the synchronous model, such as perform multiple actions at the same time, it is easy to combine `await` with the direct use of promises.

## Generators

{{index "async function"}}

This ability of functions to be paused and then resumed again is not exclusive to `async` functions. JavaScript also has a feature called _((generator))_ functions. These are similar, but without the promises.

When you define a function with `function*` (placing an asterisk after the word `function`), it becomes a generator. When you call a generator, it returns an ((iterator)), which we already saw in [Chapter ?](object).

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

Initially, when you call `powers`, the function is frozen at its start. Every time you call `next` on the iterator, the function runs until it hits a `yield` expression, which pauses it and causes the yielded value to become the next value produced by the iterator. When the function returns (the one in the example never does), the iterator is done.

Writing iterators is often much easier when you use generator functions. The iterator for the `Group` class (from the exercise in [Chapter ?](object#group_iterator)) can be written with this generator:

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

There's no longer a need to create an object to hold the iteration state—generators automatically save their local state every time they yield.

Such `yield` expressions may occur only directly in the generator function itself and not in an inner function you define inside of it. The state a generator saves, when yielding, is only its _local_ environment and the position where it yielded.

{{index "await keyword"}}

An `async` function is a special type of generator. It produces a promise when called, which is resolved when it returns (finishes) and rejected when it throws an exception. Whenever it yields (awaits) a promise, the result of that promise (value or thrown exception) is the result of the `await` expression.

## The event loop

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
