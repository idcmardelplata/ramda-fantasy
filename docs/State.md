# Estado

El tipo `State` se puede utilizar para almacenar un estado a lo largo de una computación.

## Construccion

Las instancias de `State` deben obtenerse a través de las propiedades estáticas `get`, `gets`, `put` y `modify` del objeto `State`. Estas instancias describen las diferentes maneras de acceder y modificar la computación con estado que eventualmente evaluaremos y describiremos con más detalle a continuación.

Un transformador de `State` tambien esta disponible via `State.T`. Que se puede utilizar para extender alguna mónada con comportamiento con estado.

## Interacción

Las instancias de `State` interactuan principalmente con composiciones a trabes del metodo chain de las diversas instancias estaticas disponibles en el objeto de tipo `State`. Para acceder al estado actual, `State.get` es una instancias estatica que se puede utilizar para proporcionar el estado a `chain`, `ap` y `map`. De forma similar, `State.gets(fn)`,nos proporciona el estado transformado por la función fn asignada.
Para cambiar el estado de la computación, `State.put(newValue)` se puede utilizar para reemplazar el estado existente con el nuevo valor `newValue`.
El estado actual tambien puede ser transformado proporcionando una función de transformación a `State.modify(transformFn)`.

Una vez que una instancia de `State` es definida, una semilla inicial se puede proporcionar a cualquiera de los metodos `eval` o `exec` para evaluar la computación y retornar su resultado o el estado final, respectivamente. Alternativamente, el metodo `run` puede ser llamado para retornar ambos, el resultado y el estado final, en una instancia de `Tuple`

```js
// An example deterministic pseudorandom number generator
// see: https://en.wikipedia.org/wiki/Linear_congruential_generator

// type RNG a = State Integer a

// rng :: RNG Float
const rng = State.get.chain(seed => {
  const newSeed = (seed * 1103515245 + 12345) & 0x7FFFFFFF;
  const randVal = (newSeed >>> 16) / 0x7FFF;
  return State.put(newSeed).map(_ => randVal);
});

rng.eval(42); // From initial seed of 42: 0.5823236793115024
rng.eval(42); // Repeating produces the same value: 0.5823236793115024
rng.exec(42); // `exec` returns the next seed: 1250496027
rng.eval(1250496027); // A different seed: 0.5198217719046602

// Chain together to pass the new seed to the next RNG
// pair :: RNG a -> RNG (Tuple a a)
const pair = rng => rng.chain(a => rng.chain(b => State.of(Tuple(a, b))));

pair(rng).eval(42); // Tuple(0.5823236793115024, 0.5198217719046602)

// Map to produce transformed random values from 1 to 6
// rollDie :: RNG Integer
const rollDie = rng.map(n => Math.ceil(n * 6));

// rollDice :: RNG (Tuple Integer Integer)
const rollDice = pair(rollDie);
rollDice.eval(123); // Tuple(2, 5)
```

## Referencia

### Constructores

#### `State`
```hs
:: (s -> Identity (Tuple a s)) -> State s a
```
Construye una instancia de `State` que representa una computación pura a partir de un estado a otro estado y resultado. Tenga en cuenta que este constructor requiere que la función dada retorne una instancia de Identity. En general, se recomienda el uso de las propiedades estaticas y metodos proporcionados en el objeto `State` en lugar de utilizar este constructor.

#### `State.T`
```hs
:: Monad m => { of :: a -> m a } -> (s -> m (Tuple a s)) -> StateT s m a
```
Construye una instancia de `State.T` que representa una computación de algún estado a un nuevo estado y resultado en el contexto de alguna otra monada. En general se recomienda el uso de las propiedades estaticas y metodos proporcionados en el objeto `State` en lugar de utilizar este constructor.

### Propiedades estaticas

#### `State.get`
```hs
:: State s s
```
Una instancia estatica de `State` que retorna el estado actual.

#### `StateT.get`
```hs
:: Monad m => StateT s m s
```
Una instancia estatica de `StateT` que retorna el estado actual.

### Metodos estaticos

#### `State.gets`
```hs
:: (s -> a) -> State s a
```
Devuelve una instancia `State` que recupera el estado actual transformado por la función dada.

#### `StateT.gets`
```hs
:: Monad m => (s -> a) -> StateT s m a
```
Devuelve una instancia `State` que recupera el estado actual transformado por la función dada.

#### `State.put`
```hs
:: s -> State s a
```
Retorna una instancia de `State` que almacena el estado proporcionado.

#### `StateT.put`
```hs
:: Monad m => s -> StateT s m a
```
Retorna una instancia de `StateT` que almacena el estado proporcionado.

#### `State.modify`
```hs
:: (s -> s) -> State s a
```
Retorna una instancia de `State` que modifica el estado almacenado con la funcion proporcionada.

#### `StateT.modify`
```hs
:: Monad m => (s -> s) -> StateT s m a
```
Retorna una instancia de `StateT` que modifica el estado almacenado con la funcion proporcionada.

#### `State.of`
```hs
:: a -> State s a
```
Retorna una instancia de `State` que evaluara el valor proporcionado.

#### `StateT.of`
```hs
:: Monad m => a -> StateT s m a
```
Retorna una instancia de `StateT` que evaluara el valor proporcionado.

#### `StateT.lift`
```hs
:: Moand m => m a -> StateT s m a
```
Eleva la monada dada en una instancia de `StateT`.

### Métodos de instancia

#### `state.run`
```hs
:: State s a ~> s -> Tuple a s
```
Ejecuta la instancia de `State` que se inicializo con el valor proporcionado y retorna el estado final junto con el resultado en una `Tuple`.

#### `stateT.run`
```hs
:: Monad m => StateT s m a ~> s -> m Tuple(a, s)
```
Ejecuta la instancia de `StateT` inicializada con el valor proporcionado y retorna el estado final junto con el resultado en una `Tuple` dentro del tipo monada subyacente del transformador.

#### `state.eval`
```hs
:: State s a ~> s -> a
```
Ejecuta la instancia `State` inicializada por el valor proporcionado y retorna el resultado.

#### `stateT.eval`
```hs
:: Monad m => StateT s m a ~> s -> m a
```
Ejecuta la instancia `StateT` inicializada por el valor proporcionado y devuelve el resultado en el contexto del tipo monada subyacente del transformador.

#### `state.exec`
```hs
:: State s a ~> s -> s
```
Ejecuta la instancia de `State`, inicializada por el valor proporcionado y retorna el estado final.

#### `stateT.exec`
```hs
:: Monad m => StateT s m a ~> s -> m s
```
Ejecuta la instancia de `StateT`, inicializada por el valor proporcionado y retorna el estado final en el contexto del tipo monada subyacente del transformador.

#### `state.map`
```hs
:: State s a ~> (a -> b) -> State s b
```
Transforma el resultado eventual de la instancia de `State` con la funcion proporcionada.

#### `stateT.map`
```hs
:: Monad m => StateT s m a ~> (a -> b) -> StateT s m b
```
Transforma el resultado eventual de la instancia de `StateT` con la funcion proporcionada 

#### `state.ap`
```hs
:: State s (a -> b) ~> State s a -> State s b
```
Aplica la funcion resultante de esta instancia de `State` al resultado de la instancia `State` provista para producir una nueva instancia de `State`

#### `stateT.ap`
```hs
:: Monad m => StateT s m (a -> b) ~> StateT s m a -> StateT s m b
```
Aplica la funcion resultante de esta instancia de `StateT` al resultado de la instancia `State` provista para producir una nueva instancia de `State`

#### `state.chain`
```hs
:: State s a ~> (a -> State s b) -> State s b
```
Crea una nueva instancia de `State` mediante la aplicación de la funcion dada al resultado de esta instancia `State`

#### `stateT.chain`
```hs
:: StateT s m a ~> (a -> StateT s m b) -> StateT s m b
```
Crea una nueva instancia de `StateT` mediante la aplicación de la funcion dada al resultado de esta instancia `StateT`
