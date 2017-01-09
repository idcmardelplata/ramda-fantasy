# IO

El tipo IO se utiliza para almacenar una función que describe algún cálculo con efectos secundarios, como leer algunos datos de un archivo, imprimir información de registro en la consola o modificar los elementos del DOM.
Describir las acciones de esta manera permite que las instancias de IO sean compuestas y pasadas alrededor mientras que se mantienen las funciones puras y la  transparencia referencial.

## Construcción

El tipo `IO` consiste en un único constructor que acepta una función nula que describe la acción a realizar, dando como resultado una `IO` del valor eventualmente devuelto de la acción `IO :: (() -> a) -> IO a`.

```js
//:: IO [String]
argsIO = IO(() => R.tail(process.argv));

//:: String -> IO String
readFile = filename => IO(() => fs.readFileSync(filename, 'utf8'));

//:: String -> IO ()
stdoutWrite = data => IO(() => process.stdout.write(data));
```
Es importante tener en cuenta que al construir una instancia de `IO`, la acción no se ejecutará hasta que se llame el método `runIO`.

## Interacción

Una instancia de `IO` implementa la especificación de mónada, permitiendo que los resultados de las acciones sean transformados o encadenados juntos. El método `runIO` debe ser llamado para finalmente ejecutar la acción. Se recomienda que la ejecución de una instancia de `IO` se delegue en los bordes exteriores de una aplicación,
Hasta el punto en que una aplicación consistirá en una única instancia de IO en el punto de entrada.

```js
// Read each file provided in the arguments of the running application, echoing
// their contents to stdout after transforming the text to uppercase characters
//:: String -> IO ()
loudCat = argsIO.chain(R.traverse(IO.of, readFile))
                .map(R.join('\n'))
                .map(R.toUpper)
                .chain(stdoutWrite);

loudCat.runIO();
```

## Referencia

### Constructores

#### `IO`
```hs
:: (() -> a) -> IO a
```
Crea una instancia de `IO` que representa alguna acción que posiblemente tenga efectos secundarios.

### Metodos estaticos

#### `IO.runIO`
```hs
:: IO a -> a
```
Ejecuta la acción descrita por la instancia `IO` dada. Esto también está disponible como un método de instancia.

#### `IO.of`
```hs
:: a -> IO a
```
Produce una instancia de `IO` que da como resultado el valor dado.

### Metodos de instancia.

#### `io.runIO`
```hs
:: IO a ~> () -> a
```
Ejecuta la acción descrita por esta instancia de `IO`. Esto también está disponible como un método estático.

#### `io.map`
```hs
:: IO a ~> (a -> b) -> IO b
```
Transforma el resultado de esta instancia de `IO` con la función proporcionada.

#### `io.ap`
```hs
:: IO (a -> b) ~> IO a -> IO b
```
Produce una nueva instancia de `IO` que cuando se ejecuta, aplica la función resultante de la acción en esta instancia de `IO` al valor resultante de la acción en la instancia de IO dada.

#### `io.chain`
```hs
:: IO a ~> (a -> IO b) -> IO b
```
Produce una instancia de `IO` que cuando se ejecuta, aplicará la función dada al resultado de la acción en esta instancia de `IO` y luego ejecutará la acción de `IO` resultante.