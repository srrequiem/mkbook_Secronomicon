# Introduction: What is Computer Systems Security

- [Diapositivas](https://docs.google.com/presentation/d/1YlTxeZg03P234EgG4E4JNGcit6LZovAxfYGL1YSLwfc/edit?usp=sharing)
- [Extended Q&A 8/19/2021](https://www.youtube.com/watch?v=2chgNhxt-Kc)

# Fundamentals

## Computer Architecture

- John Mauchly (Físico).
- John Presper Eckbert (Ingeniero eléctrico).
- John Von Neumann (Matemático).

John Von Neumann, EDVAC, 1945.

![Arquitectura de computadoras como se conoce al día de hoy](images/computer_arch.png)

## Introduction to Binary Files

Partiendo de un binario de sistema como lo es `/bin/cat`, buscando responder a la pregunta:

```bash
❯ file /bin/cat
/bin/cat: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7b3c4131450fabd010fe74ac5589074c9431cf4a, for GNU/Linux 3.2.0, stripped
```

### What is an ELF?

- Executable and Linkable Format.
- ELF es un formato de archivo binario.
- Contiene el programa y su data.
- Describe como un programa debería ser cargado (program/segment headers).
- Contiene metadata que describen los componentes del programa (section headers).

### ELF Program Headers

- Especifícan la información requerida para preparar el programa para su ejecución. Los tipos de entrada más importantes son:
  - **INTERP:** define la librería que debería ser usada para cargar este ELF en la memoria.
  - **LOAD:** define una parte del archivo que debería ser cargada en la memoria.
- Son la fuente de información usada cuando se carga un archivo.

*Extracción de `readelf -a /bin/cat`:*

```bash
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x0000000000000268 0x0000000000000268  R      0x8
  INTERP         0x00000000000002a8 0x00000000000002a8 0x00000000000002a8
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000001540 0x0000000000001540  R      0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x0000000000004819 0x0000000000004819  R E    0x1000
  LOAD           0x0000000000007000 0x0000000000007000 0x0000000000007000
                 0x0000000000001f28 0x0000000000001f28  R      0x1000
  LOAD           0x0000000000009c50 0x000000000000ac50 0x000000000000ac50
                 0x0000000000000630 0x00000000000007c8  RW     0x1000
  DYNAMIC        0x0000000000009df8 0x000000000000adf8 0x000000000000adf8
                 0x00000000000001e0 0x00000000000001e0  RW     0x8
  NOTE           0x00000000000002c4 0x00000000000002c4 0x00000000000002c4
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_EH_FRAME   0x0000000000007eb0 0x0000000000007eb0 0x0000000000007eb0
                 0x00000000000002e4 0x00000000000002e4  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000009c50 0x000000000000ac50 0x000000000000ac50
                 0x00000000000003b0 0x00000000000003b0  R      0x1
```

### ELF Section Headers

- Son una vista diferente de un ELF con información útil para introspección, debuggeo, etc.
- Secciones importantes:
  - **.text:** el código ejecutable del programa.
  - **.plt** y **.got:** usado para resolver y despachar llamadas de librerías.
  - **.data:** usado por data global pre-inicializada que se puede modificar (así como arreglos globales con valores iniciales).
  - **.rodata:** usada por data de sólo lectura (tal como cadenas de caracteres constantes).
  - **.bss:** usada por data global no inicializada que se puede modificar (así como arreglos globales sin valores iniciales).
- Los Section headers *no* son necesariamente una parte del ELF, sólo segmentos definidos por los program headers son necesarios para su carga y su operación.
- Los Section headers son sólo metadata.

### Symbols

- Binarios (y librerías) que usen librerías cargadas dinámicamente dependen de los `symbols` (los nombres) para encontrar las librerías, y resolver las llamadas a funciones a esas librerías, etc.

![Tabla de symbols](images/symbols.jpg)

### Interacting with your ELF

- **gcc** para crear el ELF.
- **readelf** para analizar el header del ELF.
- **objdump** para analizar el header del ELF y desensamblar el código fuente.
- **nm** para visualizar los symbols del ELF.
- **patchelf** para cambiar algunas propiedades del ELF.
- **objcopy** para intercambiar secciones del ELF.
- **strip** para remover información relevante (tal como los symbols).
- **katai** struct (http://ide.kaitai.io/) para visualizar a través del ELF interactivamente.

## Linux Process Loading

### Portrait of a process

Cada proceso de Linux tiene:

- estado (running, waiting, stopped, zombie).
- prioridad (y otro tipo de información relacionada a la programación/planificación).
- padre, hermanos, hijos.
- recursos compartidos (archivos, pipes, sockets).
- espacio en memoria virtual.
- contexto de seguridad.
  - uid y gid efectivos.
  - uid y gid guardados.
  - capabilities.

### But where do processes come from?

En Linux, los procesos ¡se progagan por mitosis!

Mediante **fork** y (más reciente) **clone** son llamadas a sistema que crean una casi idéntica copia  del proceso del que lo está llamando: un padre y un hijo.

Después, el proceso hijo usualmente usa la syscall **execve** para remplazarse a sí mismo con otro proceso.

Ejemplo:

- Cuando se usa `/bin/cat` en bash.
- bask hace un **fork** de sí mismo a un viejo proceso padre y al proceso hijo.
- El proceso hijo usa **execve** `/bin/cat` transformándose en `/bin/cat`.

### Can we load?

Antes que cualquier cosa sea cargada, el kernel verifica los permisos de ejecución.

Si un archivo no es ejecutable, **execve** fallará.

### What to load?

Para identificar que cargar, el kernel de Linux lee la primera parte del archivo (`/bin/cat` por ejemplo) y toma una decisión:

1. Si el archivo empieza con **#!** (lo que es conocido como el shebang), el kernel extrae el intérprete del resto de la línea y ejecuta el intérprete con el archivo original como argumento.
2. Si el archivo corresponde al formato de `/proc/sys/fs/binfmt_misc`, el kernel ejecuta el intérprete especificado por ese formato con el archivo original como argumento.
3. Si el archivo es un ELF enlazado dinámicamente, el kernel lee el intérprete/loader definido en el ELF, carga el intérprete y el archivo original, y deja que el intérprete tome el control.
4. Si el archivo es un ELF enlazado estáticamente, el kernel lo cargará.
5. Otros formatos de archivos legacy son verificados.

¡Esto puede ser recursivo!

### Dynamically linked ELFS: the interpreter

La carga del proceso es realizada por el intérprete del ELF específicado en el binario.

```bash
❯ readelf -a /bin/cat | grep interpreter
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

Coloquialmente conocido como "the loader".

Puede ser sobreescrito: `/lib64/ld-linux-x86-64.so.2 /bin/cat /flag`

O cambiado permanentemente `patchelf --set-interpreter`

### Dynamically linked ELFS: the loading process

1. El programa y su intérprete son cargados por el kernel.
2. El intérprete ubica las librerías.
   1. La variable de entorno `LD_PRELOAD`, y cualquier cosa que se encuentre en `/etc/ld.so.preload`.
   2. La variable de entorno `LD_LIBRARY_PATH` (puede ser configurada en la shell).
   3. `DT_RUNPATH` o `DT_RPATH` especificados en el binario (ambos pueden ser modificados por patchelf).
   4. Configuración del sistema (`/etc/ld.so.conf`).
   5. `/lib` y `/usr/lib`.
3. El intérprete carga las librerías.
   1. Estas librerías pueden depender de otras, causando que también sean cargadas.
   2. Actualización de reubicaciones.

### Where is all this getting loaded to?

Cada proceso en Linux tiene un espacio de memoria virtual. El cuál contiene:

- El binario.
- Las librerías.
- El "heap" (para la memoria asignada dinámicamente).
- El "stack" (para variables locales de funciones).
- Cualquier memoria escíficamente mapeada por el programa.
- Algunas regiones de apoyo.
- Código de kernel en la "mitad superior" de la memoria (arriba de 0x80000000000000 en arquitecturas de 64 bits), inaccesibles por el proceso.

La memoria virtual es dedicada al proceso.

La memoria física es compartida por todo el sistema.

Se puede visualizar todo el espacio mirando en `/proc/self/maps`.

### The Standard C library

libc.so es enlazada a casi cada proceso.

Provee de funcionalidada para:

- printf().
- scanf().
- socket().
- atoi().
- malloc().
- free().

...y a otro tipo de cosas

- xdr_keystatus()

### The loading process (for statically linked binaries)

1. El binario es cargado.

### Cat is initialized

Cada binario ELF puede especificar constructores, los cuales son funciones que son ejecutados antes de que el programa sea lanzado.

Por ejemplo, dependiendo en la versión, libc puede inicializar regiones en la memoria para asignaciones dinámicas (malloc/free) cuando el programa sea ejecutado.

¡Es posible especificar los propios!

```c
__attribute__((constructor)) void haha() {
    write(1, "Hello world!\n", 6);
}
```

Permitiendo ser válido también para `LD_PRELOAD`.

## Linux Process Runtime

### Cat is launched

Un ELF normal automáticamente llama `__libc_start_main()` en libc, lo cuál llama a la función `main()` del programa.

¡El código se está ejecutando!

¿Ahora qué?

### Cat reads its arguments and environment

`int main(int argc, void **argv, void **envp);`

Todo el input del programa proveniente del mundo exterior, al momento de la ejecución, comprende de:

- Los objetos cargados (binarios y librerías).
- Argumentos de la línea de comandos en argv.
- Variables de entorno o "environment" en envp.

Por supuesto, el programa necesita seguir interactuando con el mundo exterior.

### Using library functions

La operación del binaro *import symbols* necesita ser resuelta usando la operación *export symbols* de las librerías.

En el pasado, esto era un proceso on-demand y llevaba consigo un gran peligro.

Hoy en día, esto es hecho hecho cuando el binario es cargado y es mucho más seguro.

Se explorará este punto más adelante.

### Cat is initialized - 13:38

### Cat is initialized

### Cat is initialized

### Cat is initialized

# Further Reading

- An awesome intro series that covers some of the fundamentals from [LiveOverflow](https://www.youtube.com/watch?v=iyAyN3GFM7A&list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN&index=1).
- Phineas Fisher’s [writeup](https://pwn.college/modules/intro/phisher-hackback.txt) of the hacking team disclosure (discussed in the What is Computer Systems Security video). Originally posted on pastebin by Phineas Fisher, but since removed.
- [Some more (mirrored) writeups](https://github.com/Alekseyyy/phineas-philes) from Phineas Fisher, for the curious.
- Executable and Linkable Format 101 Part 1-4 - https://www.intezer.com/blog/research/executable-linkable-format-101-part1-sections-segments/
- System calls in the Linux kernel. Part 4. -  https://0xax.gitbooks.io/linux-insides/content/SysCall/linux-syscall-4.html
- Understanding the Memory Layout of Linux Executables - https://gist.github.com/CMCDragonkai/10ab53654b2aa6ce55c11cfc5b2432a4
- 