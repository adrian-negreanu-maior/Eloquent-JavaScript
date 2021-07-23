{{meta {load_files: ["code/chapter/12_language.js"], zip: "node/html"}}}

# Proiect: un limbaj de programare

{{quote {author: "Hal Abelson and Gerald Sussman", title: "Structure and Interpretation of Computer Programs", chapter: true}

Evaluatorul, care determină semnificația unei expresii într-un limbaj de programare, este doar un alt program.

quote}}

{{index "Abelson, Hal", "Sussman, Gerald", SICP, "project chapter"}}

{{figure {url: "img/chapter_picture_12.jpg", alt: "Imaginea unui ou cu mai multe ouă în interior", chapter: "framed"}}}

Construirea propriului limbaj de programare este surprinzător de ușoară (cât timp nu țintiți prea sus) și foarte revelatoare.

Principalul lucru pe care vreau să vi-l arăt este că nu există magie în construirea propriului limbaj. Adesea am avut senzația că unele invenții au fost atât de inteligente și de complicate încât nu voi fi niciodată capabil să le înțeleg. Dar puțină lectură și câteva experimente le-au transformat în ceva obișnuit.

{{index "Egg language", [abstraction, "in Egg"]}}

Vom construi un limbaj de programare numit _Egg_. Va fi un limbaj simplu, dar suficient de puternic pentru a putea exprima orice computație la care v-ați putea gândi. Va permite abstracții simple, bazate pe funcții.

{{id parsing}}

## Parsarea

{{index parsing, validation, [syntax, "of Egg"]}}

Cea mai vizibilă parte a unui limbaj de programare este _sintaxa_, sau notațiile. Un _parser_ este un program care citește o bucată de text și produce o structură de date care reflectă structura programului conținut în acel text. Dacă textul nu formează un program valid, parserul ar trebui să indice eroarea.

{{index "special form", [function, application]}}

Limbajul nostru va avea o sintaxă simplă și uniformă. Totul în Egg va fi o expresie. O expresie poate fi numele unui binding, un număr, un string sau o _aplicație_. Aplicațiile sunt utilizate pentru apelurile de funcții dar și pentru construcții cum ar fi `if` sau `while`.

{{index "double-quote character", parsing, [escaping, "in strings"], [whitespace, syntax]}}

Pentru a păstra lucrurile simple, stringurile în Egg nu vor permite secvențe escape. Un string va fi doar o secvență de caractere care nu sunt ghilimele, plasate între ghilimele. Un număr este o secvență de cifre. Numele bindingurilor pot conține orice caracter care nu este un spațiu și nu are semnificație specială pentru sintaxă.

{{index "comma character", [parentheses, arguments]}}

Aplicațiile sunt scrise ca și în JavaScript, prin plasarea parantezelor după o expresie și pot avea oricâte argumente între paranteze, separate prin virgulă.

```{lang: null}
do(define(x, 10),
   if(>(x, 5),
      print("large"),
      print("small")))
```

{{index block, [syntax, "of Egg"]}}

Uniformitatea limbajului Egg înseamnă că operatorii din JavaScript, cum ar fi (`>`) sunt bindinguri normale în acest limbaj, aplicate ca și alte funcții. Și, deoarece sintaxa nu are conceptul de bloc, avem nevoie de un construct `do` pentru a reprezenta execuția mai multor acțiuni într-o secvență.

{{index "type property", parsing, ["data structure", tree]}}

Structura de date pe care parserul o va utiliza pentru a descrie un program constă din obiecte pentru expresii, fiecare având o proprietate `type` ce indică tipul de expresie și alte proprietăți, pentru a descrie conținutul.

{{index identifier}}

Expresiile de tip `"value"` reprezintă literali de tip string sau număr. Proprietatea `value` conține stringul sau numărul pe care îl reprezintă. Expresiile de tip `"word"` sunt utilizate pentru identificatori (nume). Asemenea obiecte au o proprietate `name` care reține numele idnetificatorului ca și un string. În sfârșit, expresiile `"apply"` reprezintă aplicații. Ele au o proprietate `operator` care se referă la expresia ce va fi aplicată, precum și o proprietate `args` ce reține un array de expresii ca și argumente.

Partea `>(x, 5)` din programul antrerior va fi reprezentată astfel:

```{lang: "application/json"}
{
  type: "apply",
  operator: {type: "word", name: ">"},
  args: [
    {type: "word", name: "x"},
    {type: "value", value: 5}
  ]
}
```

{{indexsee "abstract syntax tree", "syntax tree", ["data structure", tree]}}

O asemenea structură de date se numește _arbore de sintaxă_. Dacă reprezentați obiectele prin puncte și legăturile dintre ele prin linii, veți obține o structură asemănătoare unui arbore. Faptul că expresiile conțin alte expresii, care ar putea la rândul lor să conțină alte expresii este similar ramificațiilor din coroana unui arbore.

{{figure {url: "img/syntax_tree.svg", alt: "Structura unui arbore de sintaxă",width: "5cm"}}}

{{index parsing}}

Comparați aceasta cu parserul pe care l-am scris pentru formatul fișierelor de configurare în [capitolul ?](regexp#ini), care avea o structură simplă: împărțea intrarea în linii și prelucra acele linii câte una pe rând. O linie putea avea doar câteva forme simple.

{{index recursion, [nesting, "of expressions"]}}

Acum trebuie să găsim o abordare diferită. Expresiile nu sunt separate în linii și au o structură recursivă. Expresiile de tip aplicație _conțin_ alte expresii.

{{index elegance}}

Din fericire, această problemă poate fi rezolvată elegant prin scrierea unei funcții de parsare care să fie recursivă și să reflecte astfel natura recursivă a limbajului.

{{index "parseExpression function", "syntax tree"}}

Vom defini o funcție `parseExpression`, care va prelua un string ca și intrare și va returna obiectul ce conține structura de date pentru expresia de la începutul stringului, împreună cu partea din string rămasă după parsarea expresiei. Când vom parsa subexpresii (argumentele unei aplicații de exemplu), această funcție va fi apelată din nou, producând expresia argument precum și textul care rămâne. Acest text ar putea conține mai multe argumente sau ar putea fi paranteza de închidere de la finalul listei de argumente.

Acesta este prima parte a parserului:

```{includeCode: true}
function parseExpression(program) {
  program = skipSpace(program);
  let match, expr;
  if (match = /^"([^"]*)"/.exec(program)) {
    expr = {type: "value", value: match[1]};
  } else if (match = /^\d+\b/.exec(program)) {
    expr = {type: "value", value: Number(match[0])};
  } else if (match = /^[^\s(),#"]+/.exec(program)) {
    expr = {type: "word", name: match[0]};
  } else {
    throw new SyntaxError("Unexpected syntax: " + program);
  }

  return parseApply(expr, program.slice(match[0].length));
}

function skipSpace(string) {
  let first = string.search(/\S/);
  if (first == -1) return "";
  return string.slice(first);
}
```

{{index "skipSpace function", [whitespace, syntax]}}

Deoarece Egg, ca și JavaScript, permite oricâte spații între elemente, trebuie să eliminăm repetat spațiile albe de la începutul stringului ce reprezintă programul. Pentru aceasta ne ajută funcția `skipSpace`.

{{index "literal expression", "SyntaxError type"}}

După ce sutn eliminate spațiile albe, `parseExpression` folosește trei expresii regulate pentru a identifica cele trei elemente atomice pe care Egg le suportă: stringuri, numere și cuvinte. Parserul construiește un tip de structură diferit în funcție de expresia pe care o potrivește. Dacă intrarea nu potrivește nici una dintre cele trei forme, expresia nu este validă și parserul va arunca o excepție. Vom utiliza `SyntaxError` ca și constructor, în loc de `Error`, care este un alt tip de eroare standard, deoarece este puțin mai specific - acesta este și tipul de eroare aruncat când se încearcă execuția unui program JavaScript invalid.

{{index "parseApply function"}}

Apoi eliminăm partea care a fost potrivită din stringul programului și transmitem stringul rămas și obiectul pentru expresie către `parseApply`, care verifică dacă expresia este o aplicație. Dacă da, parsează lista de argumente din paranteze.

```{includeCode: true}
function parseApply(expr, program) {
  program = skipSpace(program);
  if (program[0] != "(") {
    return {expr: expr, rest: program};
  }

  program = skipSpace(program.slice(1));
  expr = {type: "apply", operator: expr, args: []};
  while (program[0] != ")") {
    let arg = parseExpression(program);
    expr.args.push(arg.expr);
    program = skipSpace(arg.rest);
    if (program[0] == ",") {
      program = skipSpace(program.slice(1));
    } else if (program[0] != ")") {
      throw new SyntaxError("Expected ',' or ')'");
    }
  }
  return parseApply(expr, program.slice(1));
}
```

{{index parsing}}

Dacă următorul caracter din program nu este o paranteză de deschidere, aceasta nu este o aplicație și `parseApply` returnează expresia care i-a fost transmisă.

{{index recursion}}

Altfel, sare peste paranteza deschisă și crează arborele de sintaxă pentru această expresie de tip aplicație. Apoi apelează recursiv `parseExpression` pentru a parsa fiecare argument până când găsește o paranteză închisă. Recursia este indirectă, `parseApply` și `parseExpression` apelându-se reciproc.

Deoarece o expresie de tip aplicație poate la rândul său să fie aplicată (ca și în `multiplier(2)(1)`), `parseApply` trebuie să se apeleze din nou după ce a parsat o aplicație pentru a verifica dacă nu cumva urmează o altă pereche de parantteze.

{{index "syntax tree", "Egg language", "parse function"}}

Acum avem tot ce ne trebuie pentru a parsa Egg. Îl împachetăm într-o funcție convenabilă `parse` care verifică dacă a ajuns la sfârșitul stringului parsat după parsarea expresiei (un program Egg este o singură expresie) și ne returnează structura de date reprezentând programul.

```{includeCode: strip_log, test: join}
function parse(program) {
  let {expr, rest} = parseExpression(program);
  if (skipSpace(rest).length > 0) {
    throw new SyntaxError("Unexpected text after program");
  }
  return expr;
}

console.log(parse("+(a, 10)"));
// → {type: "apply",
//    operator: {type: "word", name: "+"},
//    args: [{type: "word", name: "a"},
//           {type: "value", value: 10}]}
```

{{index "error message"}}

Funcționează! Nu ne oferă informații foarte utile când eșuează și nu memorează linia și coloana pe care expresia începe, ceea ce ar putea fi util în raportarea erorilor, dar este suficient pentru scopul nostru.

## Evaluatorul

{{index "evaluate function", evaluation, interpretation, "syntax tree", "Egg language"}}

Ce putem face cu arborele de sintaxă al unui program? Să îl rulăm, desigur! Și acesta este rolul evaluatorului. Îi transmiteți un arbore de sintaxă și un obiect care asociază numele cu valori și va evalua expresia reprezentată de către arbore, returnând valoarea produsă.

```{includeCode: true}
const specialForms = Object.create(null);

function evaluate(expr, scope) {
  if (expr.type == "value") {
    return expr.value;
  } else if (expr.type == "word") {
    if (expr.name in scope) {
      return scope[expr.name];
    } else {
      throw new ReferenceError(
        `Undefined binding: ${expr.name}`);
    }
  } else if (expr.type == "apply") {
    let {operator, args} = expr;
    if (operator.type == "word" &&
        operator.name in specialForms) {
      return specialForms[operator.name](expr.args, scope);
    } else {
      let op = evaluate(operator, scope);
      if (typeof op == "function") {
        return op(...args.map(arg => evaluate(arg, scope)));
      } else {
        throw new TypeError("Applying a non-function.");
      }
    }
  }
}
```

{{index "literal expression", scope}}

Evaluatorul are cod pentru fiecare tip de expresie. O expresie reprezentând o valoare literală își produce valoarea. (De exemplu, expresia `100` se evaluează la numărul 100). Pentru un binding, trebuie să verificăm dacă este definit în domeniul de vizibilitate și, dacă da, să extragem valoarea bindingului.

{{index [function, application]}}

Aplicațiile sunt mai elaborate. Dacă au o formă specială, cum ar fi `if`, nu evaluăm nimic și transmitem expresiile argument împreună cu obiectul ce definește domeniul de vizibilitate, spre funcția care gestionează această formă. Dacă este un apel normal, evaluăm operatorul, verificăm dacă este o funcție și o apelăm cu argumentele evaluate.

Utilizăm valori de tip funcție JavaScript pentru a reprezenta valorile de tip funcție în Egg. Vom reveni [mai târziu](language#egg_fun), când vom defini forma specială numită `fun`.

{{index readability, "evaluate function", recursion, parsing}}

Structura recursivă a funcției `evaluate` se aseamănă cu structura similară a parserului și ambele reflectă structura programului în sine. Am putea chiar să integrăm parserul cu evaluatorul și să evaluăm pe durata parsării, dar separând cele două concepte avem o structură mai clară a programului.

{{index "Egg language", interpretation}}

Cam de atât avem nevoie pentru a interpreta Egg. Este atât de simplu. Dar fără a defini câteva forme speciale și a adăuga câteva valori utile la mediu, încă nu putem realiza mare lucru cu acest limbaj.

## Forme speciale

{{index "special form", "specialForms object"}}

Obiectul `specialForms` este utilizat pentru a defini sintaxa specială în Egg. Acesta asociază cuvinte cu funcții care evaluează asemenea forme. Deocamdată este un obiect gol. Haideți să adăugăm `if`.

```{includeCode: true}
specialForms.if = (args, scope) => {
  if (args.length != 3) {
    throw new SyntaxError("Wrong number of args to if");
  } else if (evaluate(args[0], scope) !== false) {
    return evaluate(args[1], scope);
  } else {
    return evaluate(args[2], scope);
  }
};
```

{{index "conditional execution", "ternary operator", "?: operator", "conditional operator"}}

Construcția `if` a limbajului Egg așteaptă exact trei argumente. Va evalua primul argument și dacă rezultatul nu reprezintă valoarea `false` va evalua cel de-al doilea argument. Altfel, cel de al treilea argument va fi evaluat. Această forma a `if` este asemănătoare operatorului ternar din JavaScript `?:`, și seamănă mai puțin cu `if` din JavaScript. Este o expresie, nu o instrucțiune, și va produce o valoare, rezultatul evaluării celui de-al doilea sau celui de-al treilea argument.

{{index Boolean}}

Egg diferă de JavaScript și prin modul în care gestionează condiția pentru `if`. Nu va trata zero sau stringul gol ca fiind false, doar valoarea exactă `false`.

{{index "short-circuit evaluation"}}

Motivul pentru care trebuie să reprezentăm `if` ca o formă specială și nu ca o funcție obișnuită este faptul că toate argumentele spre funcții sunt evaluate înainte ca funcțiile să fie apelate, în timp ce `if` trebuie să evalueze doar argumentul relevant dintre cel de-al doilea și cel de-al treilea, în funcție de valoarea evaluată a primului argument.

Forma `while` este asemănătoare.

```{includeCode: true}
specialForms.while = (args, scope) => {
  if (args.length != 2) {
    throw new SyntaxError("Wrong number of args to while");
  }
  while (evaluate(args[0], scope) !== false) {
    evaluate(args[1], scope);
  }

  // Since undefined does not exist in Egg, we return false,
  // for lack of a meaningful result.
  return false;
};
```

Un alt bloc de bază este "do", care execută toate argumentele sale de sus în jos.Valoarea sa este valoarea produsă de ultimul argument.

```{includeCode: true}
specialForms.do = (args, scope) => {
  let value = false;
  for (let arg of args) {
    value = evaluate(arg, scope);
  }
  return value;
};
```

{{index ["= operator", "in Egg"], [binding, "in Egg"]}}

Pentru a putea crea bindinguri și să le dăm valori noi, vom crea o formă numită `define`. Va primi un cuvânt ca și prim argument și o expresie ce produce o valoare ce va fi atribuită acelui cuvânt ca și al doilea argument. Deoarece `define` este și ea o expresie, va trebui să returneze o valoare. O vom construi astfel încât să returneze valoarea atribuită (ca și operatorul `=` din JavaScript).

```{includeCode: true}
specialForms.define = (args, scope) => {
  if (args.length != 2 || args[0].type != "word") {
    throw new SyntaxError("Incorrect use of define");
  }
  let value = evaluate(args[1], scope);
  scope[args[0].name] = value;
  return value;
};
```

## Mediul

{{index "Egg language", "evaluate function", [binding, "in Egg"]}}

Domeniul de vizibilitate acceptat de `evaluate` este un obiect cu proprietăți ale căror nume corespund numelor bindingurilor și ale căror valori corespund valorilor de care acele bindinguri sunt legate. Să definim un obiect pentru a reprezenta domeniul de vizibilitate global.

Pentru a putea utiliza construcția `if` pe care am declarat-o, trebuie să avem acces la valori booleene. Deoarece avem doar două valori booleene, nu avem nevoie de o sintaxa specială pentru ele. Doar legăm două nume de valorile `true` și `false` și le utilizăm ca atare.

```{includeCode: true}
const topScope = Object.create(null);

topScope.true = true;
topScope.false = false;
```

Acum putem evalua o expresie simplă care neagă o valoare booleană.

```
let prog = parse(`if(true, false, true)`);
console.log(evaluate(prog, topScope));
// → false
```

{{index arithmetic, "Function constructor"}}

Pentru a defini operatorii de bază pentru operații aritmetice și comparații, vom adăuga de asemenea câteva valori de tip funcție în domeniul de vizibilitate. Pentru a păstra codul scurt, vom utiliza `Function` pentru a defini o serie de operatori într-o buclă, în loc să îi definim individual.

```{includeCode: true}
for (let op of ["+", "-", "*", "/", "==", "<", ">"]) {
  topScope[op] = Function("a, b", `return a ${op} b;`);
}
```

O modalitate de a afișa valori ar fi de asemenea utilă, astfel că vom împacheta `console.log` într-o funcție numită `print`.

```{includeCode: true}
topScope.print = value => {
  console.log(value);
  return value;
};
```

{{index parsing, "run function"}}

Cele de mai sus ne dau suficiente instrumente pentru a scrie programe simple. Funcția de mai jos ne dă o modalitate convenabilă de a parsa un program și a-l rula într-un domeniu de vizibilitate nou:

```{includeCode: true}
function run(program) {
  return evaluate(parse(program), Object.create(topScope));
}
```

{{index "Object.create function", prototype}}

Vom utiliza lanțuri de prototipuri pentru a reprezenta domeniile de vizibilitate subordonate astfel încât programul va putea adăuga bindinguri în domeniul său de vizibilitate local fără a modifica domeniul de vizibilitate principal.

```
run(`
do(define(total, 0),
   define(count, 1),
   while(<(count, 11),
         do(define(total, +(total, count)),
            define(count, +(count, 1)))),
   print(total))
`);
// → 55
```

{{index "summing example", "Egg language"}}

Acesta este programul pe care l-am vazut de câteva ori în trecut, care calculează suma numerelor de la 1 la 10, exprimat în Egg. Este evident mai urât decât programul JavaScript echivalent - dar nu e rău pentru un limbaj de programare implementat în mai puțin de 150 de linii de cod.

{{id egg_fun}}

## Funcții

{{index function, "Egg language"}}

Un limbaj de programare fără funcții este un limbaj de programare sărac.

Din fericire, nu este deloc greu să adăugăm o construcție `fun`, care tratează ultimul argument ca fiind corpul funcției și utilizează toate argumentele anterioare ca și nume ale parametrilor funcției.

```{includeCode: true}
specialForms.fun = (args, scope) => {
  if (!args.length) {
    throw new SyntaxError("Functions need a body");
  }
  let body = args[args.length - 1];
  let params = args.slice(0, args.length - 1).map(expr => {
    if (expr.type != "word") {
      throw new SyntaxError("Parameter names must be words");
    }
    return expr.name;
  });

  return function() {
    if (arguments.length != params.length) {
      throw new TypeError("Wrong number of arguments");
    }
    let localScope = Object.create(scope);
    for (let i = 0; i < arguments.length; i++) {
      localScope[params[i]] = arguments[i];
    }
    return evaluate(body, localScope);
  };
};
```

{{index "local scope"}}

Funcțiile în Egg vor avea propriul domeniu de vizibilitate local. Funcția produsă de forma `fun` crează acest domeniu de vizibilitate local și își adaugă argumentele pentru bindinguri. Apoi evaluează corpul funcției în acest domeniu de vizibilitate și returnează rezultatul.

```{startCode: true}
run(`
do(define(plusOne, fun(a, +(a, 1))),
   print(plusOne(10)))
`);
// → 11

run(`
do(define(pow, fun(base, exp,
     if(==(exp, 0),
        1,
        *(base, pow(base, -(exp, 1)))))),
   print(pow(2, 10)))
`);
// → 1024
```

## Compilarea

{{index interpretation, compilation}}

Ceea ce am construit este un interpretor. Pe parcursul evaluării, acesta acționează direct asupra reprezentării programului produsă de către parser.

{{index efficiency, performance, [binding, definition], [memory, speed]}}

_Compilarea_ este procesul prin care se adaugă încă un pas între parsarea și rularea unui program, care transformă programul în ceva ce poate fi evaluat mai eficient prin efectuarea în avans a cât mai mult efort posibil. De exemplu, în limbajele bine concepute, este evident pentru fiecare utilizare a unui binding ce binding este referit, fără a rula programul. Aceasta poate fi utilizată pentru a evita căutarea bindingului după nume de fiecare dată când este accesat, prin extragerea sa directă dintr-o anumită locație de memorie predeterminată.

Tradițional, compilarea presupune conversia programului în _cod-mașină_, formatul brut pe care procesorul computerului poate să îl execute. Dar orice proces care convertește un program într-o formă diferită poate fi considerat "compilare".

{{index simplicity, "Function constructor", transpilation}}

Am putea scrie o strategie alternativă de evaluare pentru Egg, una care mai întâi convertește programul într-un program JavaScript, utilizează `Function` pentru a invoca compilatorul JavaScript asupra lui și apoi rulează rezultatul. Implementată corect, această abordare ar permite ca programele Egg să ruleze foarte rapid dar să rămână relativ simmplu de scris.

Dacă sunteți interesați și doriți să petreceți ceva timp, vă încurajez să încercați să implementați un asemenea compilator ca și exercițiu.

## Înșelăciune

{{index "Egg language"}}

Când am definit `if` și `while`, probabil ați observat că ele sunt împachetări mai mult sau mai puțin triviale ale `if` și `while` din JavaScript. Similar, valorile din Egg sunt doar valori obișnuite din JavaScript. Este

Dacă comparașii implementarea Egg, construită deasupra JavaScript, cu volumul de muncă și complexitatea necesară pentru a construi un limbaj de programare direct peste funcționalitatea brută furnizată de mașină, diferența este imensă. Dar sper că acest exemplu v-a dat o idee solidă despre cum funcționează limbajele de programare.

Iar când vine vorba despre a vă termina munca, această abordare ar fi cu siguranță mult mai eficientă decât să faceți totul de la zero. Deși limbajul de jucărie pe care l-am construit în acest capitol nu face nimic care nu ar putea fi făcut mai bine în JavaScript, _există_ situații în care scrierea unui mic limbaj de programare ajută la finalizarea muncii în lumea reală.

Un asemenea limbaj nu ar trebui să se asemene cu un limbaj de programare tipic. Dacă JavaScript nu ar fi fost echipat cu expresii regulate, de exemplu, ați fi putut scrie propriul parser și evaluator de expresii regulate.

{{index "artificial intelligence"}}

Sau imaginați-vă că aveți de construit un dinozaur robotizat uriaș și trebuie să îi programați comportamentul. JavaScript ar putea să nu fie cel mai eficient mod de a rezolva problema. Ați putea opta pentru un limbaj de programare care ar arăta astfel:

```{lang: null}
behavior walk
  perform when
    destination ahead
  actions
    move left-foot
    move right-foot

behavior attack
  perform when
    Godzilla in-view
  actions
    fire laser-eyes
    launch arm-rockets
```

{{index expressivity}}

Denumirea consacrată pentru un astfel de limbaj este _limbaj de programare specific domeniului (domain-specific language)_, un limbaj adecvat pentru a descrie un domeniu restrâns al cunoașterii. Un asemenea limbaj poate fi mult mai expresiv decât un limbaj cu aplicare generală deoarece este conceput să descrie exact ceea ce trebuie descris în domeniul său și nimic altceva.

## Exerciții

### Array-uri

{{index "Egg language", "arrays in egg (exercise)", [array, "in Egg"]}}

Adăugați suport pentru array-uri în Egg prin adăugarea următoarelor trei funcții în domeniul de vizibilitate principal: `array(...values)` pentru a construi un array ce conține valorile argumentului, `length(array)` pentru a determina lungimea unui array, și `element(array, n)` pentru a obține cel de-al n-lea element dintr-un array.

{{if interactive

```{test: no}
// Modify these definitions...

topScope.array = "...";

topScope.length = "...";

topScope.element = "...";

run(`
do(define(sum, fun(array,
     do(define(i, 0),
        define(sum, 0),
        while(<(i, length(array)),
          do(define(sum, +(sum, element(array, i))),
             define(i, +(i, 1)))),
        sum))),
   print(sum(array(1, 2, 3))))
`);
// → 6
```

if}}

{{hint

{{index "arrays in egg (exercise)"}}

Cel mai simplu mod de a soluționa problema este să reprezentăm array-urile Egg ca și array-uri JavaScript.

{{index "slice method"}}

Valorile adăugate în domeniul de vizibilitate principal trebuie să fie funcții. Utilizând un argument rest (notația cu trei puncte `...`), definția unui `array` poate fi _foarte_ simplă.

hint}}

### Closure

{{index closure, [function, scope], "closure in egg (exercise)"}}

Modul în care am definit `fun` permite funcțiilor din Egg să refere domeniul de vizibilitate înconjurător, permițănd corpului funcției să utilizeze valori locale care erau vizibile atunci când funcția a fost definită, așa cum se întâmplă și cu funcțiile JavaScript.

Următorul program ilustrează aceasta: funcția `f` returnează o funcție care își adună argumentul la argumentul lui `f`, ceea ce înseamnă că trebuie să acceseze domeniul local din `f` pentru a putea utiliza bindingul `a`.

```
run(`
do(define(f, fun(a, fun(b, +(a, b)))),
   print(f(4)(5)))
`);
// → 9
```

Reveniți la definiția pentru forma `fun` și explicați mecanismul care permite funcționarea codului de mai sus.

{{hint

{{index closure, "closure in egg (exercise)"}}

Din nou, ne vom baza pe un mecanism JavaScript pentru a implementa funcționalitatea echivalentă în Egg. Formelor speciale le este transmis domeniul de vizibilitate local în care acestea sunt evaluate astfel că ele își vor putea evalua subformele în acel domeniu de vizibilitate. Funcția returnată de către `fun` are acces la argumentul `scope` transmis funcției care o include și utilizează acest obiect pentru a crea domeniul de vizibilitate local al funcției atunci când este apelată.

{{index compilation}}

Aceasta înseamnă că prototipul domeniului de vizibilitate local este domeniul de vizibilitate în interiorul căruia a fost creată funcția, ceea ce permite accesarea bindingurilor din acel domeniu de vizibilitate din funcție. Cam de atât aveți nevoie pentru a implementa un closure (deși compilarea sa intr-un mod care este de fapt eficient necesită mai multă muncă).

hint}}

### Comentarii

{{index "hash character", "Egg language", "comments in egg (exercise)"}}

Ar fi util dacă am putea scrie comentarii în Egg. De exemplu, un simbol `#` să ne permită să tratăm restul liniei ca fiind un comentariu pe care să îl putem ignora în execuție, simmilar simbolului `//` din JavaScript.

{{index "skipSpace function"}}

Nu avem de făcut nici o modificare majoră în parser pentru a suporta aceasta. Putem doar să modificăm `skipSpace` ca să tratăm comentariile ca și cum ar fi spații albe astfel încât, oriunde se apelează `skipSpace` să ignorăm și comentariile. Realizați această modificare.

{{if interactive

```{test: no}
// This is the old skipSpace. Modify it...
function skipSpace(string) {
  let first = string.search(/\S/);
  if (first == -1) return "";
  return string.slice(first);
}

console.log(parse("# hello\nx"));
// → {type: "word", name: "x"}

console.log(parse("a # one\n   # two\n()"));
// → {type: "apply",
//    operator: {type: "word", name: "a"},
//    args: []}
```
if}}

{{hint

{{index "comments in egg (exercise)", [whitespace, syntax]}}

Asigurați-vă că soluția voastră tratează cazul comentariilor multiple pe un singur rând, cu spații între sau după ele.

O expresie regulată este probabil cel mai ușor mod de rezolvare. Scrieți o expresie care potrivește "spațiu alb sau comentariu, de zero sau mai multe ori". Utilizați metoda `exec` sau `match` și verificați lungimea primului element din array-ul returnat (potrivirea completă) pentru a determina câte caractere să eliminați.

hint}}

### Fixarea domeniului de vizibilitate

{{index [binding, definition], assignment, "fixing scope (exercise)"}}

Deocamdată, singurul mod de a atribui o valoare unui binding este `define`. Această construcție acționează atât ca un mod de a defini noi bindinguri cât și de a atribui bindingurilor existente valori noi.

{{index "local binding"}}

Această ambiguitate cauzează o problemă. Când încercați să atribuiți unui binding non-local o nouă valoare, veți defini de fapt un binding local cu același nume. Unele limbaje de programare sunt concepute să funcționeze astfel, dar cred că este o modalitate ciudată de a gestiona vizibilitatea.

{{index "ReferenceError type"}}

Adăugați o formă specială `set`, similară cu `define`, care atribuie unui binding o valoare nouă, actualizând bindingul dintr-un domeniu de vizibilitate extern dacă acesta nu există deja în domeniul de vizibilitate curent. Dacă bindingul nu este definit deloc, aruncați o excepție `ReferenceError` (un alt tip standard de eroare).

{{index "hasOwnProperty method", prototype, "getPrototypeOf function"}}

Tehnica reprezentării domeniilor de vizibilitate ca și obiecte, care a fost convenabilă până acum, o să vă încurce puțin. Ați putea încerca să utilizați funcția `Object.getPrototypeOf`, care returnează prototipul unui obiect. De asemenea, domeniile de vizibilitate nu derivă din `Object.prototype`, astfel că, dacă vă este util să apelați `hasOwnProperty` asupra lor, va trebui să utilizați această expresie stângace:

```{test: no}
Object.prototype.hasOwnProperty.call(scope, name);
```

{{if interactive

```{test: no}
specialForms.set = (args, scope) => {
  // Your code here.
};

run(`
do(define(x, 4),
   define(setx, fun(val, set(x, val))),
   setx(50),
   print(x))
`);
// → 50
run(`set(quux, true)`);
// → Some kind of ReferenceError
```
if}}

{{hint

{{index [binding, "compilation of"], assignment, "getPrototypeOf function", "hasOwnProperty method", "fixing scope (exercise)"}}

Va trebui să ciclați printre domeniile de vizibilitate, pe rând, utilizând `Object.getPrototypeOf` pentru a trece la domeniul de vizibilitate exterior. Pentru fiecare domeniu de vizibilitate, utilizați `hasOwnProperty` pentru a verifica dacă bindingul indicat de proprietatea `name`, primul argument al `set`, există în acel domeniu de vizibilitate. Dacă da, îl setați la rezultatul evaluării celui de-al doilea argument al `set` și returnați acea valoare.

{{index "global scope", "run-time error"}}

Dacă s-a atins domeniul de vizibilitate cel mai exterior (`Object.getPrototypeOf` returnează `null`) și încă nu am găsit bindingul, atunci vom arunca o excepție.

hint}}
