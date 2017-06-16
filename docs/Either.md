# Either

El tipo `Either` es muy similar al tipo `Maybe`, el cual se utiliza a menudo para
representar la nocion de un fallo de alguna manera. La diferencia entre los dos
es que un error representado con `Either` puede contener algun valor 
(tal vez una excepcion o algun mensaje de error), mientras que `Maybe`
solo puede indicar la ausencia de algun valor. 

Mientras que el tipo `Either` es usado a menudo para representar potenciales errores, no hay nada que lo restrinja a ese proposito. Por lo tanto es quizas mas apropiado pensar simplemente en `Either` como una representacion de dos posibles tipos de valores, a veces denominados *union disjunta*, o *coproducto* de dos tipos.


## Construcción

El tipo `Either` consiste en dos constructores, `Left :: a -> Either a b` y `Right :: b -> Either a b`. Donde un tipo `Either` es usado para representar la posibilidad de un error, `Left` es tipicamente usado para mantener el valor del error y `Right`
 mantiene el valor "exitoso" (se puede pensar en `Right` como el valor correcto).

Vale la pena resaltar que los tipos de valor almacenados en un `Left` no tienen porque ser del mismo tipo que en `Right` en el mismo `Either`. Esta es la razon por la cual se documenta que tiene dos parametros de tipo `Either a b`, donde `a` representa el tipo contenido en `Left` y `b` el valor contenido en `Right`.

## Interacción

Con el fin de satisfacer una serie de leyes relacionadas con las interfaces implementadas, solo un unico parametro de tipo se puede utilizar como el argumento de las diversas funciones. Por esta razón, `Either` es *right-biased*, significa que los metodos como `map`, `ap` y `chain` solamente llamaran a la funcion dada para una instancia de `Right` mientras que en `Left` pasara simplemente la instancia ignorando la funcion proporcionada.

Aparte de los metodos de transformación mencionados anteriormente, `Either.either` se puede utilizar para extraer el valor de un tipo `Either`, este metodo toma una funcion para manejar el valor en `Left` y otra funcion para manejar el valor en `Right`. Solo se llamara a una de estas funciones, dependiendo del constructor que contenga en valor.

## Referencia

### Constructores

#### `Either.Left`
```hs
:: a -> Either a b
```
Construye una instancia `Left` de `Either`. Esto es comunmente utilizado para reprensentar un fallo.

#### `Either.Right`
```hs
:: b -> Either a b
```
Construye una instancia `Right` de `Either`. Esto es comunmente usado para representar un exito donde existe la posibilidad de un fracaso.

### Funciones estaticas

#### `Either.either`
```hs
:: (a -> c) -> (b -> c) -> Either a b -> c
```
Usado para extraer el valor dentro de un `Either` proporcionando una funcion para manejar los tipos de valores contenidos tanto en `Left` como en `Right`.

#### `Either.of`
```hs
:: b -> Either a b
```
Crea una instancia pura de `Either`. Este es efectivamente el constructor de `Right`

#### `Either.isLeft`
```hs
:: Either a b -> Boolean
```
Retorna `true` si la instancia de `Either` dada es `Left`, sino retorna `false`.

#### `Either.isRight`
```hs
:: Either a b -> Boolean
```
Retorna `true` si la instancia de `Either` dada es `Right`, en caso contrario retorna `false`.

### Metodos de instancia

#### `either.map`
```hs
:: Either a b ~> (b -> c) -> Either a c
```
Modifica el valor contenido en `Right` con la funcion dada. Si la instancia es `Left`, el valor se deja sin modificar.

#### `either.ap`
```hs
:: Either a (b -> c) ~> Either a b -> Either a c
```
Aplica la funcion contenida en la instancia de `Right` a un valor contenido dentro de un `Right` proporcionado, produciendo otra instancia de `Right` conteniendo el resultado. Si la instancia es `Left`, el resultado sera la instancia de `Left`. Si la instancia es `Right` y el either es `Left` el resultado sera el `Left` proporcionado.

#### `either.chain`
```hs
:: Either a b ~> (b -> Either a c) -> Either a c
```
Aplica la funcion proporcionada al valor contenido en la instancia de `Right`, resultando en el valor de retorno de la funcion. Si la instancia es `Left`, la funcion es ignorada y la instancia de `Left` es retornada sin modificar.

#### `either.extend`
```hs
:: Either a b ~> (Either a b -> c) -> Either a c
```
Aplica la funcion proporcionada a la instancia de `Right`, retornando un `Right` conteniendo el resultado de la aplicacion de la funcion. Si la instancia es `Left`, la funcion es ignorada y la instancia de `Left` es retornada sin modificaciones.

#### `either.bimap`
```hs
:: Either a b ~> (a -> c) -> (b -> d) -> Either c d
```
Transforma un `Either` aplicando la primera funcion al valor contenido si la instancia es un `Left`, o la segunda funcion si la instancia es un `Right`.

#### `either.equals`
```hs
:: Either a b ~> * -> Boolean
```
Determina si el valor proporcionado es igual a esta instancia, asegurando que ambos son del mismo constructor y ambos contienen valores iguales.

#### `either.toString`
```hs
:: Either a b ~> () -> String
```
Retorna una reprensentacion en string de la instancia de `Either`.