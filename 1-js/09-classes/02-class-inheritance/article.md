# Ereditarietà delle classi

Immaginiamo di avere due classi.

`Animal`:

```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stands still.`);
  }
}

let animal = new Animal("My animal");
```

![rabbit-animal-indipendent-animal](rabbit-animal-independent-animal.svg)

...E `Rabbit`:

```js
class Rabbit {
  constructor(name) {
    this.name = name;
  }
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("My rabbit");
```

![rabbit-animal-indipendent-rabbit](rabbit-animal-independent-rabbit.svg)

Al momento sono entrambi completamente indipendenti.

Ora invece vorremmo che `Rabbit` estendesse `Animal`. In altre parole, i conigli (rabbits) dovrebbero essere derivati dagli animali (animals), avere accesso ai metodi di `Animal` ed estendere la classe con i loro metodi.

Per ereditare da un'altra classe è necessario scrivere `"extends"` e la classe padre prima delle parentesi graffe `{..}`.

Qui `Rabbit` eredita da `Animal`:

```js run
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stands still.`);
  }
}

// Eredita da Animal specificando "extends Animal"
*!*
class Rabbit extends Animal {
*/!*
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("White Rabbit");

rabbit.run(5); // White Rabbit runs with speed 5.
rabbit.hide(); // White Rabbit hides!
```

Ora il codice di `Rabbit` è diventato un po' più corto, dato che utilizza il costruttore di `Animal`, e può anche correre (usare il metodo `run`) come gli animali (animals).

Internamente, `extends` aggiunge da `Rabbit.prototype` un riferimento `[[Prototype]]` a `Animal.prototype`:

![animal-rabbit-extends](animal-rabbit-extends.svg)

Dunque, se un elemento non viene trovato all'interno di `Rabbit.prototype`, JavaScript lo cerca in `Animal.prototype`.

Come già detto nel capitolo <info:native-prototypes>, JavaScript usa la stessa ereditarietà del prototipo per gli oggetti base (build-in objects). Per esempio `Date.prototype.[[Prototype]]` corrisponde a `Object.prototype`, quindi le date possono usufruire dei metodi di un oggetto generico.

````smart header="Qualsiasi espressione è ammessa dopo `extend`"
Usare la parola chiave `class` permette di specificare non solo una classe, ma anche un'espressione dopo la parola `extends`.

Per esempio, una chiamata ad una funzione che genera la classe padre:

``js run
function f(phrase) {
  return class {
    sayHi() { alert(phrase) }
  }
}

*!*
class User extends f("Hello") {}
*/!*

new User().sayHi(); // Hello
````

In questo codice la `class User` eredita dal risultato della funzione `f("Hello")`.

Questa particolarità può tornare utile nella programmazione avanzata, quando abbiamo bisogno di generare delle classi padre a seconda di vari parametri.
``

## Sovrascrivere un metodo

Proseguiamo ora e vediamo come sovrascrivere un metodo. Al momento, `Rabbit` eredita il metodo `stop` dalla classe `Animal`, il quale imposta `this.speed` a `0`.

Se specifichiamo il nostro metodo `stop` in `Rabbit`, esso verrà scelto al posto del metodo ereditato dal padre:

```js
class Rabbit extends Animal {
  stop() {
    // ...questo verrà utilizzato per rabbit.stop()
  }
}
```

...Normalmente però non vogliamo rimpiazzare completamente il metodo ereditato, ma piuttosto costruire su esso, modificarlo leggermente o estendere le sue funzionalità. Nel nostro metodo compiamo delle azioni, ma ad un certo punto richiamiamo il metodo ereditato.

Le classi forniscono la parola chiave `"super"` per questo scopo.

- `super.method(...)` per richiamare un metodo dal padre;
- `super(...)` per richiamare il costruttore del padre (valido solo all'interno del nostro costruttore).

Per esempio, facciamo sì che il nostro coniglio si nasconda automaticamente quando si ferma:

```js run
class Animal {

  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  run(speed) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }

  stop() {
    this.speed = 0;
    alert(`${this.name} stands still.`);
  }

}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }

*!*
  stop() {
    super.stop(); // richiama il metodo stop() dal padre
    this.hide(); // and then hide
  }
*/!*
}

let rabbit = new Rabbit("White Rabbit");

rabbit.run(5); // White Rabbit runs with speed 5.
rabbit.stop(); // White Rabbit stands still. White rabbit hides!
```

Ora `Rabbit` contiene il metodo `stop`, che richiama al suo interno il metodo `super.stop()`.

````smart header="Le funzioni a freccia (Arrow functions) non hanno `super`"
Come accennato nel capitolo <info:arrow-functions>, all'interno delle funzioni a freccia (arrow functions)non si può utilizzare la parola `super` 

Se acceduto, esso viene preso dalla funzione esterna. Per esempio:
```js
class Rabbit extends Animal {
  stop() {
    setTimeout(() => super.stop(), 1000); // richiama il metodo stop dal padre dopo 1 secondo
  }
}
```

Il `super` nella funzione a freccia (arrow function) è lo stesso di `stop()`, quindi funziona come dovrebbe. Se specificassimo una funzione "regolare" (regular) otterremmo un errore:

```js
// Unexpected super
setTimeout(function() { super.stop() }, 1000);
```
````

## Sovrascrivere il costruttore

Sovrascrivere un costruttore è leggermente più complicato.

Finora, `Rabbit` non ha avuto il suo metodo`constructor`.

Secondo le [specifiche](https://tc39.github.io/ecma262/#sec-runtime-semantics-classdefinitionevaluation), se una classe ne estende un'altra e non ha un suo metodo `constructor` viene generato il seguente `constructor` "vuoto":

```js
class Rabbit extends Animal {
  // generato per classi figlie senza un costruttore proprio
*!*
  constructor(...args) {
    super(...args);
  }
*/!*
}
```

Come possiamo vedere, esso richiama il `constructor` del padre, passandogli tutti gli argomenti. Questo accade se non creiamo un costruttore ad hoc.

Aggiungiamo quindi un `constructor` personalizzato per `Rabbit`, che specificherà, oltre al `name`, anche la proprietà `earLength`:

```js run
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  // ...
}

class Rabbit extends Animal {

*!*
  constructor(name, earLength) {
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
*/!*

  // ...
}

*!*
// Non funziona!
let rabbit = new Rabbit("White Rabbit", 10); // Error: this is not defined. (Errore: "this" non è definito)
*/!*
```

Ops! Abbiamo ricevuto un errore. Ora non possiamo creare conigli (rabbits). Cosa è andato storto?

La risposta breve è: i costruttori delle classi figlie devono richiamare `super(...)` e (!) farlo prima di usare `this`.

...Ma perchè? Cosa sta succedendo?
In effetti, questa richiesta sembra un po' strana.

Ovviamente una spiegazione c'è. Addentriamoci nei dettagli, così da capire cosa effettivamente succede.

In JavaScript vi è una netta distinzione tra il "metodo costruttore di una classe figlia" e tutte le altre. In una classe figlia, il costruttore viene etichettato con una proprietà interna speciale: `[[ConstructorKind]]:"derived"`.

La differenza è:

- Quando viene eseguito un costruttore normale, esso crea un oggetto vuoto chiamato `this` e continua a lavorare su quello. Questo non avviene quando il costruttore di una classe figlia viene eseguito, dato che si aspetta che il costruttore del padre lo faccia per lui.

Se stiamo creando il costruttore di un figlio dobbiamo per forza richiamare `super`, altrimenti l'oggetto referenziato da `this` non verrebbe creato. E riceveremmo un errore.

Per far funzionare `Rabbit` dobbiamo richiamare `super()` prima di usare `this`:

```js run
class Animal {

  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  // ...
}

class Rabbit extends Animal {

  constructor(name, earLength) {
*!*
    super(name);
*/!*
    this.earLength = earLength;
  }

  // ...
}

*!*
// finalmente
let rabbit = new Rabbit("White Rabbit", 10);
alert(rabbit.name); // White Rabbit
alert(rabbit.earLength); // 10
*/!*
```

## Super: meccanismi interni, [[HomeObject]]

Andiamo più a fondo all'interno di `super`.

In primis, da quel che abbiamo imparato finora, è impossibile che `super` funzioni!

Beh, proviamo a chiederci, come può funzionare? Quando un metodo viene eseguito, il suo oggetto di appartenenza viene indicato con `this`. Se richiamiamo `super.method()`, dunque, esso dovrà recuperare il metodo dal prototipo dell'oggetto corrente. 

Questa attività può sembrare semplice, ma non lo è. Il motore (engine) conosce l'oggetto `this`, quindi potrebbe ottenere il metodo dalla classe padre attraverso `this.__proto__.method`. Sfortunatamente, una soluzione così "naif" non funzionerà.

Dimostriamo il problema, usando per semplicità degli oggetti piani (plain objects).

Nell'esempio sottostante, `rabbit.__proto__ = animal`. Ora proviamo: in `rabbit.eat()` richiamiamo `animal.eat()` attraverso `this.__proto__`:

```js run
let animal = {
  name: "Animal",
  eat() {
    alert(`${this.name} eats.`);
  }
};

let rabbit = {
  __proto__: animal,
  name: "Rabbit",
  eat() {
*!*
    // super.eat() dovrebbe funzionare presumibilmente così
    this.__proto__.eat.call(this); // (*)
*/!*
  }
};

rabbit.eat(); // Rabbit eats.
```

Alla linea `(*)` prendiamo `eat` dal prototipo (`animal`) e lo richiamiamo all'interno dell'oggetto. Nota che `.call(this)` è importante, dato che `this.__proto__.eat()` richiamerebbe il metodo `eat` nel contesto della classe padre, non nella classe figlio.

Nell'esempio precedente in effetti il metodo funzionava a dovere: abbiamo ricevuto l'`alert` corretto.

Ora proviamo ad aggiungere un altro oggetto. Vedremo cosa non va:

```js run
let animal = {
  name: "Animal",
  eat() {
    alert(`${this.name} eats.`);
  }
};

let rabbit = {
  __proto__: animal,
  eat() {
    // ...salta in giro come un coniglio e richiama il metodo dalla classe padre (animal)
    this.__proto__.eat.call(this); // (*)
  }
};

let longEar = {
  __proto__: rabbit,
  eat() {
    // ...fa qualcosa con longEar e richiama il metodo dalla classe padre (rabbit)
    this.__proto__.eat.call(this); // (**)
  }
};

*!*
longEar.eat(); // Error: Maximum call stack size exceeded (Errore: limite massimo di chiamate allo stack superato)
*/!*
```

Il codice non funziona più! Possiamo vedere l'errore provando a richiamare `longEar.eat()`.

Potrebbe non essere così scontato, ma se tracciamo la chiamata di `longEar.eat()` possiamo capire perché ciò accade. Nelle linee `(*)`
 e `(**)` il valore di `this` è l'oggetto corrente (`longEar`). Questo è fondamentale: tutti i metodi di un oggetto ricevono l'oggetto corrente come `this`, non attraverso un prototipo o simili.

 Quindi, sia nella linea `(+)` che nella linea `(**)` il valore di `this.__proto__` è esattamente lo stesso: `rabbit`. Entrambi richiamano `rabbit.eat` senza salire la catena, generando un ciclo (loop) infinito.

 Questa immagine rappresenta ciò che accade: 

 ![this-super-loop.svg](this-super-loop.svg)

 1. Dentro a `longEar.eat()`, la linea `(**)` richiama `rabbit.eat`assieme a `this=longEar`.
  
  ```js
    // dentro a longEar.eat() abbiamo this = longEar
    this.__proto__.eat.call(this) // (**)
    // diventa
    longEar.__proto__.eat.call(this)
    // che è uguale a
    rabbit.eat.call(this);
    ```  

2. Poi nella linea `(*)` di `rabbit.eat` vorremo passare la chiamata ancora più in alto nella catena, ma `this=longEar`, dunque `this.__proto__.eat` è ancora `rabbit.eat`!

    ```js
    // dentro a rabbit.eat() abbiamo ancora this = longEar
    this.__proto__.eat.call(this) // (*)
    // diventa
    longEar.__proto__.eat.call(this)
    // oppure (nuovamente)
    rabbit.eat.call(this);
    ```

3. ...Dunque `rabbit.eat` richiama sé stesso in un ciclo (loop) infinito, perché non può più salire.

Il problema non può essere risolto utilizzando solo `this`.

### `[[HomeObject]]`

Per dare una soluzione, JavaScript ha un'altra speciale proprietà interna per le funzioni: `[[HomeObject]]`.

Quando una funzione appartiene ad una classe o ad un metodo, la sua proprietà `[[HomeObject]]` diventa l'oggetto.

Quindi viene utilizzata da `super` per capire il prototipo del padre e i suoi metodi.

Vediamo come funziona:

```js run
let animal = {
  name: "Animal",
  eat() {         // animal.eat.[[HomeObject]] == animal
    alert(`${this.name} eats.`);
  }
};

let rabbit = {
  __proto__: animal,
  name: "Rabbit",
  eat() {         // rabbit.eat.[[HomeObject]] == rabbit
    super.eat();
  }
};

let longEar = {
  __proto__: rabbit,
  name: "Long Ear",
  eat() {         // longEar.eat.[[HomeObject]] == longEar
    super.eat();
  }
};

*!*
// funziona correttamente
longEar.eat();  // Long Ear eats.
*/!*
```

Funziona come dovrebbe, grazie alle meccaniche di `[[HomeObject]]`. Un metodo, per esempio `longEar.eat`, conosce il suo `[[HomeObject]]` e prende il metodo della classe padre da quel prototipo, senza utilizzare `this`.

### I metodi non sono "liberi"

Come abbiamo già visto, generalmente le funzioni sono libere, ovvero non sono legate ad un oggetto in JavaScript, così da poter essere copiate tra gli oggetti ed essere richiamate con un altro `this`.

L'esistenza di `[[HomeObject]]` viola questo principio, perché i metodi "ricordano" i loro oggetti. `[[HomeObject]]` non può essere modificato, quindi questo legame dura per sempre.

L'unico posto in cui `[[HomeObject]]` viene utilizzato è in `super`. Quindi, se un metodo non utilizza `super` è ancora libero e copiabile. Ma con `super` le cose potrebbero andar male.

Qui di seguito è rappresentato un utilizzo sbagliato di `super`:

```js run
let animal = {
  sayHi() {
    console.log(`I'm an animal`);
  }
};

// rabbit inherits from animal
let rabbit = {
  __proto__: animal,
  sayHi() {
    super.sayHi();
  }
};

let plant = {
  sayHi() {
    console.log("I'm a plant");
  }
};

// tree inherits from plant
let tree = {
  __proto__: plant,
*!*
  sayHi: rabbit.sayHi // (*)
*/!*
};

*!*
tree.sayHi();  // I'm an animal (?!?)
*/!*
```

Una chiamata a `tree.sayHi()` mostra "I'm an animal". Completamente sbagliato.

La ragione è semplice:

- Nella linea `(*)`, il metodo `tree.sayHi` viene copiato da `rabbit`. Forse volevamo evitare doppioni nel codice?

- Quindi il suo `[[HomeObject]]` è `rabbit`, dato che è stato creato in `rabbit`. Non c'è modo di cambiare `[[HomeObject]]`;

- Il codice di `tree.sayHi()` contiene `super.sayHi()`, che va fino a `rabbit` e prende il metodo da `animal`.

![super-homeobject-wrong](super-homeobject-wrong.svg)

### Metodi, non proprietà di una funzione

`[[HomeObject]]` viene definito per metodi appartenenti a classi e ad oggetti piani (plain objects), ma per gli oggetti i metodi vanno definiti come `method()`, non `"method: function()"`.

La differenza potrebbe non essere rilevante per noi, ma lo è per JavaScript.

Nel prossimo esempio, viene utilizzata una sintassi errata (non-method syntax) per fare un confronto. La proprietà `[[HomeObject]]`non viene impostata e l'ereditarietà non funziona: 

```js run
let animal = {
  eat: function() { // dovrebbe corrispondere a eat(){...}
    // ...
  }
};

let rabbit = {
  __proto__: animal,
  eat: function() {
    super.eat();
  }
};

*!*
rabbit.eat();  // Errore nella chiamata a super (dato che [[HomeObject non esiste]])
*/!*
```

## Summary

1. Per estendere una classe: `class Child extends Parent`:

    - Questo significa che `Child.prototype.__proto__` dventerà `Parent.prototype`, quindi i metodi vengono ereditati.

2. Quando sovrascriviamo un costruttore:

    - Dobbiamo richiamare il costruttore del padre attraverso `super()` nel costruttore di `Child` prima di utilizzare `this`.

3. Quando sovrascriviamo un metodo:

    - Possiamo usare `super.method()` in un metodo di `Child` per richiamare il metodo da `Parent`.

4. Meccaniscmi interni:

    - I metodi tengono traccia del loro oggetto o della loro classe nella proprietà `[[HomeObject]]`, così da poter utilizzare `super` per accedere ai metodi della classe padre.

    - Non è quindi sicuro copiare un metodo in un altro oggetto attraverso `super`.

Inoltre:

- Le funzioni a freccia (arrow functions) non hanno un loro `this` o `super`, dunque si adattano al contesto in cui si trovano.
