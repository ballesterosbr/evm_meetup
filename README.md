# evm_meetup

## Intro

La máquina virtual Ethereum (EVM) es el entorno de tiempo de ejecución (runtime) para contratos inteligentes en Ethereum. Se encuentra aislado, lo que significa que el código que se ejecuta dentro de la EVM no tiene acceso a la red, sistema de archivos u otros procesos de la blockchain. Los contratos inteligentes pueden tener acceso limitado a otros contratos inteligentes.

Los contratos viven en la cadena de bloques en un formato binario específico de Ethereum (EVM bytecode). Sin embargo, los contratos generalmente se escriben en un lenguaje de alto nivel de Ethereum, se compilan en bytecode usando un compilador de EVM y finalmente se cargan en la cadena de bloques utilizando un cliente de Ethereum. 

---

## Funciones

- Proporcionar seguridad y ejecución de código no confiable en ordenadores de todo el mundo.
- Prevenir denegación de servicios (DDoS).
- Asegurar que programas no tienen acceso al estado de otros.
- Asegurar que no haya interferencias.

## Restricciones

- Cada paso computacional que se realice en la ejecución de un programa debe ser pagado por adelantado (Gas). --> DDoS protection.

- Los programas sólo interactuan a través de la transmisión de matrices de datos únicas. --> No acceden a otros programas.

- La ejecución de programas es en modo sandbox. Un programa puede alterar su propio estado y acceder al mismo, además puede lanzar la ejecución de otro programa. --> No accede al estado de otros programas.

- La EVM tiene un comportamiento determinista.

---

Cada nodo contiene su propia implementación de la EVM capaz de ejecutar las mismas operaciones, lo que facilita la labor de desarrolladores a la hora de tener un entorno de pruebas.

Es una puerta de acceso a la creación de smart contracts en Solidity, pero también la EVM se ha implementado en Python, Ruby, C++ y otros lenguajes.

---

Interpretación de valores binarios de 256-bits como enteros. Cuando un dato de 256-bits se convierte a una address o hash de 160-bits, se conservan los 20 bytes más a la derecha mientras que los otros 12 bytes se descartan o se rellenan con ceros.

Aclaración big-endian --> MSB to LSB
```
13 	--> 0x3133 
trece 	--> 0x74726563650d0a
``` 
---

Se podría decir que la EVM es una cuasi máquina de Turing; "cuasi" por el hecho de que la computación está intrínsecamente limitada por un factor, el Gas, que limita la cantidad total de cálculo.

---

## Basics

- Arquitectura basada en pila.
- Tamaño de palabra de 256-bits. --> Tamaño de elemento de pila 256-bits.
- Basado para facilitar el esquema la función keccack-256 y cálculos de curva elíptica.
- Tamaño máximo de la pila: 1024.
- El modelo de memoria es una matriz de bytes (word-addressed byte array).
- El modelo de almacenamiento (independiente) es una matriz de palabras direccionables (word-addressable word array).

A diferencia de la memoria, que es volátil, el almacenamiento no es volátil y se mantiene como parte del estado del sistema. Todas las ubicaciones de almacenamiento y memoria están definidas inicialmente como cero.

La máquina no sigue la arquitectura estándar de la máquina de von Neumann. En lugar de almacenar el código del programa en una memoria o almacenamiento accesible, se almacena en una ROM virtual con la cual sólo se puede interactuar a través de una instrucción especializada.

---

Pueden suceder excepciones por varias razones, desbordamientos de pila, instrucciones no válidas. Una excepción bastante común suele ser "out-of-gas (OOG)" que no deja los cambios de estado intactos.

La máquina se detiene inmediatamente e informa el problema al procesador de transacciones o, recursivamente, al entorno agente de ejecución, ya sea el procesador de transacciones o, recursivamente, el entorno de ejecución que controlará la excepción.

---

Yellowpaper Ethereum: https://ethereum.github.io/yellowpaper/paper.pdf

---

## Demo

### Demo 1 - Simple disasm

Añade a la pila dos números de un byte y realiza la suma de ambos.

```
bytecode: 6005600401
```

Disasm del bytecode:
```
$ echo "6005600401" >> add1 && evm disasm add1 
6005600401
000000: PUSH1 0x05
000002: PUSH1 0x04
000004: ADD
```

- ```PUSH1``` añade a la pila un elemento de un byte. En este caso ```0x04``` y ```0x05```.
- ```ADD``` realiza la suma de los dos elementos añadidos a la pila.

EVM debug:
```
$ evm --debug --codefile add1 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

PUSH1           pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005

ADD             pc=00000004 gas=9999999994 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  0000000000000000000000000000000000000000000000000000000000000005

STOP            pc=00000005 gas=9999999991 cost=0
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000009
```

---

### Demo 2 - Simple disasm v.2

Análogo a la Demo 1 pero en este caso la suma se realiza de dos valores de dos bytes.

```
bytecode: 61123461432101
```

Disasm del bytecode:
```
$ echo "61123461432101" >> add2 && evm disasm add2
61123461432101
000000: PUSH2 0x1234
000003: PUSH2 0x4321
000006: ADD
```

- ```PUSH2``` añade a la pila un elemento de dos bytes. En este caso ```0x1234``` y ```0x4321```.
- ```ADD``` realiza la suma de los dos elementos añadidos a la pila.

EVM debug:
```
$ evm --debug --codefile add2 run
#### TRACE ####
PUSH2           pc=00000000 gas=10000000000 cost=3

PUSH2           pc=00000003 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000001234

ADD             pc=00000006 gas=9999999994 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000004321
00000001  0000000000000000000000000000000000000000000000000000000000001234

STOP            pc=00000007 gas=9999999991 cost=0
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000005555
```
---

### Demo 3 - Paso de parámetros como atributos de función.

Suma de tres elementos de 32 bytes pasados como input.

input:
```0000000000000000000000000000000000000000000000000000000000000005```
```0000000000000000000000000000000000000000000000000000000000000004```
```0000000000000000000000000000000000000000000000000000000000000003```

```
bytecode: 6000356020356040350101
```

Disasm del bytecode:

```
$ echo "6000356020356040350101" >> add3 && evm disasm add3
6000356020356040350101
000000: PUSH1 0x00
000002: CALLDATALOAD
000003: PUSH1 0x20
000005: CALLDATALOAD
000006: PUSH1 0x40
000008: CALLDATALOAD
000009: ADD
000010: ADD
```
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```CALLDATALOAD``` desde donde obtener el input.
- ```CALLDATALOAD``` carga en la pila en notación big-endian el input desde donde se le ha indicado anteriormente:
	- ```0x00``` (00 en decimal): Obtiene los primeros 32 bytes del input.
	- ```0x20``` (32 en decimal): Obtiene los siguientes 32 bytes a partir del valor ```0x20``` en hexadecimal.
	- ```0x40``` (64 en decimal): Obtiene los siguientes 32 bytes a partir del valor ```0x40``` en hexadecimal.
- Una vez los tres elementos están almacenado en la pila, la función ```ADD``` realizará la suma de los dos primeros elementos. 
	- Recordad el esquema de la función ```ADD```, saca los dos primeros elementos de la pila y añade el resultado a la misma.
	```python
   		0x01: ['ADD', 2, 1, 3],
	``` 
- Se vuelve a ejecutar la instrucción ADD porque en este caso la suma es de tres elementos, por lo tanto suma el resultado del primer ADD con el tercer input.

EVM debug:
```
$ evm --debug --codefile add3 --input 000000000000000000000000000000000000000000000000000000000000000500000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000003 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

CALLDATALOAD    pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

PUSH1           pc=00000003 gas=9999999994 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005

CALLDATALOAD    pc=00000005 gas=9999999991 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000020
00000001  0000000000000000000000000000000000000000000000000000000000000005

PUSH1           pc=00000006 gas=9999999988 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  0000000000000000000000000000000000000000000000000000000000000005

CALLDATALOAD    pc=00000008 gas=9999999985 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000040
00000001  0000000000000000000000000000000000000000000000000000000000000004
00000002  0000000000000000000000000000000000000000000000000000000000000005

ADD             pc=00000009 gas=9999999982 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000004
00000002  0000000000000000000000000000000000000000000000000000000000000005

ADD             pc=00000010 gas=9999999979 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000007
00000001  0000000000000000000000000000000000000000000000000000000000000005

STOP            pc=00000011 gas=9999999976 cost=0
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000000c
```
---

### Demo 4 - Loop - Ineficiencia.

En este ejemplo se tiene un counter que realiza la cuenta atrás de uno en uno hasta llegar a cero desde el valor que se le pase como input.

En el primer caso tenemos un código ineficiente debido al elevado uso de memoria, almacenando y obteniendo el valor del counter constantemente en la memoria, lo que conlleva un consumo elevado de Gas.

```
bytecode: 6000356000525b600160005103600052600051600657
```

Disasm del bytecode:
```
$ echo "6000356000525b600160005103600052600051600657" >> testloop && evm disasm testloop
6000356000525b600160005103600052600051600657
000000: PUSH1 0x00
000002: CALLDATALOAD
000003: PUSH1 0x00
000005: MSTORE
000006: JUMPDEST
000007: PUSH1 0x01
000009: PUSH1 0x00
000011: MLOAD
000012: SUB
000013: PUSH1 0x00
000015: MSTORE
000016: PUSH1 0x00
000018: MLOAD
000019: PUSH1 0x06
000021: JUMPI
```

- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```CALLDATALOAD``` desde donde obtener el input.
- ```CALLDATALOAD``` carga en la pila en notación big-endian el input desde donde se le ha indicado anteriormente:
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```MSTORE``` dónde almacenar el input obtenido.
- ```MSTORE``` almacena en memoria el valor del input que se encontraba en la pila
- ```JUMPDEST``` es la instrucción para indicar un destino válido de una operación de salto.
- ```PUSH1``` añade el valor ```0x01``` a la pila para indicar cuál será el valor a reducir por el counter (una unidad en este ejemplo).
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar la posición de memoria desde donde obtener el valor almacenado.
- ```MLOAD``` realiza la carga del valor almacenado en memoria en la posición indicada (```0x00```).
- ```SUB``` realiza la resta entre el valor obtenido y el valor que se añadió anteriormente. La operación consume los dos valores y añade a la pila el resultado.
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```MSTORE``` dónde almacenar el resultado de ```SUB```.
- ```MSTORE``` almacena en memoria el valor del resultado de ```SUB```.
- ```PUSH1``` añade el valor ```0x00``` para indicar la posición de memoria desde donde obtener el valor almacenado.
- ```MLOAD``` realiza la carga del valor almacenado en memoria en la posición indicada (```0x00```).
- ```PUSH1``` añade el valor ```0x06``` a la pila para indicar a la instrucción ```JUMPI``` el destino en caso de que la ejecución sea satisfactoria.
- ```JUMPI``` realiza un salto a la posición ```0x06``` en caso de que el valor obtenido desde la memoria sea distinto de cero. Esta instruccción consume los dos valores, el destino de la instrucción de salto y el valor a comprobar.
- A partir de aquí el código es recursivo y ejecutará las mismas instrucciones hasta que la condición se cumpla en ```JUMPI``` y el counter esté a cero, con lo cual el programa se detendrá habiendo completado su ejecución.

EVM debug:
```
$ evm --debug --codefile testloop --input 0000000000000000000000000000000000000000000000000000000000000003 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

CALLDATALOAD    pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

PUSH1           pc=00000003 gas=9999999994 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003

MSTORE          pc=00000005 gas=9999999991 cost=6
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000003
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

JUMPDEST        pc=00000006 gas=9999999985 cost=1
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

PUSH1           pc=00000007 gas=9999999984 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

PUSH1           pc=00000009 gas=9999999981 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

MLOAD           pc=00000011 gas=9999999978 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

SUB             pc=00000012 gas=9999999975 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

PUSH1           pc=00000013 gas=9999999972 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

MSTORE          pc=00000015 gas=9999999969 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000002
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03  |................|

PUSH1           pc=00000016 gas=9999999966 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

MLOAD           pc=00000018 gas=9999999963 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

PUSH1           pc=00000019 gas=9999999960 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

JUMPI           pc=00000021 gas=9999999957 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000006
00000001  0000000000000000000000000000000000000000000000000000000000000002
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

JUMPDEST        pc=00000006 gas=9999999947 cost=1
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

PUSH1           pc=00000007 gas=9999999946 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

PUSH1           pc=00000009 gas=9999999943 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

MLOAD           pc=00000011 gas=9999999940 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

SUB             pc=00000012 gas=9999999937 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

PUSH1           pc=00000013 gas=9999999934 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

MSTORE          pc=00000015 gas=9999999931 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 02  |................|

PUSH1           pc=00000016 gas=9999999928 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

MLOAD           pc=00000018 gas=9999999925 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

PUSH1           pc=00000019 gas=9999999922 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

JUMPI           pc=00000021 gas=9999999919 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000006
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

JUMPDEST        pc=00000006 gas=9999999909 cost=1
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

PUSH1           pc=00000007 gas=9999999908 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

PUSH1           pc=00000009 gas=9999999905 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

MLOAD           pc=00000011 gas=9999999902 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

SUB             pc=00000012 gas=9999999899 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

PUSH1           pc=00000013 gas=9999999896 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

MSTORE          pc=00000015 gas=9999999893 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 01  |................|

PUSH1           pc=00000016 gas=9999999890 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

MLOAD           pc=00000018 gas=9999999887 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

PUSH1           pc=00000019 gas=9999999884 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

JUMPI           pc=00000021 gas=9999999881 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000006
00000001  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

STOP            pc=00000022 gas=9999999871 cost=0
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

El anterior código es ineficiente en el uso de la memoria. Se puede mantener el elemento en la pila en lugar de estar utilizando la memoria constantemente. Además la cantidad de Gas consumida en el código anterior es más elevada que el código de a continuación como se puede comprobar.

```
bytecode: 6000355b6001900380600357
```
En este ejemplo, únicamente se utiliza la pila para la ejeucicón del counter.

- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```CALLDATALOAD``` desde donde obtener el input.
- ```CALLDATALOAD``` carga en la pila en notación big-endian el input desde donde se le ha indicado anteriormente:
- ```JUMPDEST``` es la instrucción para indicar un destino válido de una operación de salto.
- ```PUSH1``` añade el valor ```0x01``` a la pila para indicar cuál será el valor a reducir por el counter (una unidad en este ejemplo).
- ```SWAP1``` es la instrucción para intercambiar la posición de los dos primeros elementos almacenados en la pila.
- ```SUB``` realiza la resta entre el valor obtenido y el valor que se añadió anteriormente. La operación consume los dos valores y añade a la pila el resultado.
- ```DUP1``` realiza un duplicado del valor resultante ```0x00...02``` porque posteriormente la instrucción ```JUMPI``` consumirá uno de ellos para la comprobación, permitiendo que no haya que rescatar el valor actual del contador en la siguiente instrucción.
- ```PUSH1``` añade el valor ```0x03``` a la pila para indicar a la instrucción ```JUMPI``` el destino en caso de que la ejecución sea satisfactoria.
- ```JUMPI``` realiza un salto a la posición ```0x03``` en caso de que el valor resultante sea distinto de cero. Esta instruccción consume los dos valores, el destino de la instrucción de salto y el valor a comprobar.
- A partir de aquí el código es recursivo y ejecutará las mismas instrucciones hasta que la condición se cumpla en ```JUMPI``` y el counter esté a cero, con lo cual el programa se detendrá habiendo completado su ejecución.

```
$ echo "6000355b6001900380600357" >> testloop2 && evm disasm testloop2
6000355b6001900380600357
000000: PUSH1 0x00
000002: CALLDATALOAD
000003: JUMPDEST
000004: PUSH1 0x01
000006: SWAP1
000007: SUB
000008: DUP1
000009: PUSH1 0x03
000011: JUMPI
```
EVM debug:
```
$ evm --debug --codefile testloop2 --input 0000000000000000000000000000000000000000000000000000000000000003 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

CALLDATALOAD    pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

JUMPDEST        pc=00000003 gas=9999999994 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003

PUSH1           pc=00000004 gas=9999999993 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003

SWAP1           pc=00000006 gas=9999999990 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000003

SUB             pc=00000007 gas=9999999987 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000008 gas=9999999984 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000009 gas=9999999981 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000002

JUMPI           pc=00000011 gas=9999999978 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000002
00000002  0000000000000000000000000000000000000000000000000000000000000002

JUMPDEST        pc=00000003 gas=9999999968 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000004 gas=9999999967 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

SWAP1           pc=00000006 gas=9999999964 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000002

SUB             pc=00000007 gas=9999999961 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000008 gas=9999999958 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

PUSH1           pc=00000009 gas=9999999955 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

JUMPI           pc=00000011 gas=9999999952 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000001
00000002  0000000000000000000000000000000000000000000000000000000000000001

JUMPDEST        pc=00000003 gas=9999999942 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

PUSH1           pc=00000004 gas=9999999941 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

SWAP1           pc=00000006 gas=9999999938 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

SUB             pc=00000007 gas=9999999935 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000008 gas=9999999932 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```
---

### Demo 5 - Input de longitud variable en lugar de 32 bytes - (Operador de desplazamiento)

Para tener que evitar introducir elementos de 32 bytes para cada operación, se recurre a la función CALLDATASIZE. Esta función sirve para obtener el tamaño de los datos de entrada. Sustituye a CALLDATALOAD que obtiene los elementos en formato big-endian.

Por tanto, la finalidad es que 0000000000000000000000000000000000000000000000000000000000000003 pase a ser 03, simulando una operación de desplazamiento común. Matemáticamente, es dividir el input entre 256^(32-L) donde L es el tamaño de los datos de entrada.

La función es la misma que en códigos anteriores, un counter que va reduciendo su valor hasta quedarse a cero.

```
bytecode: 366020036101000a600035045b6001900380600c57
```

Disasm del byteccode:
```
$ echo  366020036101000a600035045b6001900380600c57 >> shift && evm disasm shift
366020036101000a600035045b6001900380600c57
000000: CALLDATASIZE
000001: PUSH1 0x20
000003: SUB
000004: PUSH2 0x0100
000007: EXP
000008: PUSH1 0x00
000010: CALLDATALOAD
000011: DIV
000012: JUMPDEST
000013: PUSH1 0x01
000015: SWAP1
000016: SUB
000017: DUP1
000018: PUSH1 0x0c
000020: JUMPI
```
- CALLDATASIZE obtiene el tamaño del input, en este caso el tamaño del input es de un byte.
- ```PUSH1``` añade el valor ```0x20``` a la pila para indicar en hexadecimal la cantidad de bytes totales. Esto es el valor decimal 32 que forma parte del exponente de la fórmula anterior.
- ```SUB``` realiza la resta entre el valor añadido a la pila y el tamaño del input obtenido. La operación consume los dos valores y añade a la pila el resultado.
- ```PUSH2``` añade el valor ```0x100``` a la pila para indicar en hexadecimal la base de la fórmula anterior.
- ```EXP``` realiza una exponencial con los valores almacenados en la pila. Será ```256^(32-1)```. Consume los dos valores y añade el resultado a la pila.
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar a la instrucción ```CALLDATALOAD``` desde donde obtener el input.
- ```CALLDATALOAD``` carga en la pila en notación big-endian el input desde donde se le ha indicado anteriormente:
	- ```0300000000000000000000000000000000000000000000000000000000000000```
- ```DIV``` realiza la operación de división entre el input y el valor resultante de la exponencial. Obteniendo como resultado un desplazamiento del input hacila derecha 31 bytes.
	- ```0000000000000000000000000000000000000000000000000000000000000003```
- ```JUMPDEST``` es la instrucción para indicar un destino válido de una operación de salto.
- ```PUSH1``` añade el valor ```0x01``` a la pila para indicar cuál será el valor a reducir por el counter (una unidad en este ejemplo).
- ```SWAP1``` es la instrucción para intercambiar la posición de los dos primeros elementos almacenados en la pila.
- ```SUB``` realiza la resta entre el valor añadido y el valor resultante de la división. La operación consume los dos valores y añade a la pila el resultado.
- ```DUP1``` realiza un duplicado del valor resultante ```0x00...02``` porque posteriormente la instrucción ```JUMPI``` consumirá uno de ellos para la comprobación, permitiendo que no haya que rescatar el valor actual del contador en la siguiente instrucción.
- ```PUSH1``` añade el valor ```0x0c``` a la pila para indicar a la instrucción ```JUMPI``` el destino en caso de que la ejecución sea satisfactoria.
- ```JUMPI``` realiza un salto a la posición ```0x0c``` en caso de que el valor resultante sea distinto de cero. Esta instruccción consume los dos valores, el destino de la instrucción de salto y el valor a comprobar.
- A partir de aquí el código es recursivo y ejecutará las mismas instrucciones hasta que la condición se cumpla en ```JUMPI``` y el counter esté a cero, con lo cual el programa se detendrá habiendo completado su ejecución.

EVM debug:
```
$ evm --debug --codefile shift --input 03 run
#### TRACE ####
CALLDATASIZE    pc=00000000 gas=10000000000 cost=2

PUSH1           pc=00000001 gas=9999999998 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

SUB             pc=00000003 gas=9999999995 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000020
00000001  0000000000000000000000000000000000000000000000000000000000000001

PUSH2           pc=00000004 gas=9999999992 cost=3
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000001f

EXP             pc=00000007 gas=9999999989 cost=60
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000100
00000001  000000000000000000000000000000000000000000000000000000000000001f

PUSH1           pc=00000008 gas=9999999929 cost=3
Stack:
00000000  0100000000000000000000000000000000000000000000000000000000000000

CALLDATALOAD    pc=00000010 gas=9999999926 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0100000000000000000000000000000000000000000000000000000000000000

DIV             pc=00000011 gas=9999999923 cost=5
Stack:
00000000  0300000000000000000000000000000000000000000000000000000000000000
00000001  0100000000000000000000000000000000000000000000000000000000000000

JUMPDEST        pc=00000012 gas=9999999918 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003

PUSH1           pc=00000013 gas=9999999917 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003

SWAP1           pc=00000015 gas=9999999914 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000003

SUB             pc=00000016 gas=9999999911 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000003
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000017 gas=9999999908 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000018 gas=9999999905 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000002

JUMPI           pc=00000020 gas=9999999902 cost=10
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000000c
00000001  0000000000000000000000000000000000000000000000000000000000000002
00000002  0000000000000000000000000000000000000000000000000000000000000002

JUMPDEST        pc=00000012 gas=9999999892 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000013 gas=9999999891 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

SWAP1           pc=00000015 gas=9999999888 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000002

SUB             pc=00000016 gas=9999999885 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000017 gas=9999999882 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

PUSH1           pc=00000018 gas=9999999879 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

JUMPI           pc=00000020 gas=9999999876 cost=10
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000000c
00000001  0000000000000000000000000000000000000000000000000000000000000001
00000002  0000000000000000000000000000000000000000000000000000000000000001

JUMPDEST        pc=00000012 gas=9999999866 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

PUSH1           pc=00000013 gas=9999999865 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

SWAP1           pc=00000015 gas=9999999862 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

SUB             pc=00000016 gas=9999999859 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000017 gas=9999999856 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

PUSH1           pc=00000018 gas=9999999853 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000000

JUMPI           pc=00000020 gas=9999999850 cost=10
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000000c
00000001  0000000000000000000000000000000000000000000000000000000000000000
00000002  0000000000000000000000000000000000000000000000000000000000000000

STOP            pc=00000021 gas=9999999840 cost=0
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
```
---

## Demo 6 - MLOAD-MSTORE vs SLOAD-SSTORE

Anteriormente se han visto dos ejemplos de almacenamiento, tanto en memoria como en pila. Esta memoria no tiene límite de acceso, se irá generando en función de la solicitud de acceso a ella. Además si nos fijamos en los opcodes asociados a memoria (MLOAD, MSTORE) el coste de gas es muy reducido en comparación con almacenamiento (SLOAD, SSTORE).

### Coste de acceso a memoria

Acceso a la ubicación 0x0100 de memoria con un coste muy elevado inicialmente. Volver a acceder a esa posición tiene un coste mínimo.

```
bytecode: 6101005151
```
Disasm del byteccode:
```
$ echo  6101005151 >> memory && evm disasm memory
6101005151
000000: PUSH2 0x100
000003: MLOAD
000004: MLOAD
```
- ```PUSH2``` añade el valor ```0x0100``` a la pila para indicar la posición de memoria a la que se accederña mediante MLOAD.
- ```MLOAD``` realiza la carga del valor almacenado en memoria en la posición indicada (```0x0100```). Inicialmente es cero.
- ```MLOAD``` realiza de nuevo la carga del valor almacenado en memoria en la posición indicada (```0x0100```).

Este coste tan reducido la segunda vez que se ejecuta ```MLOAD``` es debido a que el tamaño de la memoria no ha variado entre una instrucción y otra, y por lo tanto no ha sido necesario acceder a posiciones más altas obligando a crecer el tamaño de la memoria.

EVM debug:
```
$ evm --debug --code 6101005151 run
#### TRACE ####
PUSH2           pc=00000000 gas=10000000000 cost=3

MLOAD           pc=00000003 gas=9999999997 cost=30
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000100
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000090  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000100  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000110  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

MLOAD           pc=00000004 gas=9999999967 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000090  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000100  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000110  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

STOP            pc=00000005 gas=9999999964 cost=0
```

### Coste de almacenamiento

Almacenar por primera vez un valor tiene un coste muy elevado, Gas = 20000. Este almacenamiento es sustituir el cero inicial por el valor que le indicamos.

Si a esa misma posición se le quiere actualizar el valor, el coste será de 5000 unidades de Gas, algo muy elevado también pero no tanto en comparación con el primer acceso al almacenamiento.

```
bytecode: 60026000556001600055
```

Disasm del byteccode:
```
$ echo  60026000556001600055 >> memory && evm disasm memory
60026000556001600055
000000: PUSH1 0x02
000002: PUSH1 0x00
000004: SSTORE
000005: PUSH1 0x01
000007: PUSH1 0x00
000009: SSTORE
```

- ```PUSH1``` añade el valor ```0x02``` a la pila que será el valor que se almacenará.
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar la posición de almacenamiento donde se almacenará el valor.
- ```SSTORE``` almacena el valor en la posición que se le ha indicado.
- ```PUSH1``` añade el valor ```0x01``` a la pila que será el valor que se almacenará.
- ```PUSH1``` añade el valor ```0x00``` a la pila para indicar la posición de almacenamiento donde se almacenará el valor.
- ```SSTORE``` almacena el valor en la posición que se le ha indicado.

Se puede ver que al realizar el primer almacenamiento es mucho más costoso, esto es debido a que se modifica el valor inicial (```0x00```) por ```0x02```. En cambio al hacerlo por segunda vez modificado el valor ```0x02``` por ```0x01``` tiene un coste elevado pero más reducido.

EVM debug:

```
$ evm --debug --code 60026000556001600055 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

PUSH1           pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

SSTORE          pc=00000004 gas=9999999994 cost=20000
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000002
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000005 gas=9999979994 cost=3
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000007 gas=9999979991 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000002

SSTORE          pc=00000009 gas=9999979988 cost=5000
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000001
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000001

STOP            pc=00000010 gas=9999974988 cost=0
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000001
```

---

## Demo 7 - Excepciones

### Out-of-gas (OOG)
Para que se complete la ejecución del programa sería necesaria una cantidad de Gas igual a 9 como se puede observar en la tabla de opcodes. 

```
cost(PUSH1) + cost(PUSH1) + cost(ADD) = 9 > 8 
```

Fijamos una cantidad de Gas a la EVM de 8 y observamos cómo la EVM lanza la excepción:

```
$ evm --debug --gas 8 --code 6005600401 run
#### TRACE ####
PUSH1           pc=00000000 gas=8 cost=3

PUSH1           pc=00000002 gas=5 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005

ADD             pc=00000004 gas=2 cost=3 ERROR: out of gas
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  0000000000000000000000000000000000000000000000000000000000000005

#### LOGS ####
0x error: out of gas
```
### Invalid opcode
En caso de que durante la ejecución del programa se encuentre con un opcode que la EVM no reconozca saltará excepción.
```
$ evm --debug --code 6005600447 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

PUSH1           pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005

Missing opcode 0x47pc=00000004 gas=9999999994 cost=3 ERROR: invalid opcode 0x47
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  0000000000000000000000000000000000000000000000000000000000000005

#### LOGS ####
0x error: invalid opcode 0x47
```
### Invalid jump destination

Reutilizamos el código anterior, a la hora de ejecutar JUMPI no encuentra un JUMPDEST y salta excepción.

```Bytecode correcto: 6000355b6001900380600357```

Modificamos la ruta del JUMPI para que salte excepción:
```
$ evm --debug --code 6000355b6001900380600257 --input 0000000000000000000000000000000000000000000000000000000000000002 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

CALLDATALOAD    pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000

JUMPDEST        pc=00000003 gas=9999999994 cost=1
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

PUSH1           pc=00000004 gas=9999999993 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002

SWAP1           pc=00000006 gas=9999999990 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000002

SUB             pc=00000007 gas=9999999987 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000001

DUP1            pc=00000008 gas=9999999984 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001

PUSH1           pc=00000009 gas=9999999981 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  0000000000000000000000000000000000000000000000000000000000000001

JUMPI           pc=00000011 gas=9999999978 cost=10
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  0000000000000000000000000000000000000000000000000000000000000001
00000002  0000000000000000000000000000000000000000000000000000000000000001

#### LOGS ####
0x error: invalid jump destination (CALLDATALOAD) 2
```
### Stack underflow
Puede suceder de múltiples formas. En este caso, se intenta eliminar un valor de la pila cuando ya está vacía, provocando la excepción.

```
$ evm --debug --code 60055050 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

POP             pc=00000002 gas=9999999997 cost=2
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005

POP             pc=00000003 gas=9999999995 cost=2 ERROR: stack underflow (0 <=> 1)

#### LOGS ####
0x error: stack underflow (0 <=> 1)
```
---

## Demo 8 - Análisis Smart Contract desplegado

```
pragma solidity ^0.4.18;

contract Addition{
	int x;
	function add(int a, int b){
		x = a + b;
	}
}
```
Fichero generado a partir del compilador de Solidity que contiene el bytecode simulando que ha sido desplegado.
```
Addition.bin-runtime:
606060405260e060020a6000350463a5f3c23b8114601a575b005b60243560043501600055601856
```

Disasm del bytecode:
```
$ evm disasm Addition.bin-runtime 
606060405260e060020a6000350463a5f3c23b8114601a575b005b60243560043501600055601856
000000: PUSH1 0x60
000002: PUSH1 0x40
000004: MSTORE
000005: PUSH1 0xe0
000007: PUSH1 0x02
000009: EXP
000010: PUSH1 0x00
000012: CALLDATALOAD
000013: DIV
000014: PUSH4 0xa5f3c23b
000019: DUP2
000020: EQ
000021: PUSH1 0x1a
000023: JUMPI
000024: JUMPDEST
000025: STOP
000026: JUMPDEST
000027: PUSH1 0x24
000029: CALLDATALOAD
000030: PUSH1 0x04
000032: CALLDATALOAD
000033: ADD
000034: PUSH1 0x00
000036: SSTORE
000037: PUSH1 0x18
000039: JUMP
```

Obtención de identificadores de funciones para posteriormente simular función "add".
```
$ solc --hashes add.sol
======= add.sol:Addition =======
Function signatures: 
a5f3c23b: add(int256,int256)
```

En este ejemplo, antes de llegar a la parte del código que nosotros hemos implementado,

```
000027: PUSH1 0x24
000029: CALLDATALOAD
000030: PUSH1 0x04
000032: CALLDATALOAD
000033: ADD
000034: PUSH1 0x00
000036: SSTORE
```
hay una comprobación que realiza el programa para verificar si el input introducido es igual que el identificador de la función, que serían las instrucciones anteriores a la posición ```000027```, obviando el ```JUMPDEST```.

En este caso se informa a ```CALLDATALOAD``` que debe realizar la carga del input a partir de la posición ```0x04``` y ```0x24```, porque los cuatro bytes iniciales se corresponden con el identificador de la función (```a5f3c23b```).

Las tres primeras instrucciones se corresponden con el puntero de memoria libre (*free memory pointer*). Existen operaciones en Solidity que pueden requerir una memoria temporal mayor de 64 bytes y no entren en el *scratch space*. Estas operaciones se colocarán en el puntero de memoria libre, pero debido a reducido tiempo de vida, este puntero no se actualiza.

Continuando con el programa, se realiza la comprobación comentada anteriormente. Esto se realiza a través del operador desplazamiento visto anteriormente, en este caso hay una variación en el desplazamiento de cuatro bytes. Por lo tanto la fórmula resultante es: ```256^(32-4)``` 

Llama la atención entonces que la operación exponencial que se realiza en la pila sea: ```2^224```. Esto es debido al compilador, pero se puede comprobar que el resultado es exactamente el mismo y el desplazamiento se ejecutará correctamente.

Una vez realizada la operación desplazamiento, mediante la instrucción ```EQ``` se comprueba que efectivamente el input corresponde con el identificador de la función ```add(...)```. Posteriormente, se realiza un salto en caso satisfactorio hacia la instrucción donde comenzaría el código que hemos implementado en nuestro contrato inteligente.

EVM debug:
```
$ evm --debug --codefile Addition.bin-runtime --input a5f3c23b00000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000000000000000000000000000000000000004 run
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

PUSH1           pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000060

MSTORE          pc=00000004 gas=9999999994 cost=12
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000040
00000001  0000000000000000000000000000000000000000000000000000000000000060
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

PUSH1           pc=00000005 gas=9999999982 cost=3
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000007 gas=9999999979 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000000000e0
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

EXP             pc=00000009 gas=9999999976 cost=60
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000002
00000001  00000000000000000000000000000000000000000000000000000000000000e0
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000010 gas=9999999916 cost=3
Stack:
00000000  0000000100000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

CALLDATALOAD    pc=00000012 gas=9999999913 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000100000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

DIV             pc=00000013 gas=9999999910 cost=5
Stack:
00000000  a5f3c23b00000000000000000000000000000000000000000000000000000000
00000001  0000000100000000000000000000000000000000000000000000000000000000
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH4           pc=00000014 gas=9999999905 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

DUP2            pc=00000019 gas=9999999902 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

EQ              pc=00000020 gas=9999999899 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
00000002  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000021 gas=9999999896 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000001
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

JUMPI           pc=00000023 gas=9999999893 cost=10
Stack:
00000000  000000000000000000000000000000000000000000000000000000000000001a
00000001  0000000000000000000000000000000000000000000000000000000000000001
00000002  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

JUMPDEST        pc=00000026 gas=9999999883 cost=1
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000027 gas=9999999882 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

CALLDATALOAD    pc=00000029 gas=9999999879 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000024
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000030 gas=9999999876 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

CALLDATALOAD    pc=00000032 gas=9999999873 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000004
00000001  0000000000000000000000000000000000000000000000000000000000000004
00000002  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

ADD             pc=00000033 gas=9999999870 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000005
00000001  0000000000000000000000000000000000000000000000000000000000000004
00000002  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

PUSH1           pc=00000034 gas=9999999867 cost=3
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000009
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|

SSTORE          pc=00000036 gas=9999999864 cost=20000
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000000
00000001  0000000000000000000000000000000000000000000000000000000000000009
00000002  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000009

PUSH1           pc=00000037 gas=9999979864 cost=3
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000009

JUMP            pc=00000039 gas=9999979861 cost=8
Stack:
00000000  0000000000000000000000000000000000000000000000000000000000000018
00000001  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000009

JUMPDEST        pc=00000024 gas=9999979853 cost=1
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000009

STOP            pc=00000025 gas=9999979852 cost=0
Stack:
00000000  00000000000000000000000000000000000000000000000000000000a5f3c23b
Memory:
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 60  |...............`|
Storage:
0000000000000000000000000000000000000000000000000000000000000000: 0000000000000000000000000000000000000000000000000000000000000009
```
