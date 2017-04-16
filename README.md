ramda-fantasy
=============
Tipos compatibles con la especificacion de [Fantasy Land][1] para la integración sencilla con [Ramda][2].

## Estado del proyecto
Este proyecto se encuentra en estado alfa. La implementación de las especificaciones de Fantasy Land deberia ser en su mayoria estable. Cualquier método fuera de las especificaciones de Fantasy Land está sujeto a cambios. Los tipos no han sido sometidos a pruebas exhaustivas de uso aún.

## Tipos disponibles.

| Nombre     | [Setoid][3]  | [Semigroup][4] | [Functor][5] | [Applicative][6] | [Monad][7] | [Foldable][8] |
| --------------- | :----------: | :------------: | :----------: | :--------------: | :--------: | :-----------: |
| [Either][9]     |    **✔︎**     |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |               |
| [Future][10]    |              |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |               |
| [Identity][11]  |    **✔︎**     |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |               |
| [IO][12]        |              |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |               |
| [Maybe][13]     |    **✔︎**     |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |     **✔︎**     |
| [Reader][14]    |              |                |     **✔︎**    |      **✔︎**       |   **✔︎**    |               |
| [Tuple][15]     |    **✔︎**     |     **✔︎**      |     **✔︎**    |                  |            |               |


El acceso es asi:
```js
  var Either = require('ramda-fantasy').Either;
```

[1]: https://github.com/fantasyland/fantasy-land
[2]: https://github.com/ramda/ramda
[3]: https://github.com/fantasyland/fantasy-land#setoid
[4]: https://github.com/fantasyland/fantasy-land#semigroup
[5]: https://github.com/fantasyland/fantasy-land#functor
[6]: https://github.com/fantasyland/fantasy-land#applicative
[7]: https://github.com/fantasyland/fantasy-land#monad
[8]: https://github.com/fantasyland/fantasy-land#foldable
[9]: docs/Either.md
[10]: docs/Future.md
[11]: docs/Identity.md
[12]: docs/IO.md
[13]: docs/Maybe.md
[14]: docs/Reader.md
[15]: docs/Tuple.md
