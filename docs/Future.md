# Future

El tipo `Future` se utiliza para representar alguna acción futura, a menudo asincrona, que potencialmente puede fallar. Es similar al tipo `Promise` nativo de JavaScript, sin embargo la computacion de una promesa es ejecutada inmediatamente, mientras que la ejecución de una instancia de `Future` se retrasa hasta que se solicite explicitamente.

## Construcción

El tipo `Future` consiste en un unico constructor que acepta una funcion que debe aceptar las dos funciones de continuación usadas para resolver o rechazar la instancia de `Future`: `Future :: ((e -> (), a -> ()) -> ()) -> Future e a`.

```js
delayed = (ms, val) => Future((reject, resolve) =>
  ms > 1000 ? reject(Error('Delay too long'))
            : setTimeout(() => resolve(val), ms));
```

## Interacción

Una vez que se ha creado una instancia de `Future`, los diversos metodos adjuntos a la instancia se pueden utilizar para instruir a que se produzcan nuevas transformaciones a la instancia de `Future` despues de que se haya resuelto o rechazado. Es importante tener en cuenta de que nada se ejecuta realmente hasta que el metodo `fork` sea llamado explicitamente.

Las funciones `map`, `ap` y `chain` se pueden utilizar para transformar valores resueltos de una instancia de `Future`, mientras que `chainReject` se puede utilizar para transformar o recuperar un valor rechazado.

```js
//:: String -> Future Error [String]
ls = path => Future((reject, resolve) =>
  fs.readdir(path, (err, files) => err ? reject(err) : resolve(files)));

//:: String -> Future Error String
cat = file => Future((reject, resolve) =>
  fs.readFile(file, 'utf8', (err, data) => err ? reject(err) : resolve(data)));

//:: String -> Future Error String
catDir = dir => ls(dir).chain(R.traverse(Future.of, cat)).map(R.join('\n'));
```

Para ejecutar una instancia de `Future`, el metodo `fork` debe ser llamado con un manejador de funciones `onRejected` y un `onResolved`. Si `fork` es llamado multiples veces, la acción descrita por la instancia de `Future` se invocara multiples veces, si se desea, se puede dar una instancia de `Future` a `Future.cache` que devuelve una nueva instancia que asegura que la acción solo se invocara una vez, con el valor en cache resuelto o rechazado que se utiliza para las llamadas posteriores a `fork`.

```js
catDir('./src').fork(err => console.error(err), data => console.log(data));
```

## Referencia

### Constructores

#### `Future`
```hs
:: ((e -> (), a -> ()) -> ()) -> Future e a
```
Construye una instancia de `Future` que representa alguna accion que tiene posibilidades de fallar.

### Metodos estaticos

#### `Future.of`
```hs
:: a -> Future e a
```
Crea una instancia de `Future` que se resuelve el valor proporcionado.

#### `Future.reject`
```hs
:: e -> Future e a
```
Crea una instancia de `Future` que se rechaza con el valor proporcionado

#### `Future.cache`
```hs
:: Future e a -> Future e a
```
Crea una nueva instancia `Future` de la instancia `Future` proporcionada, donde el valor resuelto o rechazado se almacena en cache para las llamadas posteriores a `fork`.

### Metodos de instancia

#### `future.fork`
```hs
:: Future e a ~> (e -> ()) -> (a -> ()) -> ()
```
Ejecuta las acciones descritas por la instancia de `Future`, llamando al primer argumento con el valor rechazado si se rechaza la instancia o el segundo argumento con el valor resuelto si se resuelve la instancia.

#### `future.map`
```hs
:: Future e a ~> (a -> b) -> Future e b
```
Transforma el valor resuelto de la instancia de `Future` con la funcion proporcionada. Si la instancia es rechazada la funcion proporcionada no sera llamada.

#### `future.ap`
```hs
:: Future e (a -> b) ~> Future e a -> Future e b
```
Aplica la funcion resuelta de esta instancia de `Future` al valor resuelto de la instancia de `Future` proporcionada, produciendo una instancia `Future` resuelta con el resultado. Si `Future` es rechazado, entonces la instancia de `Future` devuelta sera rechazada con ese valor. Cuando `fork` es llamado en la instancia devuelta, las acciones de esa instancia de `Future` y la instancia de `Future` proporcionada se ejecutara en paralelo.

#### `future.chain`
```hs
:: Future e a ~> (a -> Future e b) -> Future e b
```
Llama a la funcion proporcionada con el valor resuelto de esa instancia de `Future`,  devolviendo la nueva instancia de `Future`. Si se rechaza cualquier instancia de `Future`, la instancia de `Future` devuelta se rechazara con ese valor.

#### `future.chainReject`
```hs
:: Future e a ~> (e -> Future e b) -> Future e b
```
Si se rechaza esta instancia de `Future`, la funcion proporcionada se llamara con el valor rechazado, donde una nueva instancia de `Future` se retornara. Esto se puede utilizar para recuperarse de una instancia de `Future` rechazada. Si se resuelve esta instancia de `Future`, se omitira la funcion proporcionada.

#### `future.bimap`
```hs
:: Future e a ~> (e -> f) -> (a -> b) -> Future f b
```
Utiliza las funciones proporcionadas para transformar esta instancia de `Future` cuando se rechaza o se resuelve respectivamente.