---
layout: page
title: Introducción a los trabajos prácticos
permalink: /tps/intro
---

# Introducción a los trabajos prácticos
{:.no_toc}

## Contenidos
{:.no_toc}

* Contenidos
{:toc}


## Programas y bibliotecas

La mayoría de las entregas de la materia no consistirán en un programa completo sino en una biblioteca. En esta sección se estudian, brevemente, las diferencias entre ambos, y la manera de compilar cada uno.


### Compilación de programas

Un _programa_ lo componen uno o más archivos de código, tal que alguno de ellos define una función llamada _main_. Por ejemplo, considérese el siguiente archivo _hola.c:_

```c
// Archivo hola.c
#include <stdio.h>

int main() {
    printf("¡Hola desde main! Soy un programa.\n");
}
```

Este archivo constituye un programa porque la función _main_ está definida.P ara compilarlo, se invoca al compilador pasando como argumento el nombre del archivo:[^dollarprompt]

[^dollarprompt]: En este y otros ejemplos, el caracter `$` indica que lo que sigue es un comando a teclear en la terminal (o intérprete de comandos). Se usa esta convención porque el intérprete muestra dicho símbolo cuando está a la espera de la siguiente orden.

```
$ gcc -o hola hola.c
$ ./hola
```

En la primera orden, la opción `-o hola` le indica a _gcc_ que el resultado de la compilación debe guardarse en un archivo llamado _hola_. La segunda orden, `./hola`, ejecuta el archivo resultante de la compilación, el cual se encuentra en el directorio actual (representado por `./`). La salida de este segundo comando sería:

```
¡Hola desde main! Soy un programa.
```

Cabe notar que el nombre del archivo destino no tiene por qué guardar relación con el archivo de código. Se puede elegir cualquier otro nombre, por ejemplo:

```
$ gcc -o mi_ejemplo hola.c
$ ./mi_ejemplo
¡Hola desde main! Soy un programa.
```

Asimismo, también se puede omitir la opción `-o`, en cuyo caso el compilador usa el nombre por omisión _a.out:_

```
$ gcc hola.c
$ ./a.out
¡Hola desde main! Soy un programa.
```


#### Opciones de compilación adicionales

Casi siempre que se compila un programa se usa la opción `-o` para especificar el nombre de archivo deseado. También hay otra serie de opciones que, en general, se incluyen siempre. Para el código de la materia usaremos, entre otras, las siguientes:

```
$ gcc -g -O2 -std=c99 -Wall hola.c
```

En la documentación de GCC se puede encontrar una descripción detallada de cada una de ellas; se incluye a continuación una breve descripción de las mismas:

  - `-g`: se solicita al compilador que incluya _información de depurado_ en el binario resultante, útil en caso de depurar con [GDB].

  - `-O2`: se solicita al compilador que genere generar código máquina _optimizado_ (esto es, que es más eficiente y corre más rápido), resultado de considerar el código C en su conjunto, y no "compilando" línea a línea.

  - `-std=c99`: se indica al compilador que el código C proporcionado está escrito en la _versión C99 del estándar_ (otras versiones del estándar de C incluyen C11 y C89).

  - `-Wall`: se pide al compilador que emita _avisos de posibles errores en el código_, esto es: código que compila y puede ser ejecutado, pero que —según la opinión del compilador— capaz albergue algún error, o constituya una mala práctica.

[GDB]: https://es.wikipedia.org/wiki/GNU_Debugger


### Compilación de bibliotecas

Una _biblioteca_ se compone de uno o más archivos de código tal que ninguno de ellos contiene una función llamada _main_. El propósito de la biblioteca es ser una repositorio de código (funciones) que otros programas pueden usar. Pero no es posible _correr_ la biblioteca en sí.

Esta forma de escribir código (sin _main)_ nos es relevante porque, como se mencionó arriba, la mayoría de las entregas de la materia serán bibliotecas, y no programas completos.

Considérese como ejemplo la entrega de una hipotética biblioteca llamada _mathbib_ que contiene una función _fact_ para calcular el factorial de un número natural, y una función _fib_ para calcular un número de la secuencia de Fibonacci. El archivo a entregar sería este:

```c
// Archivo mathbib.c
int fact(unsigned n) {
    if (n <= 1)
        return n;
    return n * fact(n-1);
}

int fib(unsigned i) {
    if (i <= 1)
        return i;
    return fib(i-1) + fib(i-2);
}
```

A la hora de compilar, si intentase hacer de la manera vista hast aahora (como un programa), se obtendría el siguiente mensaje de error:

```
$ gcc -g -O2 -std=c99 -Wall funciones.c
ld: in function ‘_start’: undefined reference to ‘main’
```

Esto ocurre porque _mathbib.c_ no contiene una función _main_ que especifique su comportamiento como programa: es tan solo una colección de funciones que otros programas pueden invocar si la incluyen en su proceso de compilación.

No obstante, sí es posible compilar una biblioteca de manera independiente a un programa, de tal manera que se pueda estar seguro que no contiene errores de sintaxis ni avisos del compilador. Ello se hace pasando la opción `-c` al compilador (y eliminando, por superflua, la opción `-o`):

```
$ gcc -g -O2 -std=c99 -Wall -c mathbib.c
```

Esta invocación no produce un archivo binario ejecutable sino un archivo objeto con extensión `.o` (en este caso, se genera un archivo llamado  _mathbib.o)._ Este archivo contiene el código máquina generado por el compilador para ambas funciones, pero no es un archivo ejecutable (no se puede correr).


#### Uso de bibliotecas

Como ejemplo de programa que usa una biblioteca, consideremos un programa _hola2.c_ que desea hacer uso de las funciones disponibles en _mathbib.o:_

```c
// Archivo hola2.c
#include <stdio.h>

int main() {
    printf("¡Hola! El factorial de 7 es: %d.\n", fact(7));
    printf("Los seis primeros números de Fibonacci son:");

    for (int i=1; i <= 6; i++) {
        printf(" %d", fib(i));
    }

    puts(".");
}
```

Para compilar este programa se debe, en general, compilar primero la biblioteca, y a continuación el programa:[^warnundef]

[^warnundef]: En el ejemplo que sigue, en la parte marcada con `...` se producen ciertos aviso de compilación que se estudiarán en la siguiente sección.

```
$ gcc -c mathbib.c
$ gcc -o hola2 hola2.c mathbib.o
...
$ ./hola2
¡Hola! El factorial de 7 es: 5040.
Los diez primeros números de Fibonacci son: 1 1 2 3 5 8.
```


#### Prototipos y cabeceras

Al compilar _hola2.c_ y _mathbib.o_ juntos, se observan los siguientes avisos de compilación:

```
$ gcc -o hola2 hola2.c mathbib.o
hola2.c: In function ‘main’:
hola2.c: warning: implicit declaration of function ‘fact’
    printf("¡Hola! El factorial de 7 es: %d.\n", fact(7));
                                                 ^~~~
hola2.c: warning: implicit declaration of function ‘fib’
    printf(" %d", fib(i));
                  ^~~
```

Esto sucede porque el compilador no conoce de la existencia de las funciones _fact_ y _fib_ (de igual manera que tampoco conocería la existencia de _puts_ y _printf_ si no se incluyese la cabecera _stdio.h_).

Es por esto que, en realidad, una biblioteca no solo se compone de archivos fuente u objeto (_mathlib.c_, _mathbib.o)_ sino de un archivo “cabecera” adicional que indica al compilador la lista de funciones que la biblioteca proporciona.

El archivo cabecera (o _header file_, en inglés) incluye las funciones en forma de prototipo, es decir: nombre de la función, tipo de los parámetros y tipo del valor de retorno. A la cabecera se la suele designar con el mismo nombre de la biblioteca, pero con extensión _.h_, así:

```c
// Archivo mathbib.h
int fact(unsigned n);
int fib(unsigned n);
```

Además, es también común incluir en este archivo una descripción breve de lo que hace cada función; por ejemplo:

```c
/*
 * Calcula el factorial de ‘n’.
 */
int fact(unsigned n);

/*
 * Calcula el elemento i-ésimo de la serie de Fibonacci.
 */
int fib(unsigned i);
```

Por tanto, en todas las entregas se pedirá tanto el código fuente (en este caso, _mathbib.c)_ como la cabecera (en este caso, _mathbib.h)_.


## Pruebas automáticas

Para cualquier programa o biblioteca es útil tener una manera de verificar que la implementación se comporta de la manera esperada. Esto es útil durante el desarrollo, para comprobar que un determinado cambio no modificó el comportamiento de la aplicación de manera no deseada; y también al terminar, para verificar que el comportamiento final es el esperado.

Es muy deseable que este proceso de prueba (o _testing_, en inglés) sea lo menos manual posible. De hecho, lo más deseable es que se trate de un proceso completamente automático, sin ningún paso a realizar “a mano”. En términos prácticos, esto supondría la ejecución de un programa que dica _OK_ o _ERROR_, indicando qué errores se produjeron durante el proceso de prueba.

A continuación se estudian varias maneras de automatizar las pruebas de la biblioteca _mathbib_, cada una más sofisticada que la anterior.[^progtest]

[^progtest] El proceso de pruebas para un programa con función _main_ se estudiará  en el TP1, que es el primer trabajo práctico que incluye un programa en la entrega.


### Automatización de las pruebas

**(1)** En la primera versión de las pruebas, se introduce un programa  _pruebas_mathbib.c_ que incluye una función _main_. Esta función solamente imprime los resultados de invocar a las funciones con diversos parámetros:

```c
// Archivo pruebas_mathbib_v1.c
#include "mathbib.h"
#include <stdio.h>

int main() {
    // Pruebas de fact(), v1.
    printf("fact(1) = %d\n", fact(1));
    printf("fact(4) = %d\n", fact(4));
    printf("fact(9) = %d\n\n", fact(9));

    // Pruebas de fib(), v1.
    printf("fib(2) = %d\n", fib(2));
    printf("fib(5) = %d\n", fib(5));
    printf("fib(9) = %d\n", fib(9));
}
```

siendo la salida:

```
fact(1) = 1
fact(4) = 24
fact(9) = 362880

fib(2) = 1
fib(5) = 5
fib(9) = 34
```

Esto no es demasiado útil, porque no queda dicho cuál sería el valor correcto de cada operación (en otras palabras: para saber si la biblioteca funciona o no, se debe realizar primero la operación a mano, y después comparar; pero realizar pasos a mano es, precisamente, lo que queremos evitar).

**(2)** La necesidad de cómputo manual se puede subsanar dejando que el propio archivo de pruebas sepa cuál es el resultado esperado, así:

```c
// Archivo pruebas_mathbib_v2.c
#include "mathbib.h"
#include <stdio.h>

int main() {
    // Pruebas de fact(), v2.
    printf("fact(1) = %d   esperado: 1\n", fact(1));
    printf("fact(4) = %d   esperado: 24\n", fact(4));
    printf("fact(8) = %d   esperado: 40320\n\n", fact(8));

    // Pruebas de fib(), v2.
    printf("fib(2) = %d   esperado: 1\n", fib(2));
    printf("fib(6) = %d   esperado: 8\n", fib(6));
    printf("fib(9) = %d   esperado: 34\n", fib(9));
}
```

La salida en este caso es:

```
fact(1) = 1     esperado: 1
fact(4) = 24    esperado: 24
fact(8) = 40320 esperado: 40320

fib(2) = 1      esperado: 1
fib(6) = 8      esperado: 8
fib(9) = 34     esperado: 34
```

Pero aun así, la comparación entre el valor real y el valor esperado la debe realizar la persona que corre las pruebas, cuando estas mismas la podrían realizar.

**(3)** En la tercera iteración de las pruebas se incluye la comparación en el propio programa de prueba, así:[^onlyfact]

[^onlyfact]: Por brevedad, se muestra tan solo para la función _fact_.


```c
// Archivo pruebas_mathbib_v3.c
#include "mathbib.h"
#include <stdio.h>

int main() {
    // Pruebas v3, con comparación entre real y esperado.
    int real;
    int esperado;
    int fallos = 0;

    // Prueba fact(1).
    esperado = 1;
    real = fact(1);

    if (real == esperado) {
        printf("fact(1)... ok.\n");
    }
    else {
        fallos++;
        printf("fact(1) = %d, esperaba %d\n", real, esperado);
    }

    // Prueba fact(4).
    esperado = 24;
    real = fact(4);

    if (real == esperado) {
        printf("fact(4)... ok.\n");
    }
    else {
        fallos++;
        printf("fact(4) = %d, esperaba %d\n", real, esperado);
    }

    // Prueba fact(8).
    esperado = 40320;
    real = fact(8);

    if (real == esperado) {
        printf("fact(8)... ok.\n");
    else {
        fallos++;
        printf("fact(8) = %d, esperaba %d\n", real, esperado);
    }

    printf("Resultado final: %s\n", fallos ? "ERROR": "Todo OK");
}
```

La salida es:

```
fact(1)... ok.
fact(4)... ok.
fact(8)... ok.
Resultado final: Todo OK.
```

En esta versión, no necesitamos mirar el valor devuelto por las funciones, solo si la prueba dijo _OK_ o no.

### La función _test()_

La tercera iteración de las pruebas es más útil desde el punto de vista de quien corre las pruebas, pero el código que produce ese resultado es más tedioso de escribir. No obstante, puede mejorarse introduciendo una función auxiliar, así:

```c
// Archivo testing.h
extern int fallos;  // Declaración de variable global.

/* Imprime un mensaje indicando si ‘real’ y  ‘esperado’ son
 * iguales. Se incrementa la variable global “fallos” si no
 * lo son.
 */
void test(const char *msg, int esperado, int real);
```

El archivo principal del pruebas quedaría:

```c
// Archivo pruebas_mathbib.c
#include "mathbib.h"
#include "testing.h"

int fallos;  // Definición (siempre junto a main).
int main() {
    puts("== Pruebas de fact()");
    test("fact(1)", 1, fact(1));
    test("fact(4)", 24, fact(4));
    test("fact(8)", 40320, fact(8));

    puts("\n== Pruebas de fib()");
    test("fib(2)", 1, fib(2));
    test("fib(6)", 8, fib(6));
    test("fib(9)", 34, fib(9));

    printf("\nResultado final: %s\n", fallos ? "ERROR": "Todo OK");
}
```

La salida en esta ocasión es:

```
== Pruebas de fact()
fact(1)... ok.
fact(4)... ok.
fact(8)... ok.

== Pruebas de fib()
fib(2)... ok.
fib(6)... ok.
fib(9)... ok.

Resultado final: Todo OK.
```

Y este es el código de _test()_, que vale para cualquier función (no solo _fact_ y _fib:_

```c
// Archivo testing.c
#include "testing.h"

void print_test(const char *msg, int esperado, int real) {
    if (real == esperado) {
        printf("%s... ok.\n", msg);
    } else {
        fallos++;
        printf("%s = %d\n, esperaba %d\n.", msg, real, esperado);
    }
}
```


### Pruebas unitarias con ‘ctest’

Este tipo de pruebas que verifican el comportamiento de una función se denominan _pruebas unitarias_ (la unidad de prueba es la función). Existen múltiples herramientas y _frameworks_ para escribirlas, con distinto nivel de funcionalidad. (La función _test()_ de la sección anterior correspondería con un ‘mini-framework’ con funcionalidad muy básica.)

En la materia usaremos un framework de funcionalidad más o meons básica denominado _ctest_. En él, _testing.h_ y _testing.c_ se sustituyen por _ctest.h_, y el archivo _pruebas_mathbib.c_ quedaría así:

```c
#define CTEST_MAIN 1
#define ctest_main main

#include "ctest.h"
#include "mathbib.h"

CTEST(factorial, casos_base) {
    ASSERT_EQUAL(0, fact(0));
    ASSERT_EQUAL(1, fact(1));
}

CTEST(factorial, casos_generales) {
    ASSERT_EQUAL(24, fact(4));
    ASSERT_EQUAL(40320, fact(8));
}

CTEST(fibonacci, casos_base) {
    ASSERT_EQUAL(0, fib(0));
    ASSERT_EQUAL(1, fib(1));
}

CTEST(fibonacci, casos_generales) {
    ASSERT_EQUAL(8, fib(6));
    ASSERT_EQUAL(33, fib(9));
}
```

Obsérvese:

  - el código es declarativo, esto es: no se realizan comparaciones explícitas, sino que —con `ASSERT`— se comunican las _expectativas_ (esto debe ser igual a esto otro);[^moreassert]

  - se agrupan las pruebas primero por función: `CTEST(factorial, ...)`; y después por “unidad lógica” (`casos_borde`, `casos_generales`).

  - no hay función main explícita, el mismo archivo _ctest.h_ la define;

  - no se include _stdio.h_ porque toda la impresión de mensajes la maneja la propia biblioteca _ctest_.

[^moreassert]: Existen más formas de _assert_ (además de `ASSERT_EQUAL`) que iremos viendo: `ASSERT_NOT_EQUAL`, `ASSERT_NULL`, `ASSERT_NOT_NULL`, etc.

La salida es:

```
TEST 1/4 factorial:casos_base [OK]
TEST 2/4 factorial:casos_normales [OK]
TEST 3/4 fibonacci:casos_base [OK]
TEST 4/4 fibonacci:casos_normales [OK]
RESULTS: 4 tests (4 ok, 0 failed, 0 skipped) ran in 0 ms
```


## Compilación con ‘make’


## El corrector automático


{% include footnotes.html %}
