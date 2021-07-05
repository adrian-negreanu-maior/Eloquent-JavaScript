{{meta {load_files: ["code/chapter/18_http.js"]}}}

# HTTP ;i formulare

{{quote {author: "Roy Fielding", title: "Architectural Styles and the Design of Network-based Software Architectures", chapter: true}

Comunicația trebuie să fie "stateless" (fără stare) în natură [...] astfel încât fiecare cerere de la client către server trebuie să conțină toată informația necesară pentru a înțege cererea și nu poate profita de orice context stocat pe server.

quote}}

{{index "Fielding, Roy"}}

{{figure {url: "img/chapter_picture_18.jpg", alt: "Picture of a web form on a medieval scroll", chapter: "framed"}}}

{{index [browser, environment]}}

_Hypertext Transfer Protocol_, menționat deja în [capitolul ?](browser#web), este mecanismul prin care datele sunt solicitate și furnizate în ((World Wide Web)). Acest capitol desrie protocolul mai în detaliu și explică modul în care JavaScript în browser are acces la el.

## Protocolul

{{index "IP address"}}

Dacă tastați `_eloquentjavascript.net/18_http.html_` în bara de adrese a browserului, browserul va căuta mai întâi adresa serverului asociat cu `_eloquentjavascript.net_` și va încerca să deschidă o conexiune TCP pe portul 80, portul implicit pentru traficul HTTP. Dacă serverul există și acceptă conexiunea, browserul ar putea trimite o cerere cam așa:

```{lang: http}
GET /18_http.html HTTP/1.1
Host: eloquentjavascript.net
User-Agent: Your browser's name
```

Apoi serverul răspunde, prin aceeași conexiune.

```{lang: http}
HTTP/1.1 200 OK
Content-Length: 65585
Content-Type: text/html
Last-Modified: Mon, 08 Jan 2018 10:29:45 GMT

<!doctype html>
... the rest of the document
```

Browserul preia apoi partea din răspuns după linia goală, _corpul răspunsului_ (a nu se confunda cu tagul `<body>`) și o afișează sub forma unui document HTML.

{{index HTTP}}

Informația trimisă de către client se numește _cerere_. Ea începe cu această linie:

```{lang: http}
GET /18_http.html HTTP/1.1
```

{{index "DELETE method", "PUT method", "GET method", [method, HTTP]}}

Primul cuvânt reprezintă _metoda_ cererii. `GET` înseamnă că vrem să _obținem_ resursa specificată. Alte metode obișnuite sunt `DELETE` pentru a șterge o resursă, `PUT` pentru a crea sau înlocui și `POST` pentru a transmite informație. Menționez că serverul nu este obligat să îndeplinească fiecare cerere pe care o primește. Dacă alegeți un server la întrebare și îi cereți să își șteargă (`DELETE`) pagina principală, probabil va refuza.

{{index [path, URL], GitHub, [file, resource]}}

Partea după numele metodei este calea către _resursa_ căreia i se aplică cererea. În cel mai simplu caz, o resursă este un fișier pe server, dar protocolul nu solicită asta. O resursă poate orice care poate fi transferat _ca și cum_ ar fi un fișier. Multe servere generează răspunsul "on-the-fly". De exemplu, dacă deschideți [_https://github.com/marijnh_](https://github.com/marijnh), serverul își va consulta baza de date pentru un utilizator numit "marijnh" și, dacă găsește unul, va genera o pagină de profil pentru acel utilizator.

După calea către resursă, prima linie a cererii menționează `HTTP/1.1` pentru a indica versiunea protocolului HTTP ce este utilizată.

În practică, multe site-uri folosesc HTTP versiunea 2, care suportă aceleași concepte ca și versiunea 1.1 dar este mult mai complicat, astfel încât poate fi mai rapid. Browserele vor comuta automat la versiunea adecvată a protocolului atunci când discută cu un server dat, iar rezultatul unei cereri este același indiferent de versiunea utilizată. Deoarece versiunea 1.1 este mai directă și mai ușor de utilizat, ne vom concentra pe aceasta.

{{index "status code"}}

Răspunsul serverului va începe tot cu o versiune, urmată de statusul răspunsului, mai întâi ca un cod de trei cifre și apoi un string lizibil.

```{lang: http}
HTTP/1.1 200 OK
```

{{index "200 (HTTP status code)", "error response", "404 (HTTP status code)"}}

Codurile de stare ce încep cu 2 indică reușita cererii. Codurile care încep cu 4 înseamnă că ceva a fost greșit cu cererea. 404 este probabil cel mai faimos cod de stare HTTP - înseamnă că resursa nu a fost găsită. Codurile care încep cu 5 înseamnă că a avut loc o eroare pe server și cererea nu trebuie blamată.

{{index HTTP}}

{{id headers}}

Prima linie a cererii sau a răspunsului poate fi urmată de oricâte _headere_. Acestea sunt linii sub forma `name: value` care specifică informații suplimentare despre cerere sau răspuns. Aceste headere au fost parte din răspunsul la exemplu: 

```{lang: null}
Content-Length: 65585
Content-Type: text/html
Last-Modified: Thu, 04 Jan 2018 14:05:30 GMT
```

{{index "Content-Length header", "Content-Type header", "Last-Modified header"}}

Acestea ne comunică dimensiunea și tipul documentului de răspuns. În cazul nostru, este un document HTML de 65,585 bytes. De asemenea, ni se spune când a fost modificat documentul pentru ultima oară.

{{index "Host header", domain}}

Pentru majoritatea headere-lor, clientul și serverul au libertatea de a decide dacă să le includă sau nu în cerere sau în răspuns. Dar câteva sunt obligatorii. De exemplu, header-ul `Host`, care specifică numele gazdei, trebuie inclus în cerere, deoarece serverul ar putea deservi gazde cu mai multe nume pe o singură adresă IP, și fără acest header serverul nu poate știi cu care dintre gazde încearcă să discute clientul.

{{index "GET method", "DELETE method", "PUT method", "POST method", "body (HTTP)"}}

După headere, atât cererea cât și răspunsul pot include o linie goalp, urmată de un corp, care conține datele care se vor transmite. Cererile `GET` și `DELETE` nu trimit nici un fel de date, dar `PUT` și `POST` vor trimite. Similar, unele tipuri de răspunsuri, cum ar fi cele de eroare, nu necesită un corp.

## Browserele și HTTP

{{index HTTP, [file, resource]}}

După cum am văzut în exemplu, un browser va efectua o cerere când introducem un URL în bara sa de adrese. Când pagina HTML rezultată va referi și alte fișiere, cum ar fi imagini sau fișiere JavaScript, acestea vor fi la rândul lor returnate.

{{index parallelism, "GET method"}}

Un website de complexitate moderată poate include ușor între 10 și 200 resurse. Pentru a le putea aduce rapid, browserele vor face mai multe cereri `GET` simultan, în loc să aștepte secvențial răspunsul la fiecare cerere.

Paginile HTML pot include _formulare_ care permit utilizatorului să completeze informația și să o trimită la server. Acesta este un exemplu de formular:

```{lang: "text/html"}
<form method="GET" action="example/message.html">
  <p>Name: <input type="text" name="name"></p>
  <p>Message:<br><textarea name="message"></textarea></p>
  <p><button type="submit">Send</button></p>
</form>
```

{{index form, "method attribute", "GET method"}}

Codul de mai sus descrie un formular cu două câmpuri: unul mai mic pentru nume și unul mai mare pentru a scrie un mesaj în el. Când apăsați butonul "Send", formularul este _trimis_, ceea ce înseamnă că va fi împachetat conținutul câmpurilor sale într-o cerere HTTP și browserul va naviga la rezultatul acelei cereri.

Când atributul `method` al elementului `<form>` are valoarea `GET` (sau este omis), informația din formular este adăugată la sfârșitul URL-ului `action` ca și un _query string_. Browserul ar putea face o cerere la acest URL:

```{lang: null}
GET /example/message.html?name=Jean&message=Yes%3F HTTP/1.1
```

{{index "ampersand character"}}

Semnul de întrebare marchează sfârșitul părții despre calea URL-ului și începutul interogării. Apoi urmează o succesiune de nume și valori, corespunzătoare atributelor `name` de pe câmpurile formularului și conținutului acelor elemente. Un caracter ampersand (`&`) este utilizat pentru a separa perechile.

{{index [escaping, "in URLs"], "hexadecimal number", "encodeURIComponent function", "decodeURIComponent function"}}

Mesajul codificat în URL este "Yes?", dar semnul de întrebare este înlocuit de un cod straniu. Unele caractere din query string trebuie să fie înlocuite. Semnul de întrebare, reprezentat ca `%3F`, este unul dintre ele. Pare să fie o regulă nescrisă că fiecare format necesită propriul său mod de "escaping". În acest caz, 3F, care reprezintă 63 în notație zecimală este codul caracterului  `?`. JavaScript vă pune la dispoziție funcțiile `encodeURIComponent` și `decodeURIComponent` pentru a codifica și decodifica acest format.

```
console.log(encodeURIComponent("Yes?"));
// → Yes%3F
console.log(decodeURIComponent("Yes%3F"));
// → Yes?
```

{{index "body (HTTP)", "POST method"}}

Dacă schimbăm atributul `method` al formularului din exemplul anterior la `POST`, cererea HTTP pentru trimiterea formularului va utiliza metoda `POST` și va plasa stringul de cerere în corpul cererii, în loc să îl adauge la URL.

```{lang: http}
POST /example/message.html HTTP/1.1
Content-length: 24
Content-type: application/x-www-form-urlencoded

name=Jean&message=Yes%3F
```

Cererile `GET` trebuie utilizate pentru cereri care nu au efecte secundare ci doar solicită informație. Cererile care modifică o resursă pe server, cum ar fi crearea unui nou cont sau postarea unui mesaj, trebuie exprimate cu alte metode, cum ar fi `POST`. Software-ul client cum ar fi un browser știe că nu trebuie să facă orbește cereri `POST` dar adesea va efectua implicit cereri `GET` - de exemplu, preîncărcarea unei resurse despre care crede că utilizatorul va avea nevoie în curând.

Vom reveni asupra formularelor și a modului în care putem interacționa cu ele în JavaScript [mai târziu în acest capitol](http#forms).

{{id fetch}}

## `fetch`

{{index "fetch function", "Promise class", [interface, module]}}

Interfața prin care JavaScript în browser poate efectua cereri HTTP se numește `fetch`. Deoarece este relativ nouă, ea utilizează în mod convenabil promisiunile (ceea ce este rar pentru interfețele browserului).

```{test: no}
fetch("example/data.txt").then(response => {
  console.log(response.status);
  // → 200
  console.log(response.headers.get("Content-Type"));
  // → text/plain
});
```

{{index "Response class", "status property", "headers property"}}

Apelarea `fetch` returnează o promisiune care se rezolvă la un obiect `Response` ce reține informația despre răspunsul serverului, cum ar fi codul de stare și header-ele. Header-ele sunt împachetate într-un obiect asemănător unui `Map` care tratează cheile (numele headerelor) fără diferența între litere mari și mici deoarece se presupune că această diferență nu trebuie să se facă pentru numele headere-lor. Aceasta înseamnă că `headers.get("Content-Type")` și `headers.get("content-TYPE")` vor returna aceeași valoare.

De menționat că promisiunea returnată de `fetch` se rezolvă cu succes chiar dacă serverul răspunde cu un cod de eroare. Dar poate fi _respinsă_ dacă există o eroare de rețea sau serverul căruia îi este adresată cererea nu poate fi găsit.

{{index [path, URL], "relative URL"}}

Primul argument al `fetch` este URL-ul ce trebuie solicitat. Când acel URL nu începe cu numele unui protocol (cum ar fi _http:_), cererea este tratată ca fiind _relativă_, adică este interpretată relativ la documentul curent. Când începe cu `/` este înlocuită calea curentă, care este partea de după numele serverului. Dacă nu , calea curentă, până la ultimul caracter `/` este plasată în fața URL-ului relativ.

{{index "text method", "body (HTTP)", "Promise class"}}

Pentru a obține conținutul propri-zis al unui răspuns, puteți utiliza metoda `text`. Deoarece promisiunea inițială este rezolvată imediat ce headerele de răspuns au fost primite și deoarece citirea corpului răspunsului ar putea necesita mai mult timp, din nou este returnată o promisiune.

```{test: no}
fetch("example/data.txt")
  .then(resp => resp.text())
  .then(text => console.log(text));
// → This is the content of data.txt
```

{{index "json method"}}

O metodă similară, numită `json`, returnează o promisiune care see rezolvă la valoarea pe care o obțineți parsând corpul ca JSON sau este respinsă dacă nu avem de a face cu JSON valid.

{{index "GET method", "body (HTTP)", "DELETE method", "method property"}}

Implicit, `fetch` utilizează metoda `GET` pentru cerere și nu include un corp al cererii. O puteți configura diferit prin transmiterea unui obiect cu opțiuni suplimentare ca al doilea argument. De exemplu, cererea de mai jos încearcă să șteargă `example/data.txt`:

```{test: no}
fetch("example/data.txt", {method: "DELETE"}).then(resp => {
  console.log(resp.status);
  // → 405
});
```

{{index "405 (HTTP status code)"}}

Codul de stare 405 înseamnă "method not allowed", modul HTTP de a răspunde "Nu pot face asta".

{{index "Range header", "body property", "headers property"}}

Pentru a adăuga un corp al cererii, puteți include o opțiune `body`. Pentru a seta headerele aveți opțiunea `headers`. De exemplu, cererea de mai jos include un header `Range`, care instruiește serverul să returneze doar o parte din răspuns.

```{test: no}
fetch("example/data.txt", {headers: {Range: "bytes=8-19"}})
  .then(resp => resp.text())
  .then(console.log);
// → the content
```

Browserul va adăuga automat câteva headere, cum ar fi "Host" și cele necesare pentru server ca să își dea seama de dimensiunea corpului. Dar adăugarea propriilor headere este adesea utilă pentru a include lucruri cum ar informația de autentificare sau pentru a informa serverul ce format de fișier am dori să primim.

{{id http_sandbox}}

## HTTP sandboxing

{{index sandbox, [browser, security]}}

Efectuarea cererilor HTTP în scripturile din paginile web ridică din nou preocupări legate de securitate. Persoana care controlează scriptul ar putea să nu aibă aceleași interese ca și persoana pe al cărei computer rulează scriptul. Mai specific, dacă vizitez _themafia.org_, nu vreau ca scripturile sale să poată face cereri către _mybank.com_, utilizând informație de identificare din browserul meu, cu instrucțiuni de transfer al banilor mei spre un cont la întâmplare. 

Din acest motiv, browserele ne protejează prin interzicerea scripturilor ca să efectueze cereri HTTP spre alte domenii.

{{index "Access-Control-Allow-Origin header", "cross-domain request"}}

Aceasta poate fi o problemă deranjantă când construim sisteme care trebuie să acceseze câteva domenii din motive legitime. Din fericire, serverele pot include un header în răspuns pentru a indica explicit că browserul este în regulă cu o cerere care provine dintr-un alt domeniu:

```{lang: null}
Access-Control-Allow-Origin: *
```

## Aprecierea HTTP

{{index client, HTTP, [interface, HTTP]}}

Când construim un sistem care necesită comunicare între un program JavaScript ce rulează în browser (client) și un program de pe server, există mai multe moduri de a modela această comunicare.

{{index [network, abstraction], abstraction}}

Un model utilizat frecvent este _remote procedure call_. În acest model, comunicarea urmează șabloanele unui apel normal de funcție, cu excepția faptului că funncția este executată pe o altă mașină. Apelul presupune realizarea unei cereri către server care include numele funcției și argumentele. Răspunsul la acea cerere conține valoarea returnată.

Când gândim în termeni de apeluri procedurale la distanță, HTTP este doar vehiculul pentru comunicare și probabil veți scrie un strat de abstractizare care îl ascunde complet.

{{index "media type", "document format", [method, HTTP]}}

O altă abordare ar fi să construiți comunicația în jurul conceptului de _resurse_ și metode HTTP. În locul unui apel la distanță către procedura `addUser`, ați putea utiliza o cerere `PUT` către `/users/larry`. În loc să codificați proprietățile utilizatorului ca argumente ale funcției, definiți un document în format JSON (sau utilizați un format existent) ce reprezintă utilizatorul. Corpul cererii `PUT` pentru a crea o nouă resursă este apoi tocmai un astfel de document. O resursă este adusă printr-o cerere `GET` către URL-ul resursei (de exemplu, `/user/larry`), care va returna din nou un document ce reprezintă resursa.

Această abordare ușurează utilizarea caracteristicilor care sunt puse la dispoziție de HTTP, cum ar fi suport pentru stocarea în cache a resurselor (păstrarea unei copii la client pentru acces rapid). Conceptele folosite în HTTP, care sutn bine concepute, oferă un set util de principii pentru a defini interfața serverului.

## Securitatea și HTTPS

{{index "man-in-the-middle", security, HTTPS, [network, security]}}

Datele ce călătoresc prin Internet tind să urmeze un drum lung și periculos. Pentru a ajunge la destinație, trebuie să treacă prin orice, de la hotspoturi WiFi din cafenele până la rețele controlate de diferite companii și state. În orice punct de-a lungul rutei, ele pot fi inspectate și chiar modificate.

{{index tampering}}

Dacă este important secretul informațiilor, cum ar fi parole sau conturi de email, sau ca datele să ajungă la destinație nemodificate, cum ar fi numărul de cont în care transferați banii prin site-ul web al unei bănci, HTTP nu este suficient de bun.

{{index cryptography, encryption}}

{{indexsee "Secure HTTP", HTTPS, [browser, security]}}

Protocolul HTTP securizat, utilizat pentru URL-uri care încep cu _https://_, împachetează traficul HTTP într-un mod care îngreunează citirea și modificarea datelor. Înainte de a schimba datele, clientul verifică identitatea serverului prin a îi cere să demonstreze că deține un certificat criptografic eliberat de către o autoritate certificată pe care browserul o recunoaște. Apoi, toate datele ce parcurg conexiunea vor fi criptate într-un mod care ar trebui să prevină citirea sau modificarea neautorizată.

Astfel, când funcționează corect, HTTPS interzice altor persoane să impersoneze site-ul web cu care încercăm să comunicăm și să spioneze comunicația noastră. Nu este perfect, și au existat incidente în care HTTPS a eșuat din cauza unor certificate furate sau "fabricate" și a unui software defect, dar este _mult_ mai sigur decât HTTP.

{{id forms}}

## Câmpuri în formulare

Formularele au fost original concepute înaintea Web-ului cu JavaScript pentru a permite web siteurilor să trimită informație de la utilizator într-o cerere HTTP. Conceptul presupune că interacțiunea cu serverul are loc întotdeauna prin navigarea la o pagină nouă.

{{index [DOM, fields]}}

Dar elementele formularelor sunt parte din DOM, ca și restul paginii, iar elementele DOM care reprezintă câmpuri din formulare suportă câteva proprietăți și evenimente care nu sunt prezente pentru alte elemente. Acestea fac posibilă inspectarea și controlul unor asemenea câmpuri de intrare cu programe JavaScript și operații cum ar fi adăugarea de funcționalitate nouă la un formular sau utilizarea formularelor și a câmpurilor ca și blocuri de construcție a aplicațiilor JavaScript.

{{index "form (HTML tag)"}}

Un formular web constă din mai multe câmpuri de intrare, grupate într-un tag `<form>`. HTML permite mai multe tipuri de câmpuri, de la simple cutii de bifare on/off până la meniuri drop-down și câmpuri pentru introducere textelor. În această carte nu vom încerca să discutăm extensiv despre toate tipurile de câmpuri, doar vom face o introducere.

{{index "input (HTML tag)", "type attribute"}}

Multe tipuri de câmpuri folosesc tag-ul `<input>`. Atributul `type` al acestui tag este utilizat pentru a selecta stilul câmpului. Iată câteva tipuri de `<input>` utilizate frecvent: 

{{index "password field", checkbox, "radio button", "file field"}}

{{table {cols: [1,5]}}}

| `text`     | Un câmp de text pe un singură linie
| `password` | Identic cu `text` dar ascunde textul care a fost introdus
| `checkbox` | Un comutator on/off
| `radio`    | Parte a unui câmp cu alegeri multiple
| `file`     | Permite utilizatorului să aleagă un fișier din computerul său

{{index "value attribute", "checked attribute", "form (HTML tag)"}}

Câmpurile nu trebuie să apară obligatoriu într-un tag `<form>`. Le puteți plasa oriunde în pagină. Asemenea câmpuri fără formular nu pot fi trimise (doar formularele pot fi trimise), dar atunci când răspundem la datele de intrare cu JavaScript, de cele mai multe ori nu vrem să trimitem datele din câmpuri în mod obișnuit.

```{lang: "text/html"}
<p><input type="text" value="abc"> (text)</p>
<p><input type="password" value="abc"> (password)</p>
<p><input type="checkbox" checked> (checkbox)</p>
<p><input type="radio" value="A" name="choice">
   <input type="radio" value="B" name="choice" checked>
   <input type="radio" value="C" name="choice"> (radio)</p>
<p><input type="file"> (file)</p>
```

{{if book

Câmpurile create de codul HTML de mai sus arată cam așa:

{{figure {url: "img/form_fields.png", alt: "Various types of input tags",width: "4cm"}}}

if}}

Interfața JavaScript pentru asemenea elemente depinde de tipul de element.

{{index "textarea (HTML tag)", "text field"}}

Câmpurile text pe mai multe linii au propriul lor tag, `<textarea>`, în principal din cauză că utilizarea unui atribut pentru a preciza o valoare pe mai multe linii ar fi ciudată. Tagul `<textarea>` are nevoie de perechea `</textarea>` pentru închidere și utilizează textul din interiorul perechii de marcaje în locul atributului `value`, ca și text inițial.

```{lang: "text/html"}
<textarea>
one
two
three
</textarea>
```

{{index "select (HTML tag)", "option (HTML tag)", "multiple choice", "drop-down menu"}}

În final, tagul `<select>` este utilizat pentru a crea un câmp care permite utilizatorului să selecteze dintre mai multe opțiuni predefinite.

```{lang: "text/html"}
<select>
  <option>Pancakes</option>
  <option>Pudding</option>
  <option>Ice cream</option>
</select>
```

{{if book

Un asemenea câmp arată astfel:

{{figure {url: "img/form_select.png", alt: "A select field", width: "4cm"}}}

if}}

{{index "change event"}}

La fiecare modificare a valorii unui câmp, va fi declanșat un eveniment `"change"`.

## Focus

{{index keyboard, focus}}

{{indexsee "keyboard focus", focus}}

Spre deosebire de majoritatea elementelor din documentele HTML, pcâmpurile formularelor pot recepționa _focus de la tastatură_. Când executăm click sau când le activăm într-un alt mod, ele devin elementul activ curent și cel care va recepționa intrarea de la tastatură.

{{index "option (HTML tag)", "select (HTML tag)"}}

Prin urmare, puteți tasta într-un câmp text doar când el deține focusul. De exemplu, un meniu `<select>` încearcă să se mute la opțiunea care conține textul pe care utilizatorul la tastat și răspunde la tastele săgeți prin mutarea sus-jos a selecției.

{{index "focus method", "blur method", "activeElement property"}}

Putem controla focusul din JavaScript, prin metodele `focus` și `blur`. Prima mută focusul pe elementul DOM asupra căruia a fost apelată iar cea de a doua îndepărtează focusul. Valoarea din `document.activeElement` corespunde elementului curent ce deține focusul.

```{lang: "text/html"}
<input type="text">
<script>
  document.querySelector("input").focus();
  console.log(document.activeElement.tagName);
  // → INPUT
  document.querySelector("input").blur();
  console.log(document.activeElement.tagName);
  // → BODY
</script>
```

{{index "autofocus attribute"}}

Pentru unele pagini, se așteaptă ca utilizatorul să dorească să interacționeze cu câmpul din formular imediat. JavaScript poate fi utilizat  pentru a focaliza acest câmp când documentul este încărcat, dar HTML definește atributul `autofocus`, care produce același efect în timp ce browserul cunoaște ce încercăm să realizăm. Aceasta dă browserului opțiunea de a dezactiva comportamentul dacă nu este adecvat, cum ar fi atunci când utilizatorul a mutat focusul pe un alt element.

{{index "tab key", keyboard, "tabindex attribute", "a (HTML tag)"}}

Tradițional, browserele permit utilizatorului să mute focusul prin document prin apăsarea tastei [tab]{keyname}. Putem 
influența ordinea în care elementele recepționează focusul prin utilizarea atributului `tabindex`. Exemplul următor ne permite să sărim la butonul OK fără a trece mai întâi prin linkul de ajutor:

```{lang: "text/html", focus: true}
<input type="text" tabindex=1> <a href=".">(help)</a>
<button onclick="console.log('ok')" tabindex=2>OK</button>
```

{{index "tabindex attribute"}}

Implicit, cele mai multe tipuri de elemente HTML nu pot recepționa focusul. Dar puteți adăuga un atribut `tabindex` oricărui element, astfel încât acesta va putea primi focusul. Un `tabindex` de -1 va forța saltul peste un element, chiar dacă în mod normal acesta poate recepționa focusul (la apăsarea tastei tab).

## Câmpuri dezactivate

{{index "disabled attribute"}}

Orice câmp al unui formular poate fi dezactivat prin atributul `disabled`. Acesta este un atribut care poate fi specificat fără valoare - doar simpla sa prezență dezactivează elementul.

```{lang: "text/html"}
<button>I'm all right</button>
<button disabled>I'm out</button>
```

Câmpurile dezactivate nu pot primi focusul și nici nu pot fi modificate, iar browserele le afișează gri și estompate.

{{if book

{{figure {url: "img/button_disabled.png", alt: "A disabled button",width: "3cm"}}}

if}}

{{index "user experience"}}

Când programul este în procesul de gestionare a unei acțiuni cauzate de vreun buton sau alt control care ar putea necesita comunicația cu serverul și deci, să dureze, ar putea fi o bună idee să dezactivăm control până când acțiunea se termină. Astfel, când utilizatorul devine nerăbdător și mai execută un click, el nu repetă accidental acțiunea.

## Formularul ca întreg

{{index "array-like object", "form (HTML tag)", "form property", "elements property"}}

Când un câmp este conținut în interiorul unui element `<form>`, elementul său DOM va avea o proprietate `form` ce îl leagă de elementul DOM al formularului. Elementul `<form>`, pe de altă parte, are o proprietate numită `elements` care conține o colecție de câmpuri aflate în interiorul său, asemănătoare unui array.

{{index "elements property", "name attribute"}}

Atributul `name` al unui câmp determină modul în care acesta este identificat atunci când formularul este trimis. El poate fi utilizat și ca nume de proprietate atunci când se accesează proprietatea `elements` a formularului, care funcționează atât ca un obiect similar unui array (accesibil prin index numeric) cât și ca un map (accesibil prin nume).

```{lang: "text/html"}
<form action="example/submit.html">
  Name: <input type="text" name="name"><br>
  Password: <input type="password" name="password"><br>
  <button type="submit">Log in</button>
</form>
<script>
  let form = document.querySelector("form");
  console.log(form.elements[1].type);
  // → password
  console.log(form.elements.password.type);
  // → password
  console.log(form.elements.name.form == form);
  // → true
</script>
```

{{index "button (HTML tag)", "type attribute", submit, "enter key"}}

Atunci când se va apăsa un buton al cărui atribut `type` are valoarea `submit`, formularul va fi trimis. Apăsarea [enter]{keyname} când un câmp al formularului are focusul va avea același efect.

{{index "submit event", "event handling", "preventDefault method", "page reload", "GET method", "POST method"}}

Trimiterea unui formular înseamnă de regulă că browserul va naviga la pagina indicată de atributul `action` al formularului, utilizând o cerere `GET` sau `POST`. Dar, înainte de aceasta, este declanșat un eveniment `"submit"`. Putem intercepta acest eveniment în JavaScript și să prevenim acest comportament implicit prin apelarea `preventDefault` asupra obiectului evenimentului.

```{lang: "text/html"}
<form action="example/submit.html">
  Value: <input type="text" name="value">
  <button type="submit">Save</button>
</form>
<script>
  let form = document.querySelector("form");
  form.addEventListener("submit", event => {
    console.log("Saving value", form.elements.value.value);
    event.preventDefault();
  });
</script>
```

{{index "submit event", validation}}

Interceptarea evenimentelor `"submit"` în JavaScript are mai multe utilități. Putem scrie cod care verifică dacă valorile introduse de utilizator au sens și să afișăm imediat un mesaj de eroare, în loc să trimitem formularul. Sau putem dezactiva modul regulat de transmitere, ca și în exemplu, și programul nostru să gestioneze intrarea, și de exemplu să utilizăm `fetch` pentru a transmite datele la server fără a reîncărca pagina.

## Câmpuri text

{{index "value attribute", "input (HTML tag)", "text field", "textarea (HTML tag)", [DOM, fields], [interface, object]}}

Câmpurile create de tagurile `<textarea>` sau tagurile `<input>` cu tipul `text` sau `password`, împart o interfață comună. Elementele lor DOM au o proprietate `value` care reține conținutul lor curent sub forma unei valori de tip string. Setarea acestei proprietăți la o altă valoare de tip string schimbă conținutul câmpului.

{{index "selectionStart property", "selectionEnd property"}}

Proprietățile `selectionStart` și `selectionEnd` ale câmpurilor de tip text ne dau informații despre cursorul și selecția din text. Când nu este selectat nimic, aceste două proprietăți rețin același număr, ce indică poziția cursorului. De exemplu, 0 indică începutul textului iar 10 indică poziționarea cursorului după al 10-lea caracter. Când o parte din text este selectată, cele două proprietăți diferă și rețin începutul și sfârșitul porțiunii de text selectate. Ca și în cazul `value`, în aceste proprietăți putem scrie.

{{index Khasekhemwy, "textarea (HTML tag)", keyboard, "event handling"}}

Imaginați-vă ca scrieți un articol despre Khasekhemwy dar aveți probleme în a-i scrie corect numele. Codul care urmează leagă un tag `<textarea>` cu un handler de eveniment astfel încât, de fiecare dată când apăsați F2, va fi inserat stringul "Khasekhemwy".

```{lang: "text/html"}
<textarea></textarea>
<script>
  let textarea = document.querySelector("textarea");
  textarea.addEventListener("keydown", event => {
    // The key code for F2 happens to be 113
    if (event.keyCode == 113) {
      replaceSelection(textarea, "Khasekhemwy");
      event.preventDefault();
    }
  });
  function replaceSelection(field, word) {
    let from = field.selectionStart, to = field.selectionEnd;
    field.value = field.value.slice(0, from) + word +
                  field.value.slice(to);
    // Put the cursor after the word
    field.selectionStart = from + word.length;
    field.selectionEnd = from + word.length;
  }
</script>
```

{{index "replaceSelection function", "text field"}}

Funcția `replaceSelection` va înlocui partea selectată a conținutului din câmpul text cu cuvîntul dat și apoi va muta cursorul după acel cuvânt astfel că utilizatorul va putea continua să scrie.

{{index "change event", "input event"}}

Evenimentul `"change"` pentru un câmd de tip text nu se declanșează de fiecare dată când se tastează ceva. Acest eveniment este declanșat atunci când câmpul își pierde focusul după ce conținutul său a fost modificat. Pentru a răspunde imediat la modificările dintr-un câmp text ar trebui să înregistrați un handler pentru evenimentul `"input"`, care se declanșează pentru fieecare caracter introdus de către utilizator, sau când utilizatorul șterge text sau manipulează în orice alt mod conținutul câmpului.

Exemplul de mai jos vă prezintă un câmp de tip text și un contor pentru lungimea curentă a textului din acel câmp:

```{lang: "text/html"}
<input type="text"> length: <span id="length">0</span>
<script>
  let text = document.querySelector("input");
  let output = document.querySelector("#length");
  text.addEventListener("input", () => {
    output.textContent = text.value.length;
  });
</script>
```

## Câmpuri de bifare și butoane radio

{{index "input (HTML tag)", "checked attribute"}}

Un câmp de bifare este un comutator binar. Valoarea lui poate fi obținută sau modificată din proprietatea `checked`, care reține o valoare booleană.

```{lang: "text/html"}
<label>
  <input type="checkbox" id="purple"> Make this page purple
</label>
<script>
  let checkbox = document.querySelector("#purple");
  checkbox.addEventListener("change", () => {
    document.body.style.background =
      checkbox.checked ? "mediumpurple" : "";
  });
</script>
```

{{index "for attribute", "id attribute", focus, "label (HTML tag)", labeling}}

Tagul `<label>` asociază o porțiune din document cu un câmp de introducere a datelor. Executând click oriunde pe etichetă se va activa câmpul, care primește focusul și va fi schimbată valoarea sa când un câmp de bifare sau un buton radio.

{{index "input (HTML tag)", "multiple-choice"}}

Un buton radio este similar cu un câmp de bifă, dar este implicit legat la alte butoane radio ce au același atribut `name` astfel încât doar unul dintre ele este activ în orice moment.

```{lang: "text/html"}
Color:
<label>
  <input type="radio" name="color" value="orange"> Orange
</label>
<label>
  <input type="radio" name="color" value="lightgreen"> Green
</label>
<label>
  <input type="radio" name="color" value="lightblue"> Blue
</label>
<script>
  let buttons = document.querySelectorAll("[name=color]");
  for (let button of Array.from(buttons)) {
    button.addEventListener("change", () => {
      document.body.style.background = button.value;
    });
  }
</script>
```

{{index "name attribute", "querySelectorAll method"}}

Parantezele pătrate din interogarea CSS transmisă către `querySelectorAll` sunt utilizate pentru a potrivi atribute. Selectează elementele al căror atribute `name` are valoarea `"color"`.

## Câmpuri de selectare

{{index "select (HTML tag)", "multiple-choice", "option (HTML tag)"}}

Câmpurile de selectare sunt conceptual similare cu butoanele radio - și ele permit utilizatorului să aleagă dintr-un set de opțiuni. Dar în timp ce butoanele radio ne permit controlul aspectului opțiunilor, aspectul unui tag `<select>` este determinat de către browser.

{{index "multiple attribute", "drop-down menu"}}

Câmpurile de seletare au și o variantă care seamănă mai mult cu o listă de câmpuri de bifare. Când îi setăm atributul `multiple`, un tag `<select>` va permite utilizatorului să selecteze oricâte opțiuni, nu doar una. În majoritatea browserelor, asemenea câmpuri sunt afișate diferit față de un selector normal, care de regulă este afișat ca un control  _drop-down_ care afișează opțiunile doar când îl deschideți.

{{index "option (HTML tag)", "value attribute"}}

Fiecare tag `<option>` are o valoare. Această valoare poate fi definită cu atributul `value`. Atunci când acesta nu este setat, textul din interiorul opțiunii va fi considerat ca valoarea asociată. Proprietatea `value` a unui element `<select>` reflectă opțiunea selectată curent. Totuși, pentru un câmp `multiple` această proprietate nu are prea multă semnificație pentru că va conține doar valoarea _uneia_ dintre opțiunile selectate.

{{index "select (HTML tag)", "options property", "selected attribute"}}

Tagurile `<option>` ale unui câmp select `<select>` pot fi accesate ca un obiect de tip array, prin proprietatea `options` a câmpului. Fiecare opțiune are o proprietate numită `selected`, care indicăă dacă opțiunea este selectată sau nu. Proprietatea poate fi și setată pentru a selecta sau deselecta o opțiune.

{{index "multiple attribute", "binary number"}}

Exemplul de mai jos extrage valorile selectate dintr-un câmp de selectare `multiple` și le utilizează pentru a compune un număr binar din biții individuali. Mențineți apăsată tasta [control]{keyname} (sau [command]{keyname} pe Mac) pentru a selecta mai multe opțiuni.

```{lang: "text/html"}
<select multiple>
  <option value="1">0001</option>
  <option value="2">0010</option>
  <option value="4">0100</option>
  <option value="8">1000</option>
</select> = <span id="output">0</span>
<script>
  let select = document.querySelector("select");
  let output = document.querySelector("#output");
  select.addEventListener("change", () => {
    let number = 0;
    for (let option of Array.from(select.options)) {
      if (option.selected) {
        number += Number(option.value);
      }
    }
    output.textContent = number;
  });
</script>
```

## Câmpuri pentru fișiere

{{index file, "hard drive", "file system", security, "file field", "input (HTML tag)"}}

Câmpurile pentru fișiere au fost inițial concepute ca o modalitate pentru încărcarea fișierelor din computerul utilizatorului prin intermediul unui formular. În browserele moderne, ele ne dau și posibilitatea de a citi asemenea fișiere în programele JavaScript. Câmpul acționează și ca un paznic. Scriptul nu poate pur și simplu să citească fișiere private din computerul utilizatorului, dar dacă utilizatorul selectează un fișier pentru un asemenea câmp, browserul interpretează acțiunea ca o permisiune pentru a citi acel fișier.

Un câmp pentru fișiere arată de obicei ca un buton etichetat cu "choose file" sau "browse", și informația despre fișierul ales afișată lângă buton.

```{lang: "text/html"}
<input type="file">
<script>
  let input = document.querySelector("input");
  input.addEventListener("change", () => {
    if (input.files.length > 0) {
      let file = input.files[0];
      console.log("You chose", file.name);
      if (file.type) console.log("It has type", file.type);
    }
  });
</script>
```

{{index "multiple attribute", "files property"}}

Proprietatea `files` a unui câmp pentru fișiere este un obiect asemănător unui array (din nou, nu un array real) ce conține toate fișierele alese în câmp. Aceasta este inițial goală. Motivul pentru care nu există doar o proprietate `file` este legat de faptul că acest tip de câmp permite și atributul `multiple`, care face posibilă încărcarea mai multor fișiere în același timp.

{{index "File type"}}

Obiectele componente ale obiectului `files` au proprietăți cum ar fi `name` (numele fișierului), `size` (dimensiunea fișierului în bytes - bucăți de câte 8 biți) și `type` (tipul media al fișierului, cum ar fi `text/plain` sau `image/jpeg`).

{{index ["asynchronous programming", "reading files"], "file reading", "FileReader class"}}

{{id filereader}}

Ceea ce lipsește este o proprietate care să rețină conținutul fișierului. Obținerea acestui conținut este puțin mai complicată. Deoarece citirea unui fișier de pe disc poate cere timp, interfața este asincornă pentru a evita înghețarea documentului.

```{lang: "text/html"}
<input type="file" multiple>
<script>
  let input = document.querySelector("input");
  input.addEventListener("change", () => {
    for (let file of Array.from(input.files)) {
      let reader = new FileReader();
      reader.addEventListener("load", () => {
        console.log("File", file.name, "starts with",
                    reader.result.slice(0, 20));
      });
      reader.readAsText(file);
    }
  });
</script>
```

{{index "FileReader class", "load event", "readAsText method", "result property"}}

Citirea unui fișier se realizează prin crearea unui obiect `FileReader`, înregistrarea unui handler pentru evenimentul `"load"` și apelarea metodei `readAsText`, care primește ca și argument fișierul pe care vrem să îl citim. Odată încheiată încărcarea, proprietatea `result` a reader-ului stochează conținutul fișierului.

{{index "error event", "FileReader class", "Promise class"}}

`FileReader` declanșează și un eveniment `"error"` dacă citirea fișierului eșuează din orice motiv. Obiectul asociat erorii va fi asociat cu proprietatea `error` a reader-ului. Această interfață a fost proiectată înainte ca promisiunile să devină parte din limbaj. Puteți împacheta codul într-o promisiune astfel:

```
function readFileText(file) {
  return new Promise((resolve, reject) => {
    let reader = new FileReader();
    reader.addEventListener(
      "load", () => resolve(reader.result));
    reader.addEventListener(
      "error", () => reject(reader.error));
    reader.readAsText(file);
  });
}
```

## Stocarea datelor în partea clientului

{{index "web application"}}

Paginile HTML simple, cu puțin JavaScript sunt un format extraordinar pentru "mini-aplicații" - mici programe ajutătoare care automatizează sarcini de bază. Prin conectarea mai multor câmpuri de formular cu handlere de evenimente, puteți realiza orice, de la o conversie între centimetri și inchi până la calcularea unei parole pornind de la o parolă principală și numele unui website.

{{index persistence, [binding, "as state"], [browser, storage]}}

Când asemenea aplicații trebuie să își amintească ceva între sesiuni, nu puteți utiliza doar bindinguri JavaScript - acestea sunt volatile și vor fi distruse de fiecare dată când se închide pagina. Ați putea seta un server, să îl conectați la Internet și aplicația voastră să stocheze date acolo. Vom vedea cum putem face asta în [capitolul ?](node). Dar aceasta este multă muncă suplimentară și complexitate adăugată. Uneori este suficient să păstrați datele în browser.

{{index "localStorage object", "setItem method", "getItem method", "removeItem method"}}

Obiectul `localStorage` poate fi folosit pentru a stoca datele într-un mod care supraviețuiește reîncărcărilor paginii. Acest obiect vă permite să stocați valori de tip string și să le asociați nume.

```
localStorage.setItem("username", "marijn");
console.log(localStorage.getItem("username"));
// → marijn
localStorage.removeItem("username");
```

{{index "localStorage object"}}

O valoare din `localStorage` este persistentă până când este suprascrisă, este eliminată cu `removeItem` sau prin acțiunea utilizatorului de a curăța datele locale.

{{index security}}

Siteurile cu domenii diferite primesc compoartimente diferite de stocare. Ceea ce înseamnă că datele stocate în `localStorage` de către un anume website pot, în principiu, să fie citite și suprascrise doar de către scripturile de pe acel site.

{{index "localStorage object"}}

Browserele impun o limită cu privire la dimensiunea datelor care se pot stoca în `localStorage` de către un site. Această restricție, împreună cu faptul că umplerea hard diskului altora cu "gunoi" nu este profitabilă, previne această funcționalitate ca să consume prea mult spațiu.

{{index "localStorage object", "note-taking example", "select (HTML tag)", "button (HTML tag)", "textarea (HTML tag)"}}

Codul de mai jos implementează o aplicație brută de luare a notițelor. Ea va menține un set de note și va permite utilizatorului să editeze notele sau să creeze altele noi.

```{lang: "text/html", startCode: true}
Notes: <select></select> <button>Add</button><br>
<textarea style="width: 100%"></textarea>

<script>
  let list = document.querySelector("select");
  let note = document.querySelector("textarea");

  let state;
  function setState(newState) {
    list.textContent = "";
    for (let name of Object.keys(newState.notes)) {
      let option = document.createElement("option");
      option.textContent = name;
      if (newState.selected == name) option.selected = true;
      list.appendChild(option);
    }
    note.value = newState.notes[newState.selected];

    localStorage.setItem("Notes", JSON.stringify(newState));
    state = newState;
  }
  setState(JSON.parse(localStorage.getItem("Notes")) || {
    notes: {"shopping list": "Carrots\nRaisins"},
    selected: "shopping list"
  });

  list.addEventListener("change", () => {
    setState({notes: state.notes, selected: list.value});
  });
  note.addEventListener("change", () => {
    setState({
      notes: Object.assign({}, state.notes,
                           {[state.selected]: note.value}),
      selected: state.selected
    });
  });
  document.querySelector("button")
    .addEventListener("click", () => {
      let name = prompt("Note name");
      if (name) setState({
        notes: Object.assign({}, state.notes, {[name]: ""}),
        selected: name
      });
    });
</script>
```

{{index "getItem method", JSON, "|| operator", "default value"}}

Scriptul își setează starea inițială din valoarea `"Notes"` stocată în `localStorage` sau, dacă nu există, crează o stare exemplu care are doar o listă de cumpărături în ea. Citirea unui câmp care nu există în `localStorage` va produce `null`. Transmiterea `null` către `JSON.parse` va determina parsarea stringului `"null"` și va returna `null`. Astfel, operatorul `||` poate fi utilizat pentru a furniza o valoare implicită în această situație.

Metoda `setState` ne asigură că DOM afișează o stare dată și stochează noua stare în `localStorage`. Handlerele pentru evenimente apelează această funcție pentru a comuta la o nouă stare.

{{index "Object.assign function", [object, creation], property, "computed property"}}

Utilizarea `Object.assign` în exemplu are intenția de a crea un nou obiect care este o clonă a `state.notes`, dar cu o proprietate adăugată sau suprascrisă. `Object.assign` preia primul său argument și îi adaugă toate proprietățile din celelalte argumente pasate. Astfel, dacă primește un obiect gol, aceasta va însemna construirea unui obiect nou. Notația cu paranteze pătrate din cel de-al treilea argument este folosită pentru a crea o proprietate al cărei nume este bazat pe o valoare dinamică.

{{index "sessionStorage object", [browser, storage]}}

Există încă un obiect, similar cu `localStorage`, numit `sessionStorage`. Diferența dintre cele două este legată de volatilitatea conținutului: `sessionStorage` este curățat la sfărșitul fiecărei _sesiuni_, ceea ce pentru majoritatea browserelor este momentul în care browserul este închis.

## Rezumat

În acest capitol am discutat despre modul de funcționare a protocolului HTTP. Un _client_ trimite o cerere, care conține o metodă (de regulă, `GET`) și o cale care identifică o resursă. _Serverul_ decide apoi ce să facă cu cererea și răspunde cu un cod de stare și un corp al răspunsului. Atât cererea cât și răspunsul pot conține headere care adaugă extra informație.

Interfața prin care JavaScript în browser poate executa cereri HTTP se numește `fetch`. Efectuarea unei cereri arată cam așa:

```
fetch("/18_http.html").then(r => r.text()).then(text => {
  console.log(`The page starts with ${text.slice(0, 15)}`);
});
```

Browserele efectuează cereri `GET` pentru a aduce resursele de care au nevoie pentru a afișa pagina. O pagină poate conține și formulare, care permit introducerea informației de către utilizator și transmiterea ei ca o cerere pentru o nouă pagină, attunci când formularul este trimis.

HTML poate reprezenta diferite tipuri de câmpuri, cum ar fi text, câmpuri de bifare, câmpuri cu alegere multiplă și cîmpuri pentru fișiere.

Asemenea câmpuri pot fi inspectate și manipulate cu JavaScript. Ele declanșează evenimentul `"change"` când sunt modificate, evenimentul `"input"` când se introduce text și primesc evenimente de la tastatură atunci când dețin focusul. Proprietățile cum ar fi `value` (pentru câmpurile de text și de selectare) sau `checked` (pentru câmpuri de bifă și butoane radio) sunt utilizate pentru a citi sau seta conținutul câmpului.

Când un formular este transmis, se declanșează un eveniment `"submit"` pentru el. Un handler JavaScript ar putea apela `preventDefault` pentru acel eveniment cu scopul de a dezactiva comportamentul implicit al browserului. Câmpurile de formular pot fi folosite și în afara unui tag pentru formular.

Când un utilizator a selectat un fișier din sistemul local de fișiere pentru un câmp de alegere a fișierelor, interfața `FileReader` potae fi utilizată pentru a accesa conținutul acestui fișier dintr-un program JavaScript.

Obiectele `localStorage` și `sessionStorage` pot fi utilizate pentru a salva informație într-un mod care supraviețuiește reîncărcărilor paginii. Primul obiect salvează datele permanent (sau până când utilizatorul decide să le elimine) iar cel de-al doilea le salvează până la închiderea browserului.

## Exerciții

### Negocierea conținutului

{{index "Accept header", "media type", "document format", "content negotiation (exercise)"}}

Unul dintre lucrurile pe care HTTP le poate face se numește _negocierea conținutului_. Header-ul pentru cereri `Accept` este utilizat pentru a preciza serverului ce fel the document ar dori clientul să primească. Multe servere ignoră acest header, dar atunci când serverul cunoaște mai multe moduri de a codifica o resursă, poate consulta acest header pentru a trimite clientului resursa în formatul pe care acesta o preferă.

{{index "MIME type"}}

URL-ul [_https://eloquentjavascript.net/author_](https://eloquentjavascript.net/author) este configurat să răspundă fie cu text simplu, fie cu HTML fie cu JSON, în funcție de ce cere clientul. Aceste formate sunt identificate prin _media type_ standard `text/plain`, `text/html` și `application/json`.

{{index "headers property", "fetch function"}}

Trimiteți cereri pentru a aduce cele trei formate ale acestei resurse. Utilizați proprietatea `headers` a obiectului cu opțiunile transmis către `fetch` pentru a seta header-ul numit `Accept` la tipul media dorit.

Apoi, încercați să setați tipul media la `application/rainbows+unicorns` și vedeți ce cod de stare produce.

{{if interactive

```{test: no}
// Your code here.
```

if}}

{{hint

{{index "content negotiation (exercise)"}}

Bazați-vă în cod pe exemplele pentru `fetch` prezentate [anterior în acest capitol](http#fetch).

{{index "406 (HTTP status code)", "Accept header"}}

Cererea unui media type fals va returna un cod de răspuns 406, "Not acceptable", care este codul pe care serverul ar trebui să îl returneze dacă nu poate satisface headerul `Accept`.

hint}}

### Un banc de lucru JavaScript

{{index "JavaScript console", "workbench (exercise)"}}

Build an interface that allows people to type and run pieces of JavaScript code.

{{index "textarea (HTML tag)", "button (HTML tag)", "Function constructor", "error message"}}

Put a button next to a `<textarea>` field that, when pressed, uses the `Function` constructor we saw in [Chapter ?](modules#eval) to wrap the text in a function and call it. Convert the return value of the function, or any error it raises, to a string and display it below the text field.

{{if interactive

```{lang: "text/html", test: no}
<textarea id="code">return "hi";</textarea>
<button id="button">Run</button>
<pre id="output"></pre>

<script>
  // Your code here.
</script>
```

if}}

{{hint

{{index "click event", "mousedown event", "Function constructor", "workbench (exercise)"}}

Use `document.querySelector` or `document.getElementById` to get access to the elements defined in your HTML. An event handler for `"click"` or `"mousedown"` events on the button can get the `value` property of the text field and call `Function` on it.

{{index "try keyword", "exception handling"}}

Make sure you wrap both the call to `Function` and the call to its result in a `try` block so you can catch the exceptions it produces. In this case, we really don't know what type of exception we are looking for, so catch everything.

{{index "textContent property", output, text, "createTextNode method", "newline character"}}

The `textContent` property of the output element can be used to fill it with a string message. Or, if you want to keep the old content around, create a new text node using `document.createTextNode` and append it to the element. Remember to add a newline character to the end so that not all output appears on a single line.

hint}}

### Conway's Game of Life

{{index "game of life (exercise)", "artificial life", "Conway's Game of Life"}}

Conway's Game of Life is a simple ((simulation)) that creates artificial "life" on a ((grid)), each cell of which is either alive or not. Each ((generation)) (turn), the following rules are applied: 

* Any live ((cell)) with fewer than two or more than three live ((neighbor))s dies.

* Any live cell with two or three live neighbors lives on to the next generation.

* Any dead cell with exactly three live neighbors becomes a live cell.

A _neighbor_ is defined as any adjacent cell, including diagonally adjacent ones.

{{index "pure function"}}

Note that these rules are applied to the whole grid at once, not one square at a time. That means the counting of neighbors is based on the situation at the start of the generation, and changes happening to neighbor cells during this generation should not influence the new state of a given cell.

{{index "Math.random function"}}

Implement this game using whichever ((data structure)) you find appropriate. Use `Math.random` to populate the grid with a random pattern initially. Display it as a grid of ((checkbox)) ((field))s, with a ((button)) next to it to advance to the next ((generation)). When the user checks or unchecks the checkboxes, their changes should be included when computing the next generation.

{{if interactive

```{lang: "text/html", test: no}
<div id="grid"></div>
<button id="next">Next generation</button>

<script>
  // Your code here.
</script>
```

if}}

{{hint

{{index "game of life (exercise)"}}

To solve the problem of having the changes conceptually happen at the same time, try to see the computation of a ((generation)) as a ((pure function)), which takes one ((grid)) and produces a new grid that represents the next turn.

Representing the matrix can be done in the way shown in [Chapter ?](object#matrix). You can count live ((neighbor))s with two nested loops, looping over adjacent coordinates in both dimensions. Take care not to count cells outside of the field and to ignore the cell in the center, whose neighbors we are counting.

{{index "event handling", "change event"}}

Ensuring that changes to ((checkbox))es take effect on the next generation can be done in two ways. An event handler could notice these changes and update the current grid to reflect them, or you could generate a fresh grid from the values in the checkboxes before computing the next turn.

If you choose to go with event handlers, you might want to attach ((attribute))s that identify the position that each checkbox corresponds to so that it is easy to find out which cell to change.

{{index drawing, "table (HTML tag)", "br (HTML tag)"}}

To draw the grid of checkboxes, you can either use a `<table>` element (see [Chapter ?](dom#exercise_table)) or simply put them all in the same element and put `<br>` (line break) elements between the rows.

hint}}
