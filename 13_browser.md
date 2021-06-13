# JavaScript și browserul

{{quote {author: "Tim Berners-Lee", title: "The World Wide Web: A very short personal history", chapter: true}

Visul din spatele web-ului este cel al unui spațiu comun pentru informații în care să comunicăm prin partajarea informațiilor. Universalitatea sa este esențială: faptul că un link poate indica orice, fie personal, fie local sau global, fie schiță sau mult finisat.

quote}}

{{index "Berners-Lee, Tim", "World Wide Web", HTTP, [JavaScript, "history of"], "World Wide Web"}}

{{figure {url: "img/chapter_picture_13.jpg", alt: "Picture of a telephone switchboard", chapter: "framed"}}}

În următoarele capitole vom discuta despre browserele web. Fără ele, JavaScript nu ar fi existat. Sau chiar și dacă ar fi existat, nimeni nu i-ar fi acordat atenție.

{{index decentralization, compatibility}}

Tehnologia web a fost descentralizată de la început, nu doar tehnic dar și din punctul de vedere al modului în care a evoluat. Diferiți producători de browsere au adăugat funcționalitate nouă ad-hoc și uneori slab proiectată, care apoi a fost adoptată de alți, și apoi inclusă în standarde.

Aceasta este în același timp o binecuvântare și un blestem. Pe de o parte, permite ca sistemul să nu fie controlat de o autoritate centrală, ci să fie îmbunătățit de mai multe părți ce colaborează între anumite limite (sau uneori în ostilitate vădită). Pe de altă parte, modul aleator în care a fost dezvoltat Web-ul înseamnă că sistemul rezultat nu este chiar un exemplu strălucit de consistență internă. Unele părți sunt creatoare de confuzie și slab proiectate.

## Rețelele și Internetul

Rețelele de computere au apărut încă din anii 1950. Dacă montați cabluri între două sau mai multe computere și le permiteți să își transmită date prin aceste cabluri, puteți realiza tot felul de lucruri.

Dacă prin conectarea a două mașini din aceeași clădire putem face lucruri extraordinare, conectarea mașinilor de pe toată planeta ar trebui să fie un lucru și mai bun. Tehnologia pentru a începe implementarea acestei viziuni a fost dezvoltată în anii 1980 și rețeaua rezultată se numește _Internet_. Categoric și-a îndeplinit promisiunea.

Un computer poate folosi această rețea pentru a transmite biți către un alt computer. Pentru o comunicație eficientă, ambele computere trebuie să fie capabile să înțeleagă ce reprezintă acești biți. Semnificația oricărei secvențe de biți depinde de tipul informației pe care aceasta încearcă să o exprime și de tipul de mecanism de codificare utilizat.

{{index [network, protocol]}}

Un _protocol de rețea_ descrie un stil de comunicare peste o rețea. Există protocoale pentru transmiterea e-mail-urilor, pentru recepționarea lor, pentru distribuirea fișierelor și chiar pentru controlul computerelor care sunt infectate cu software malițios.

{{indexsee "Hypertext Transfer Protocol", HTTP}}

De exemplu, _protocolul de transfer al hipertextului (HTTP)_ este un protocol pentru returnarea resurselor denumite (bucăți de informație cum ar fi pagini web sau imagini). Acesta specifică obligativitatea pentru partea care face o cerere ca să înceapa solicitarea cu o linie asemănătoare celei de mai jos, denumind resursa și versiunea protocolului pe care încearcă să o utilizeze:

```{lang: "text/plain"}
GET /index.html HTTP/1.1
```

Există mult mai multe reguli cu privire la modul în care solicitantul poate include mai multe informații în cerere și modul în care cealaltă parte, care returnează resursa, poate să împacheteze conținutul. Vom analiza protocolul HTTP mai în detaliu în [capitolul ?](http).

{{index layering, stream, ordering}}

Majoritatea protocoalelor sunt construite peste alte protocoale. HTTP tratează rețeaua ca fiind un dispozitiv pentru fluxuri de date, în care se plasează biți care ajung la destinația corectă în ordinea corectă. Așa cum am văzut în [capitolul ?](async), asigurarea acestor aspecte este deja în sine o problemă dificilă.

{{index TCP}}

{{indexsee "Transmission Control Protocol", TCP}}

_Protocolul pentru controlul transmisiunii (TCP)_ este un protocol care adresează această problemă. Toate dispozitivele conectate la Internet îl "vorbesc" și majoritatea comunicației pe Internet este construită pe baza acestuia.

{{index "listening (TCP)"}}

O conexiune TCP funcționează astfel: un computer trebuie să aștepte, sau să _asculte (listening)_, pentru alte computere care ar putea începe să comunice cu el. Pentru a putea asculta pentru diferite tipuri de comunicație în același timp, pe o singură mașină, fiecare listener are un număr asociat lui (numit _port_). Majoritatea protocoalelor specifică portul implicit ce va fi utilizat. De exemplu, când intenționăm să trimitem un email utilizând protocolul SMTP, mașina prin care îl trimitem este așteptată să asculte pe portul 25.

Un alt computer poate apoi să stabilească o conexiune cu mașina țintă utilizând portul corect. Dacă mașina țintă poate fi atinsă și ascultă pe acel port, conexiunea este creată cu succes. Computerul care ascultă se numește _server_ iar cel care se conectează se numește _client_.

{{index [abtraction, "of the network"]}}

O asemenea conexiune funcționează ca și o "conductă" bidirecțională prin care biții pot să "curgă" - mașinile de la ambele capete pot să pună biți în conductă. Imediat ce biții au fost transmiși cu succes, ei pot fi citiți de către mașina de la celălalt capăt. Acesta este un model convenabil. Putem spune că TCP furnizează o abstracție a rețelei.

{{id web}}

## Web-ul

_World Wide Web_ (a nu se confunda cu Internetul ca și întreg) este un set de protocoale și formate care ne permit să vizităm pagini într-un browser. Cuvântul "Web" din nume se referă la faptul că asemenea pagini pot fi ușor legate între ele, conectându-se astfel într-o rețea imensă prin care utilizatorii pot să se deplaseze.

Pentru a deveni parte din Web, tot ceea ce trebuie să faceți este să conectați o mașină la Internet și să o puneți să asculte pe portul 80 cu protocolul HTTP astfel încât alte computere să îi poată cere documente.

{{index URL}}

{{indexsee "Uniform Resource Locator", URL}}

Fiecare document de pe Web are o denumire bazată pe un URL (_Uniform Resource Locator_), care arată astfel:

```{lang: null}
  http://eloquentjavascript.net/13_browser.html
 |      |                      |               |
 protocol       server               path
```

{{index HTTPS}}

Prima parte ne spune că acest URL folosește protocolul HTTP (spre deosebire de, de exemplu, HTTP encriptat care ar fi _https://_). Apoi urmează partea care identifică serverul de la care facem solicitarea documentului. Apoi avem un string care identifică documentul specific (sau _resursa_) care ne interesează.

Mașinile conectate la Internet primesc o _adresă IP_ care este un număr ce poate fi utilizat pentru a trimite mesaje la aceea mașină și arată cam așa: `149.210.142.219` sau `2001:4860:4860::8888`. Dar aceste numere sunt mai mult sau mai puțin aleatoare și greu de reținut, astfel că putem înregistra un _nume de domeniu_ pentru una sau mai multe adrese. Asocierea unui nume pentru adresa IP face mai ușoară memorarea acestuia de către utilizatori.

{{index browser}}

Dacă introduceți URL-ul de mai sus în bara de adresă a browserului, browserul va încercă să returneze și să afișeze documentul de la acel URL. Mai întâi, browserul va încerca să identifice la ce se referă adresa _eloquentjavascript.net_. Apoi, utilizând protocolul HTTP, va realiza o conexiune la serverul de la adresa respectivă și va solicita resursa _/13_browser.html_. Dacă totul merge bine, serverul va transmite un document care va fi apoi afișat de către browserul vostru pe ecran.

## HTML

{{index HTML}}

{{indexsee "Hypertext Markup Language", HTML}}

HTML, prescurtarea de la _Hypertext Markup Language_, este formatul pentru documente utilizat în paginile web. Un document HTML conține text și _tag-uri_ care dau structură textului, descriind lucruri cum ar fi legături, paragrafe sau titluri.

Un document HTML scurt ar arăta cam așa:

```{lang: "text/html"}
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
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

{{if book

Cam așa ar arăta un astfel de document în browser:

{{figure {url: "img/home-page.png", alt: "My home page",width: "6.3cm"}}}

if}}

{{index [HTML, notation]}}

Tagurile, plasate între paranteze unghiulare (`<` și `>`), ne dau informații despre structura documentului. Restul conținutului este doar text obișnuit. 

{{index doctype, version}}

Documentul începe cu `<!doctype html>`, ceea ce informează browserul că pagina poate fi interpretată ca și HTML _modern_. În trecut au fost utilizate alte dialecte (versiuni ale HTML)

{{index "head (HTML tag)", "body (HTML tag)", "title (HTML tag)", "h1 (HTML tag)", "p (HTML tag)"}}

Documentele HTML au un antet și un corp. Antetul conține informații despre document iar corpul reprezintă conținutul documentului în sine. În cazul nostru, antetul declară că titlul documentului este "My home page" și că se utilizează codificare UTF-8, care este o modalitate de a codifica textul Unicode ca și date binare. Corpul documentului conține un titlu (`<h1>`, cu semnificația "heading 1"—`<h2>` până la `<h6>` permit definirea unor subtitluri pe mai multe nivele) și două paragrafe (`<p>`).

{{index "href attribute", "a (HTML tag)"}}

Tagurile au mai multe forme. Un element începe cu un _tag de deschidere_ cum ar fi `<p>` și se termină cu un tag de închidere, cum ar fi `</p>`. Unele taguri conțin informații suplimentare, numite _atribute_, sub forma `name="value"`. De exemplu, legatura creată în documentul anterior prin tag-ul `<a>` are destinația specificată prin atributul `href="http://eloquentjavascript.net"`, în care `href` înseamnă "hypertext reference".

{{index "src attribute", "self-closing tag", "img (HTML tag)"}}

Unele taguri nu conțin text deci nu trebuie să fie închise. Un exemplul este tagul pentru metadate `<meta charset="utf-8">`.

{{index [escaping, "in HTML"]}}

Pentru a putea include parantezele unghiulare în textul documentului, chiar dacă ele au semnificație specială în HTML, există o notație specială. Puteți introduce o paranteză unghiulară deschisă folosind `&lt;` ("less than"), iar o paranteză de închidere o puteți insera cu `&gt;` ("greater than"). În HTML, un caracter ampersand (`&`) urmat de un nume sau un cod de caracter și un punct și virgulă (`;`) se numește _entitate_ și va fi înlocuit de către caracterul pe care îl codifică.

{{index ["backslash character", "in strings"], "ampersand character", "double-quote character"}}

Aceasta este o abordare similară celei utilizate de JavaScript pentru stringuri. Deoarece acest mecanism oferă semnificație specială caracterului ampersand, acesta va putea fi introdus folosind `&amp;`. În valorile atributelor, care sunt precizate între ghilimele, puteți utiliza `&quot;` pentru a insera un caracter reprezentând ghilimelele.

{{index "error tolerance", parsing}}

HTML se interpretează cu o toleranță remarcabilă la erori. Când taguri care ar trebui să fie prezente lipsesc, browserul le reconstruiește. Modul în care se realizează acest lucru a fost standardizat și puteți fi siguri că toate browserele moderne realizează acest lucru în același mod.

Documentul următor va fi tratat ca și cel prezentat anterior:

```{lang: "text/html"}
<!doctype html>

<meta charset=utf-8>
<title>My home page</title>

<h1>My home page</h1>
<p>Hello, I am Marijn and this is my home page.
<p>I also wrote a book! Read it
  <a href=http://eloquentjavascript.net>here</a>.
```

{{index "title (HTML tag)", "head (HTML tag)", "body (HTML tag)", "html (HTML tag)"}}

Tagurile `<html>`, `<head>` și `<body>` au fost eliminate. Browserul știe că `<meta>` și `<title>` fac parte din antet iar `<h1>` însemnă că începe corpul documentului. Mai mult, nu mai închidem paragrafele pentru că deschiderea unui nou paragraf înseamnă că s-a încheiat paragraful anterior deci acesta este închis implicit.

Aceasta carte va omite de regulă tagurile `<html>`, `<head>` și `<body>` în exemple, pentru claritate și concizie. Dar vom închide tagurile și vom include valorile atributelor între ghilimele.

{{index browser}}

De asemenea, voi omite declarația `doctype` și `charset`. Dar aceasta nu înseamnă că vă încurajez să le eliminați din documentele HTML. Browserele vor face adesea greșeli ridicole atunci când le omiteți. Mai degrabă considerați că declarațiile `doctype` și `charset` sunt prezente implicit în exemplele prezentate, chiar dacă ele nu apar în text.

{{id script_tag}}

## HTML și JavaScript

{{index [JavaScript, "in HTML"], "script (HTML tag)"}}

În contextul acestei cărți, cel mai important tag HTML este `<script>`. Acest tag permite introducerea unei bucăți de cod JavaScript în document.

```{lang: "text/html"}
<h1>Testing alert</h1>
<script>alert("hello!");</script>
```

{{index "alert function", timeline}}

Un asemenea script va fi executat imediat ce browserul identifică tagul `<script>` în timp ce citește HTML.

{{index "src attribute"}}

Includerea unor programe mai lungi direct în codul HTML este adesea nepractică. Tagul `<script>` are un atribut `src` a cărui valoare reprezintă URL-ul unui fișier script care să fie inclus (un fișier text ce conține cod JavaScript).

```{lang: "text/html"}
<h1>Testing alert</h1>
<script src="code/hello.js"></script>
```

Fișierul _code/hello.js_ inclus în exemplul de mai sus conține același program - `alert("hello!")`. Când o pagină HTML se referă la alte URL-uri, cum ar fi un fișier imagine sau un script, browserele web vor descărca imediat resursa și o vor include în pagină.

{{index "script (HTML tag)", "closing tag"}}

Un tag script trebuie întotdeauna închis cu `</script>`, chiar dacă se referă la un fișier și nu conține cod. Dacă omiteți acest lucru, restul paginii va fi interpretat ca și parte a scriptului.

{{index "relative path", dependency}}

Puteți încărca module ES (consultați [capitolul ?](modules#es)) în browser prin precizarea atributului `type="module"` pentru tagul `script`. Asemenea module pot să depindă de alte module prin utilizarea unor URL-uri relative la locația lor ca și nume ale unor module în declarațiile `import`

{{index "button (HTML tag)", "onclick attribute"}}

Unele atribute pot la rândul lor să conțină un program JavaScript. Tagul `<button>` din exemplul de mai jos, are un atribut `onclick`. Valoarea atributului este un cod JavaScript ce va fi executat ori de câte ori se efectuează click pe buton.

```{lang: "text/html"}
<button onclick="alert('Boom!');">DO NOT PRESS</button>
```

{{index "single-quote character", [escaping, "in HTML"]}}

Observați că am utilizat apostroafe pentru stringul JavaScript deoarece ghilimele le-am utilizat pentru a specifica valoarea atributului. Dar aș fi putut utiliza și `&quot;`.

## În "sandbox"

{{index "malicious script", "World Wide Web", browser, website, security}}

Rularea programelor descărcate de pe Internet este potențial periculoasă. Nu cunoașteți prea multe despre persoanele din spatele majorității site-urilor pe care le vizitați, și aceste persoane nu sunt neapărat bine intenționate. Programele rulate de către persoane fără intenții prea bune conduce la infectarea computerului vostru, furtul de date și spargerea conturilor voastre.

Totuși atracția web-ului este că puteți să navigați fără a fi obligați să aveți încredere în toate paginile pe care le vizitați. De aceea browserele limitează sever lucrurile pe care un program JavaScript poate să le facă: nu poate să acceseze fișierele din computerul vostru și nici nu poate modifica nimic ce nu este legat de pagina web în care a fost inclus.

{{index isolation}}

Izolarea unui mediu de programare în acest mod se numește _sandboxing_, ideea fiind că un program poate fi executat astfel fără efecte dăunătoare. Dar imaginați-vă că acest "sandbox" este de fapt o cușcă cu bare groase de oțel, astfel că programul din interior nu are nici o șansă să iasă.

Partea dificilă în "sandboxing" este datorată faptului că programul trebuie să aibă suficientă libertate pentru a fi util dar, în același timp, să fie restricționat în a realiza acțiuni periculoase. Multă funcționalitate utilă, cum ar fi comunicarea cu alte servere sau citirea conținutului din clipboard, poate și ea să fie utilizată pentru a executa operații problematice, care vă pot invada intimitatea.

{{index leak, exploit, security}}

Din când în când, cineva vine cu o idee nouă de a ocoli limitările unui browser și a provoca lucruri dăunătoare, mergând de la scurgeri minore de informații private, până la preluarea controlului asupra întregului computer în care rulează browserul. Developerii browserului răspund prin închiderea golului și totul este în regulă din nou, până când este desoperită o altă problemă, care să sperăm că va fi făcută publică în loc să fie exploatată în secret de o agenție guvernamentală sau de către mafie.

## Compatibilitatea și războiul browserelor

{{index Microsoft, "World Wide Web"}}

La începutul Web-ului, piața era dominată de către un browser numit _Mosaic_. După câțiva ani, balanța s-a înclinat către _Netscape_, care apoi a fost înlocuit pe larg de către _Internet Explorer_ de la Microsoft. În orice perioadă în care un singur browser a fost dominant, compania dezvoltatoare s-a simțit îndreptățită să inventeze unilateral noi funcționalități pentru Web. Deoarece majoritatea utilizatorilor foloseau cel mai popular browser, site-urile web începeau să folosească aceste funcționalități, ignorând alte browsere.

Acesta a fost evul întunecat al compatibilității, denumit _războiul browserelor_. Developerii web nu aveau la îndemână un Web unificat ci două-trei platforme incompatibile. Pentru a înrăutăți lucrurile, browserele utilizate în perioada 2003 erau pline de buguri și bugurile erau diferite pentru fiecare browser. Viața era grea pentru cei care creau pagini web.

{{index Apple, "Internet Explorer", Mozilla}}

Mozilla Firefox, o organizație non-profit a Netscape, a contestat poziția Internet Explorer-ului la sfârșitul anilor 2000. Deoarece Microsoft nu avea nici un interes particular în a rămâne competitiv pe atunci, Firefox a reușit să preia o mare cotă din piață. Cam în aceași perioadă, Google a introdus browserul Chrome iar browserul Safari de la Apple a câștigat popularitate, conducând la o situație în care erau patru jucători majori pe piață.

{{index compatibility}}

Noii jucători au avut o atitudine mai serioasă în privința standardelor și practici de inginerie mai bune, ceea ce a condus la o incompatibilitate mai scăzută și mai puține buguri. Microsoft, văzând o diminuare continuă a cotei sale de piață, a adoptat aceste atitudini pentru dezvoltarea browserului său Edge, care înlocuiește Internet Exporer. Dacă începeți să învățați web development azi, considerați-vă norocoși. Ultimele versiuni ale browserelor majore au un comportament destul de uniform și relativ puține buguri.
