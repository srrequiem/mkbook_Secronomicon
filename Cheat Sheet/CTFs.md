# Pwning

## Validaciones iniciales

| Herramienta | Protección | Valor | Descripción |
|---|---|---|---|
| `file` | not stripped | NA | Indica que al hacer ingenieria inversa no se podrá ver el nombre de las funciones, etc. |
| `checksec` | PIE | PIE {enabled, disabled} | Cada vez que el programa se ejecute, contará con diferentes direcciones en memoria. |
| `checksec` | NX | NX {enabled, disabled} | Si esta deshabilitado indica que si se llegase a tener la capacidad de inyectar codigo en la pila se podrá ejecutar. |
| `checksec` | Stack CANARY | Canary found \| No canary found | Si se encuentra habilitado, las funciones cuentan con un valor aleatorio asignado si este valor se ve modificado el binario mostrará un mensaje de tipo `stack smashing detected`. |
| `checksec` | RELRO | {Full, PARTIAL} RELRO | Indica la capacidad que se tiene para la lectura y escritura. *TODO: Verificar mas adelante |

## Rop Attacks

Altera la ejecución a `libc`.
- Así como el regular `ret-2-libc`, puede ser usado con stacks no ejecutables ya que la instrucciones puede ser ejecutadas.
- A diferencia de `ret-2-libc` no requiere ejecutar funciones que se encuentren `libc` (puede ejecutar cualquier código arbitrario).

### Explotación

#### Paso 1: Encontrar Gadgets

- Un gadget es una sequencia corta de instrucciones seguidas de un `return`. Ejemplo: `pop rdi; ret;`
- Las instrucciones útiles no deberán transferir el control fuera del gadget.
- Este paso es de pre-procesamiento por medio de análisis estático a la librería `libc`.

#### Paso 2: Stitching

