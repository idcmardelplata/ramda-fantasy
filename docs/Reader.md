# Reader
El tipo `Reader` es utilizado para proveer un "entorno compartido" para los calculos.
A menudo se utiliza como una forma de proporcionar configuración o inyectar dependencias, donde la responsabilidad de estas preocupaciones se delegan a los bordes exteriores de la aplicación. Se puede pensar en esto como si fuera una funcion que tiene acceso a los argumentos que le son proporcionados por el llamador de la funcion. De echo, el tipo `Reader` es efectivamente un tipo de envoltorio alrededor de una funcion para proporcionar las diversas instancias de monada y functor.

## Construcción

El tipo `Reader` puede ser construido a través de una simple funcion constructora `Reader r a :: (r -> a) -> Reader r a` La función dada al constructor es responsable de calcular un valor basado en el entorno que recibe.

```js
const dbReader = Reader(env => env.dbConn);
```
Una instancia estática `Reader` también está disponible a través de Reader.ask que pasa su entorno directamente, proporcionando acceso al entorno dentro de las funciones a su metodo `chain`.

```js
const dbReader = Reader.ask.chain(env => Reader.of(configureDb(env.dbConn)));
```
Para proporcionar un mejor soporte para los tipos de mónadas anidadas como `Reader r (m a)` donde `m` es algún tipo de mónada, se puede construir un transformador de mónada haciendo pasar el tipo de monada anidado al constructor del transformador.


```js
// Reader.T :: Monad m => { of: a -> m a } -> ReaderT r m a
const ReaderTMaybe = Reader.T(Maybe);
```
Esto conecta los diversos métodos disponibles en una mónada (`of`, `map`, `ap` y `chain`) desde la instancia externa de `ReaderT` al tipo de mónada interna, eliminando la necesidad de "desempaquetar" cada capa de instancias de mónada en las que interactúa la instancia de transformador.

El método estático `lift` en un tipo de `ReaderT` permite que una instancia del tipo de mónada interna se "levante" en una instancia de `ReaderT`, similar a cómo se levanta un valor ordinario en una instancia de un functor aplicativo.

```js
const maybe42R = ReaderTMaybe.lift(Just(42)); 
```

## Interacción

El código que requiere acceso al entorno debe existir dentro de un tipo `Reader`. Para asegurarse de que el código sigue siendo componible, `Reader` implementa la especificación de mónada y por extensión tambien la especificacion de `applicative` y `functor`. Esto permite que varias instancias de `Reader` estén compuestas entre sí, mientras que también puede elevar funciones llanas para funcionar dentro del contexto del tipo `Reader`.

```js
/**
 * Env        :: { dbConn: DbConnection, httpClientPool: HttpClientPool }
 * selectOne  :: String -> Table -> Object -> DbConnection -> Future Error DbRow
 * fetchImage :: URL -> HttpClientPool -> Future Error Image
 */

//:: String -> Reader Env (Future Error DbRow)
fetchUser = email => Reader(env =>
  selectOne('email = :email', Users, { email: email }, env.dbConn));

//:: String -> Reader Env (Future Error ImageData)
fetchUserImage = email => Reader.ask.chain(env =>
  fetchUser(email).map(futureUserRow => futureUserRow.chain(userRow =>
    fetchImage(userRow.imageURL, env.httpClientPool))));

//:: Future Error ImageData
fooImageFuture = fetchUserImage('foo@example.com').run({
  dbConn: initDb(),
  httpClientPool: initHttpClientPool
});
```

Alternativamente, un `ReaderT` podría ser utilizado para ayudar a reducir algunas de las llamadas anidadas de `chain` y `map` en la función `fetchUserImage` en el ejemplo anterior.

```js
//:: (Env -> Future e a) -> ReaderT Env (Future e) a
App = Reader.T(Future);

//:: String -> ReaderT Env (Future Error) DbRow
fetchUser = email => App(env =>
  selectOne('email = :email', Users, { email: email }, env.dbConn));

//:: String -> ReaderT Env (Future Error) ImageData
fetchUserImage = email => App.ask.chain(env =>
  fetchUser(email).chain(userRow =>
    App.lift(fetchImage(userRow.imageURL, env.httpClientPool))));
```

## Referencia

### Constructores

#### `Reader`
```hs
:: (r -> a) -> Reader r a
```
Construye una instancia de `Reader` que calcula un valor `a` usando un entorno `r`.

#### `Reader.T`
```hs
:: Monad m => { of: a -> m a } -> (r -> m a) -> ReaderT r m a
```
Construye una instancia de `ReaderT` que calcula un valor `a` dentro de una monada `m` usando algún entorno `r`.

### Propiedades estaticas

#### `Reader.ask`
```hs
:: Reader r r
```
Una instancia estatica de `Reader` que simplemente devuelve su entorno.

#### `ReaderT.ask`
```hs
:: Monad m => ReaderT r m a
```
Una instancia estática de `ReaderT` que simplemente devuelve su entorno.


### Funciones estaticas

#### `Reader.of`
```hs
:: a -> Reader r a
```
Produce una instancia pura de `Reader` para el valor dado.

#### `ReaderT.of`
```hs
:: Monad m => a -> ReaderT r m a
```
Produce una instancia pura de `ReaderT` para el valor dado.


#### `ReaderT.lift`
```hs
:: Monad m => m a -> ReaderT r m a
```

Eleva un valor de monada dentro de una instancia de `ReaderT`

### Metodos de instancia

#### `reader.run`
```hs
:: Reader r a ~> r -> a
```
Ejecuta la instancia de `Reader` con el entorno dado `r`.


#### `readerT.run`
```hs
:: Monad m => ReaderT r m a ~> r -> m a
```
Ejecuta la instancia de `ReaderT` con el entorno dado `r`.

#### `reader.map`
```hs
:: Reader r a ~> (a -> b) -> Reader r b
```
Transforma el resultado del cálculo de la instancia `Reader` con la función dada.

#### `readerT.map`
```hs
:: Monad m => ReaderT r m a ~> (a -> b) -> ReaderT r m b
```
Transforma el resultado del cálculo de la instancia de `ReaderT` con la función dada.

#### `reader.ap`
```hs
:: Reader r (a -> b) ~> Reader r a -> Reader r b
```
Aplica el `a` en la instancia de `Reader` a la función en esta instancia de `Reader`, produciendo una nueva instancia de `Reader` que contiene el resultado.


#### `readerT.ap`
```hs
:: Monad m => ReaderT r m (a -> b) ~> ReaderT r m a -> ReaderT r m b
```
Aplica el `a` en la instancia de `ReaderT` a la función en esta instancia de `ReaderT`, produciendo una nueva instancia de `ReaderT` que contiene el resultado.


#### `reader.chain`
```hs
:: Reader r a ~> (a -> Reader r b) -> Reader r b
```
Produce una nueva instancia de `Reader` aplicando la función proporcionada con el valor de esta instancia de Reader. Tanto esta instancia como la instancia devuelta por la función proporcionada recibirán el mismo entorno cuando se ejecute.

#### `readerT.chain`
```hs
:: Monad m => ReaderT r m a ~> (a -> ReaderT r m b) -> ReaderT r m b
```
Produce una nueva instancia de `ReaderT` aplicando la función proporcionada con el valor de esta instancia de ReaderT. Tanto esta instancia como la instancia devuelta por la función proporcionada recibirán el mismo entorno cuando se ejecute.

