# evm_meetup

## Intro

La máquina virtual Ethereum (EVM) es el entorno de tiempo de ejecución (runtime) para contratos inteligentes en Ethereum. Se encuentra aislado, lo que significa que el código que se ejecuta dentro de la EVM no tiene acceso a la red, sistema de archivos u otros procesos de la blockchain.

Los contratos viven en la cadena de bloques en un formato binario específico de Ethereum (código de bytes EVM). Sin embargo, los contratos generalmente se escriben en un lenguaje de alto nivel de Ethereum, se compilan en código de bytes usando un compilador de EVM y finalmente se cargan en la cadena de bloques utilizando un cliente de Ethereum. Los contratos inteligentes pueden tener acceso limitado a otros contratos inteligentes.

---

Proporcionar seguridad y ejecución de código no confiable en ordenadores de todo el mundo.

- Prevenir denegación de servicios (DDoS)
- Asegurar que programas no tienen acceso al estado de otros
- Asegurar que no haya interferencias

Para hacer todo esto de una forma segura se imponen las siguientes restricciones:
- Cada paso computacional que se realice en la ejecución de un programa debe ser pagado por adelantado (Gas). --> DDoS protection

- Los programas sólo interactuan a través de la transmisión de matrices de datos únicas. --> No acceden a otros programas.

- La ejecución de programas es en modo sandbox. Un programa puede alterar su propio estado y acceder al mismo, además puede lanzar la ejecución de otro programa, pero nada más. --> No accede al estado de otros programas.

- La ejecución del programa es completamente determinista y por lo tanto produce transiciones de estado idénticas para un estado de comienzo idntico.

---

Cada nodo contiene su propia implementación de la EVM capaz de ejecutar las mismas operaciones, lo que facilita la labor de desarrolladores a la hora de tener un entorno de pruebas.

Es una puerta de acceso a la creación de smart contracts en Solidity, pero también la EVM se ha implementado en Python, Ruby, C++ y otros códigos.

------------------------

Interpretación de valores binarios de 256-bits como enteros. Cuando un dato de 256-bits se convierte hacia o desde una address o hash de 160-bits, se conservan los 20 bytes más a la derecha mientras que los otros 12 bytes se descartan o se rellenan con ceros.

Aclaración big endian
13 --> 0x3133 (como texto)
trece --> 0x74726563650d0a
MSB to LSB

---

Se podría decir que la EVM es una cuasi máquina de Turing; "cuasi" por el hecho de que la computación está intrínsecamente limitada por un factor, el gas, que limita la cantidad total de cálculo.

---

Basics

Arquitectura basada en pila.
Tamaño de palabra de 256-bits. --> Tamaño de elemento de pila 256-bits
El modelo de memoria es una matriz de bytes (word-addressed byte array)
Basado para facilitar el esquema la función keccack-256 y cálculos de curva elíptica.
Tamaño máximo de la pila: 1024.
Modelo de almacenamiento independiente: es una matriz de palabras direccionables (word addressable word array)

---

A diferencia de la memoria, que es volátil, el almacenamiento no es volátil y se mantiene como parte del estado del sistema. Todas las ubicaciones en almacenamiento y memoria están bien definidas inicialmente como cero.

La máquina no sigue la arquitectura estándar de la máquina de von Neumann

En lugar de almacenar el código del programa en una memoria o almacenamiento accesible, se almacena en una ROM virtual con la cual sólo se puede interactuar a través de una instrucción especializada.

---

Pueden suceder excepciones por varias razones, desbordamientos de pila, instrucciones no válidas. Una excepción bastante común suele ser "out-of-gas (OOG)" que no deja los cambios de estado intactos.

La máquina se detiene inmediatamente e informa el problema al procesador de transacciones o, recursivamente, al entorno agente de ejecución (ya sea el procesador de transacciones o, recursivamente, el entorno de ejecución que controlará la excepción.

---

## Demo

---

La computación en la EVM se realiza utilizando un lenguaje de códigos de bytes basado en pila que es como un cruce entre Bitcoin Script, ensamblaje tradicional y Lisp (la parte de Lisp se debe a la funcionalidad recursiva de envío de mensajes). Un programa en EVM es una secuencia de códigos de operación, como esta:

La EVM es una máquina virtual basada en pila con una matriz de bytes de memoria y almacenamiento de clave-valor. Los elementos en la pila son palabras de 32 bytes, y todas las claves y valores almacenados son de 32 bytes. Hay más de 100 códigos de operación, divididos en categorías delineadas en múltiplos de 16. Se puede consultar aquí:


```python
# schema: [opcode, ins, outs, gas]

[opcode,
elementos que entran a la pila,
elementos que salen de la pila,
costa en gas]

opcodes = {

    # arithmetic
    0x00: ['STOP', 0, 0, 0],
    0x01: ['ADD', 2, 1, 3],
    0x02: ['MUL', 2, 1, 5],
    0x03: ['SUB', 2, 1, 3],
    0x04: ['DIV', 2, 1, 5],
    0x05: ['SDIV', 2, 1, 5],
    0x06: ['MOD', 2, 1, 5],
    0x07: ['SMOD', 2, 1, 5],
    0x08: ['ADDMOD', 3, 1, 8],
    0x09: ['MULMOD', 3, 1, 8],
    0x0a: ['EXP', 2, 1, 10],
    0x0b: ['SIGNEXTEND', 2, 1, 5],

    # boolean
    0x10: ['LT', 2, 1, 3],
    0x11: ['GT', 2, 1, 3],
    0x12: ['SLT', 2, 1, 3],
    0x13: ['SGT', 2, 1, 3],
    0x14: ['EQ', 2, 1, 3],
    0x15: ['ISZERO', 1, 1, 3],
    0x16: ['AND', 2, 1, 3],
    0x17: ['OR', 2, 1, 3],
    0x18: ['XOR', 2, 1, 3],
    0x19: ['NOT', 1, 1, 3],
    0x1a: ['BYTE', 2, 1, 3],

    # crypto
    0x20: ['SHA3', 2, 1, 30],
    
    # contract context
    0x30: ['ADDRESS', 0, 1, 2],
    0x31: ['BALANCE', 1, 1, 20],
    0x32: ['ORIGIN', 0, 1, 2],
    0x33: ['CALLER', 0, 1, 2],
    0x34: ['CALLVALUE', 0, 1, 2],
    0x35: ['CALLDATALOAD', 1, 1, 3],
    0x36: ['CALLDATASIZE', 0, 1, 2],
    0x37: ['CALLDATACOPY', 3, 0, 3],
    0x38: ['CODESIZE', 0, 1, 2],
    0x39: ['CODECOPY', 3, 0, 3],
    0x3a: ['GASPRICE', 0, 1, 2],
    0x3b: ['EXTCODESIZE', 1, 1, 20],
    0x3c: ['EXTCODECOPY', 4, 0, 20],

    # blockchain context
    0x40: ['BLOCKHASH', 1, 1, 20],
    0x41: ['COINBASE', 0, 1, 2],
    0x42: ['TIMESTAMP', 0, 1, 2],
    0x43: ['NUMBER', 0, 1, 2],
    0x44: ['DIFFICULTY', 0, 1, 2],
    0x45: ['GASLIMIT', 0, 1, 2],
  
    # storage and execution
    0x50: ['POP', 1, 0, 2],
    0x51: ['MLOAD', 1, 1, 3],
    0x52: ['MSTORE', 2, 0, 3],
    0x53: ['MSTORE8', 2, 0, 3],
    0x54: ['SLOAD', 1, 1, 50],
    0x55: ['SSTORE', 2, 0, 0],
    0x56: ['JUMP', 1, 0, 8],
    0x57: ['JUMPI', 2, 0, 10],
    0x58: ['PC', 0, 1, 2],
    0x59: ['MSIZE', 0, 1, 2],
    0x5a: ['GAS', 0, 1, 2],
    0x5b: ['JUMPDEST', 0, 0, 1],

    # logging
    0xa0: ['LOG0', 2, 0, 375],
    0xa1: ['LOG1', 3, 0, 750],
    0xa2: ['LOG2', 4, 0, 1125],
    0xa3: ['LOG3', 5, 0, 1500],
    0xa4: ['LOG4', 6, 0, 1875],
	
    # arbitrary length storage (proposal for metropolis hardfork)
    0xe1: ['SLOADBYTES', 3, 0, 50],
    0xe2: ['SSTOREBYTES', 3, 0, 0],
    0xe3: ['SSIZE', 1, 1, 50],

    # closures
    0xf0: ['CREATE', 3, 1, 32000],
    0xf1: ['CALL', 7, 1, 40],
    0xf2: ['CALLCODE', 7, 1, 40],
    0xf3: ['RETURN', 2, 0, 0],
    0xf4: ['DELEGATECALL', 6, 0, 40],
    0xff: ['SUICIDE', 1, 0, 0],
}

# push
for i in range(1, 33):
    opcodes[0x5f + i] = ['PUSH' + str(i), 0, 1, 3]

# duplicate and swap
for i in range(1, 17):
    opcodes[0x7f + i] = ['DUP' + str(i), i, i + 1, 3]
    opcodes[0x8f + i] = ['SWAP' + str(i), i + 1, i + 1, 3]
```

---

## Demo

---

### Tools

---

### Demo 1 - Simple disasm

Añade dos números de 1 byte cada uno a la pila y los suma.

```
bytecode: 6005600401
```
Guardar bytecode en archivo para poder ejecutar disasm

```
$ echo "6005600401" >> add1 && evm disasm add1 
6005600401
000000: PUSH1 0x05
000002: PUSH1 0x04
000004: ADD
```
EVM:
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

Consideraciones:
- Si se quieren sumar más de dos elementos --> Recordad esquema
```python
    0x01: ['ADD', 2, 1, 3],
``` 
- Si se añade un elemento de 2 bytes saltará excepción.

---

### Demo 2 - Simple disasm v.2

Añade dos números de 2 byte cada uno a la pila y los suma.

```
bytecode: 61123461432101
```
Guardar bytecode en archivo para poder ejecutar disasm

```
$ echo "61123461432101" >> add2 && evm disasm add2
61123461432101
000000: PUSH2 0x1234
000003: PUSH2 0x4321
000006: ADD
```
EVM:
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

Suma tres elementos pasados como parámetos.

Input tres elementos de 32 bytes (0x00...05, 0x00...04, 0x00...03)

```
bytecode: 6000356020356040350101
```
Guardar bytecode en archivo para poder ejecutar disasm

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
EVM:
```
$ evm --debug --code 6000356020356040350101 --input 000000000000000000000000000000000000000000000000000000000000000500000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000003 run
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









