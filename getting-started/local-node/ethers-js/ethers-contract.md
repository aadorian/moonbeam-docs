---
title: Implementar un contrato
description: Aprenda a desplegar contratos inteligentes basados ​​en Solidity utilizando Moonbeam,  sin modificar y sin realizar cambios usando  un  script sencillo de Ethers.js 
---

# Utilizando Ethers.js para desplegar Contratos Inteligentes en Moonbeam

## Introducción
Esta guía lo guía a través del proceso de uso del compilador Solidity y [ethers.js] (https://docs.ethers.io/) para implementar e interactuar con un contrato inteligente basado en Solidity en un nodo stand-alone de Moonbeam. Dadas las características de compatibilidad con Ethereum de Moonbeam, la biblioteca ethers.js se puede usar directamente con un nodo Moonbeam.

La presente guía asume que se encuentra en el contexto de un nodo Moonbeam local ejecutándose en modo `--dev`. Puede encontrar instrucciones para configurar un nodo local [aquí] Moonbeam (/getting-started/local-node/setting-up-a-node/).

!!! note
Este tutorial se creó utilizando la versión v3 de [Moonbase Alpha] (https://github.com/PureStake/moonbeam/releases/tag/v0.3.0). La plataforma Moonbeam y los componentes de [Frontier] (https://github.com/paritytech/frontier) en los que se basa para la compatibilidad con Ethereum desarrollado en Substrate, aún se encuentran en un desarrollo constante. Los ejemplos de esta guía asumen un entorno basado en Ubuntu 18.04 y deberán adaptarse en consecuencia para MacOS o Windows.

## Verificando Prerequisitos

Siguiendo este [tutorial](/getting-started/local-node/setting-up-a-node/), debería tener un nodo Moonbeam "stand-alone" produciendo bloques en su entorno local, que se ve así:

![Moonbeam local node](/images/etherscontract/ethers-contract-1.png)

Además, necesitamos instalar Node.js (usaremos v15.x) y el administrador de paquetes npm. Puede hacer esto ejecutando en su terminal:

```
curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
```

```
sudo apt install -y nodejs
```

Podemos verificar que todo se instaló correctamente consultando la versión de cada paquete:

```
node -v
```

```
npm -v
```

En el momento de redactar esta guía, las versiones utilizadas fueron 15.2.1 y 7.0.8, respectivamente.

A continuación, podemos crear un directorio para almacenar todos nuestros archivos relevantes (en una ruta separada de los archivos del nodo Moonbeam local) ejecutando:

```
mkdir incrementer && cd incrementer/
```

Y cree un archivo package.json simple:

```
npm init --yes
```

Con el archivo package.json creado, podemos instalar los paquetes ethers.js y el compilador Solidity (corregido en la versión v0.7.4), ejecutando:

```
npm install ethers
```

```pypy
npm install solc@0.7.4
```

Para verificar la versión instalada de ethers.js o el compilador Solidity, puede usar el comando `ls`:
```
npm ls ethers
```

```
npm ls solc
```

En el momento de redactar esta guía, las versiones utilizadas fueron 5.0.22 y 0.7.4 (como se mencionó anteriormente), respectivamente.


De manera similar a nuestro [web3.js contract tutorial](/getting-started/local-node/web3-js/web3-contract/), usaremos esa configuración para este ejemplo, por lo que algunos archivos se verán similares:


-  _Incrementer.sol_: el archivo con nuestro código Solidity
-  _compile.js_: compilará el contrato con el compilador de Solidity
-  _deploy.js_: manejará la implementación en nuestro nodo Moonbeam local
-  _get.js_: hará una llamada al nodo para obtener el valor actual del número
-  _increment.js_: hará una transacción para incrementar el número almacenado en el nodo Moonbeam
-  _reset.js_: la función a llamar que restablecerá el número almacenado a cero

## El archivo de contrato y el script de compilación
Aunque estamos usando una librería diferente, estos dos archivos siguen siendo idénticos. El primero es el contrato inteligente escrito en Solidity y el segundo es el script de compilación, los cuales no requieren las bibliotecas web3.js o ethers.js.

### El archivo del contrato

El contrato que usaremos es un incrementador muy simple (llamado arbitrariamente _Incrementer.sol_, y que puede encontrar [here](/code-snippets/web3-contract-local/Incrementer.sol)). El código de Solidity es el siguiente:

```solidity
--8<-- 'web3-contract-local/Incrementer.sol'
```

Nuestra función `constructor`, que se ejecuta cuando se implementa el contrato, establece el valor inicial de la variable numérica que se almacena en el nodo Moonbeam (el valor predeterminado es 0). La función `increment` agrega` _value` proporcionado al número actual, pero es necesario enviar una transacción ya que esto modifica los datos almacenados. Y, por último, la función `reset` restablece el valor almacenado a cero.

!!! note
   Este contrato es solo un ejemplo simple que no maneja valores "wrapped", y es solo para fines ilustrativos.

### El archivo de compilación


El único propósito del archivo _compile.js_ (nombrado arbitrariamente, y que puede encontrar [aquí] [here](/code-snippets/web3-contract-local/compile.js)), es usar el compilador Solidity para generar el código de bytes y interfaz de nuestro contrato.

Primero, necesitamos cargar los diferentes módulos que usaremos para este proceso. Los módulos _path_ y _fs_ están incluidos por defecto en Node.js (por eso no tuvimos que instalarlo antes).

A continuación, tenemos que leer el contenido del archivo Solidity (en codificación UTF8).

Luego, construimos el objeto de entrada para el compilador Solidity.

Y finalmente, ejecutamos el compilador y extraemos los datos relacionados con nuestro contrato de incremento porque, para este simple ejemplo, eso es todo lo que necesitamos.

```javascript
--8<-- 'web3-contract-local/compile.js'
```

## Deploy del script e interacción con nuestro contrato
En esta sección veremos algunas diferencias entre bibliotecas con respecto al despliegue de contratos, pero con el mismo resultado final.

### El archivo desplegado


El archivo de implementación (que puede encontrar [here](/code-snippets/ethers-contract-local/deploy.js)) se divide en dos subsecciones: la inicialización y el contrato de implementación.


Primero, necesitamos cargar nuestro módulo ethers.js y exportar el archivo _compile.js_, del cual extraeremos el `bytecode` y` abi`.

A continuación, definimos la variable `privKey` como la clave privada de nuestra cuenta génesis, que es donde se almacenan todos los fondos al implementar el nodo Moonbeam local. Recuerdar que en ethers.js debemos proporcionar el prefijo `0x`. Además, tenemos que definir el proveedor pasando la URL de RPC del nodo Moonbeam independiente. Ambos se utilizarán para crear la instancia de `wallet` y acceder a todos sus métodos.

Para desplegar el contrato, primero necesitamos crear una instancia local usando `ethers.ContractFactory (abi, bytecode, wallet)`. Luego,  en una función asíncrona, podemos usar el método `deploy (args)` de esta instancia local, que usa una firma para desplegar el contrato con los argumentos pasados ​​al constructor, para luego  devolver la transacción que contiene la dirección donde se desplegará. Al usar el método `contract.deployed ()`, esperamos que la transacción sea procesada.

```javascript
--8<-- 'ethers-contract-local/deploy.js'
```

!!! note
   El script _deploy.js_ proporciona la dirección del contrato como salida. Esto resulta útil ya que se utiliza para los archivos de interacción del contrato.

### Archivos para interactuar con el contrato
En este apartado repasaremos rápidamente los archivos que interactúan con nuestro contrato, ya sea realizando llamadas o enviando transacciones para modificar su almacenamiento.


Primero, revisemos el archivo _get.js_ (el más simple de todos, que puede encontrar (the simplest of them all, which you can find [here](/code-snippets/ethers-contract-local/get.js))), que recupera el valor actual almacenado en el contrato . Necesitamos cargar nuestro módulo ethers.js y exportar el archivo _compile.js_, del cual extraeremos el `abi`. A continuación, definimos el proveedor para poder acceder a los métodos necesarios para llamar a nuestro contrato.


El siguiente paso es crear una instancia local del contrato usando el comando `ethers.Contract (contractAddress, abi, provider)`. El `contractAddress` se registra en la consola mediante el archivo _deploy.js_. Esta instancia local se está definiendo con el `proveedor`, por lo que solo están disponibles los métodos de solo lectura. Luego, envuelto en una función asíncrona, podemos escribir la llamada del contrato ejecutando `contractInstance.myMethods ()`, donde establecemos el método o función que queremos llamar y proporcionamos las entradas para esta llamada. Esta promesa devuelve los datos que podemos iniciar sesión en la consola. Y por último, ejecutamos nuestra función "get".
```javascript
--8<-- 'ethers-contract-local/get.js'
```

Definamos ahora el archivo para enviar una transacción que agregará el valor proporcionado a nuestro número. El archivo _increment.js_ [here](/code-snippets/ethers-contract-local/increment.js)) es algo diferente al ejemplo anterior, y eso se debe a que aquí estamos modificando los datos almacenados, y para ello, necesitamos enviar una transacción que pague el gas. En el caso de ethers.js, la inicialización es similar a la del script de implementación, donde definimos un proveedor y una billetera. Sin embargo, también se incluyen la dirección del contrato y el valor agregado.



La transacción del contrato comienza creando una instancia local del contrato como antes, pero en este caso pasamos a la `wallet` como firmante, para tener acceso de lectura y escritura a los métodos del contrato.


Luego, usamos el método `increment` de la instancia local, proporcionando el valor para incrementar nuestro número. Ethers.js procederá a crear el objeto de transacción, enviará la transacción y devolverá el recibo. Podemos aprovechar el método `wait ()` de la respuesta de la transacción para esperar a que se procese.


```js
--8<-- 'ethers-contract-local/increment.js'
```
El archivo _reset.js_ (que puede encontrar  [aqui](/code-snippets/ethers-contract-local/reset.js), es casi idéntico al ejemplo anterior. La única diferencia es que necesitamos llamar al método `reset ()` que no toma ninguna entrada. En este caso, estamos configurando manualmente el límite de gas de la transacción en `40000`, ya que el método` calculateGas () `devuelve un valor no válido (algo en lo que estamos trabajando).



```js
--8<-- 'ethers-contract-local/reset.js'
```

## Interactuar con el contrato
Con todos los archivos listos, podemos proceder a implementar nuestro contrato en el nodo Moonbeam local. Para ello, ejecutamos el siguiente comando en el directorio donde se encuentran todos los archivos:

```
node deploy.js
```

Después de una implementación exitosa, debería obtener el siguiente resultado:

![Moonbeam local node](/images/etherscontract/ethers-contract-2.png)

Primero, verifiquemos y confirmemos que el valor almacenado es igual al que pasamos como entrada de del constructor (que era 5), ​​lo hacemos ejecutando:

```
node get.js
```

Con la siguiente salida:

![Moonbeam local node](/images/etherscontract/ethers-contract-3.png)

Luego, podemos usar nuestro archivo incrementador, recuerdar que `_value = 3`. Podemos usar inmediatamente nuestro archivo getter para solicitar el valor después de la transacción:

```
node incrementer.js
```

```
node get.js
```

Con la siguiente salida:

![Moonbeam local node](/images/etherscontract/ethers-contract-4.png)

Por último, podemos restablecer nuestro número utilizando el archivo de restablecimiento:

```
node reset.js
```

```
node get.js
```

Con la siguiente salida:

![Moonbeam local node](/images/etherscontract/ethers-contract-5.png)

## Nosotros queremos saber de ti

Este ejemplo proporciona un contexto sobre cómo puede comenzar a trabajar con Moonbeam y cómo puede probar sus características de compatibilidad con Ethereum, como la biblioteca ethers.js. Estamos interesados ​​en conocer su experiencia siguiendo los pasos de esta guía o su experiencia probando otras herramientas basadas en Ethereum con Moonbeam. No dudes en unirte a nosotros en [Moonbeam Discord here](https://discord.gg/PfpUATX). Nos encantaría escuchar sus comentarios sobre Moonbeam y responder cualquier pregunta que tengas.

