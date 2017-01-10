# Maybe

El tipo `Maybe` representa la posibilidad de algún valor o ninguno. A menudo se utiliza donde `null` tradicionalmente representaría la ausencia de un valor. La ventaja de usar un tipo `Maybe` sobre `null` es que es a la vez componible y requiere que el desarrollador reconozca explícitamente la ausencia potencial de un valor, ayudando a evitar la existencia de excepciones de puntero nulo.  

## Construcción

El tipo `Maybe` consta de dos constructores, `Just :: a -> Maybe a` y `Nothing :: () -> Maybe a`, que representa la existencia y ausencia de algún tipo `a` respectivamente.

```js
const M       = require('ramda-fantasy').Maybe;
const Just    = M.Just;
const Nothing = M.Nothing;

const safeDiv = R.curry((n, d) => d === 0 ? Nothing() : Just(n / d));
const lookup = R.curry((k, obj) => k in obj ? Just(obj[k]) : Nothing());
```

## Interacción

Una vez que se obtiene una instancia de un tipo `Maybe`, hay varias maneras en que se puede acceder al valor contenido si este existe, mientras que nos aseguramos que el caso `Nothing` es manejado.

```js
// getOrElse :: Maybe a ~> a -> a -- provide a default value if Nothing
lookup('foo', { foo: 'bar' }).getOrElse('baz'); // 'bar'
lookup('foo', { abc: 'bar' }).getOrElse('baz'); // 'baz'
```

A menudo es útil transformar el valor potencial de un `Maybe` aplazando la extracción hasta más adelante en el programa. Las dos funciones, `map` y `chain`, existen para soportar este comportamiento. Ambas funciones son un tanto parecidas puesto que tampoco pueden transformar un `Nothing` en un `Just`, aunque `chain` se considera mas potente, ya que permite a una funcion transformar un `Just` en un `Nothing`, mientras que `map` solo puede transformar el valor contenido dentro de un `Just`.

```js
// map :: Maybe a ~> (a -> b) -> Maybe b
safeDiv(42, 2).map(R.inc); // Maybe(22)
safeDiv(42, 0).map(R.inc); // Nothing

// chain :: Maybe a ~> (a -> Maybe b) -> Maybe b
lookup('a', { a: { b: 'foo' }}).chain(lookup('b')); // Just('foo')
lookup('a', { a: {}}).chain(lookup('b'));           // Nothing
lookup('a', {}).chain(lookup('b'));                 // Nothing
```

## Referencia

### Constructores

#### `Maybe.Just`
```hs
:: a -> Maybe a
```
Construye una instancia de `Maybe` que representa la existencia de algún valor.

#### `Maybe.Nothing`
```hs
:: () -> Maybe a
```
Construye una instancia de `Maybe` que representa la ausencia de un valor.

### Funciones estaticas

#### `Maybe.maybe`
```hs
:: b -> (a -> b) -> Maybe a -> b
```
Transforma el valor de `Just` con la función proporcionada, o devuelve el valor predeterminado si se recibe `Nothing`.


#### `Maybe.of`
```hs
:: a -> Maybe a
```
Produce una instancia pura de `Maybe` a partir de un valor dado. Efectivamente, el constructor de `Just`

#### `Maybe.isJust`
```hs
:: Maybe a -> Boolean
```
Retorna `true` si la instancia de `Maybe` es `Just`, caso contrario retorna `false`.

#### `Maybe.isNothing`
```hs
:: Maybe a -> Boolean
```

Retorna `true` si la instancia de `Maybe` es `Nothing`, caso contrario retorna `false`


### Instance methods

#### `maybe.getOrElse`
```hs
:: Maybe a ~> a -> a
```

Retorna el valor si la instancia es `Just`, en caso contrario el valor especificado es retornado.

#### `maybe.map`
```hs
:: Maybe a ~> (a -> b) -> Maybe b
```
Transforma el valor de `Just` con la funcion proporcionada, retornando un nuevo `Just`. Si `Nothing` es recibido, `Nothing` sera retornado.

#### `maybe.ap`
```hs
:: Maybe (a -> b) ~> Maybe a -> Maybe b
```

Aplica la funcion contenida en el primer `Just` al valor del segundo `Just`, retornando un `Just` con el resultado. Si cualquiera de los argumentos es `Nothing`, el resultado sera `Nothing`.

#### `maybe.chain`
```hs
:: Maybe a ~> (a -> Maybe b) -> Maybe b
```
Retorna el resultado de aplicar la funcion proporcionada al valor contenido en la instancia de `Just`. Si la instancia es `Nothing`, entonces `Nothing` es retornado.

#### `maybe.reduce`
```hs
:: Maybe a ~> (b -> a -> b) -> b -> b
```
Retorna el resultado de aplicar la funcion proporcionada al valor inicial y el valor de `Just`. Si la instancia es `Nothing`, entonces el valor inicial se devuelve en su lugar.

#### `maybe.equals`
```hs
:: Maybe a ~> * -> Boolean
```
Retorna `true` si tanto la instancia como el valor proporcionado son `Nothing`. O si la instancia es `Just` y el valor proporcionado es `Just`, donde ambos valores contenidos se consideran iguales como lo determina la funcion `R.equals`. Sino `false` es retornado.

#### `maybe.toString`
```hs
:: Maybe a ~> () -> String
```
Retorna la representacion en string de la instancia
