---
title: Configurar un nodo
description: Siga este tutorial para aprender cómo configurar su primer nodo Moonbeam. También aprenderá cómo conectarlo y controlarlo con la GUI de Polkadot JS.
---

# Configuración de un nodo Moonbeam y conexión a la GUI de Polkadot JS

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//p_0OAHSlHNM' frameborder='0' allowfullscreen></iframe></div>
<style>.caption { font-family: Open Sans, sans-serif; font-size: 0.9em; color: rgba(170, 170, 170, 1); font-style: italic; letter-spacing: 0px; position: relative;}</style><div class='caption'>You can find all of the relevant code for this tutorial on the <a href="{{ config.site_url }}resources/code-snippets/">code snippets page</a></div>

## Introducción

Esta guía describe los pasos necesarios para crear un nodo de desarrollo para probar las características de compatibilidad de Ethereum de Moonbeam.


!!! note
     Este tutorial se creó utilizando  {{ networks.development.build_tag }}  [Moonbase Alpha](https://github.com/PureStake/moonbeam/releases/tag/{{ networks.development.build_tag }}). La plataforma Moonbeam y los componentes de [Frontier](https://github.com/paritytech/frontier) en los que se basa para la compatibilidad con Ethereum basada en Substrate aún se encuentran en un desarrollo muy activo.
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

Un nodo de desarrollo Moonbeam es un entorno propio de desarrollo para crear y probar aplicaciones en Moonbeam. Para los desarrolladores de Ethereum, es comparable a Ganache. Le permite comenzar rápida y fácilmente sin la sobrecarga de una relay chain. Es posible activar el nodo con la opción `--sealing` para crear bloques de forma instantánea, manual o en un intervalo personalizado después de recibir las transacciones. De forma predeterminada, se creará un bloqueo cuando se reciba una transacción, que es similar a la función instamine de Ganache.

Si seguimos hasta el final esta guía, podemos tener un nodo de desarrollo Moonbeam ejecutándose en un entorno local, con 10 [cuentas precargadas](#pre-funded-development-accounts), y podremos conectarlo a la GUI predeterminada de Polkadot JS.


Hay dos formas de comenzar a ejecutar un nodo Moonbeam:  utilizando [docker para correr un binario precargado ](#getting-started-with-docker)  o localmente [ instalando y seteando un nodo propio ](#installation-and-setup). El uso de Docker es una forma rápida y conveniente de comenzar, ya que no tendrá que instalar Substrate y todas las dependencias, y también podemos omitir el proceso de construcción del nodo. Requiere de la instación de [Docker](https://docs.docker.com/get-docker/). Por otro lado, si se decide  por el proceso de creación de un nodo propio, podría tardar aproximadamente 30 minutos o más en completarse, según su hardware.


## Introducción a Docker

El uso de Docker nos permite activar un nodo en cuestión de segundos.
Una vez que tengamos Docker instalado, podemos ejecutar el siguiente comando para descargar la imagen correspondiente:
```
docker pull purestake/moonbeam:{{ networks.development.build_tag }}
```

El final del registro de la consola debería verse así:

![Docker - imaged pulled](/images/setting-up-a-node/setting-up-node-1.png)

Una vez que se descarga la imagen de Docker, el siguiente paso es ejecutar la imagen.
Puede ejecutar la imagen de Docker con lo siguiente:

=== "Ubuntu"
    ```
    docker run --rm --name {{ networks.development.container_name }} --network host \
    purestake/moonbeam:{{ networks.development.build_tag }} \
    --dev
    ```

=== "MacOS"
    ```
    docker run --rm --name {{ networks.development.container_name }} -p 9944:9944 -p 9933:9933 \
    purestake/moonbeam:{{ networks.development.build_tag }} \
    --dev --ws-external --rpc-external
    ```

Esto debería activar un nodo de desarrollo Moonbeam en modo de sello instantáneo para pruebas locales, de modo que los bloques se creen instantáneamente a medida que se reciben las transacciones.
Si tenemos éxito, deberíamos ver una salida que muestra un estado inactivo esperando a que se creen los bloques:

![Docker - output shows blocks being produced](/images/setting-up-a-node/setting-up-node-2.png)


Para obtener más información sobre algunas de las banderas y opciones utilizadas en el ejemplo, podemos consultar [Banderas y opciones comunes] (# common-flags-and-options). Si desea ver una lista completa de todos los indicadores, opciones y subcomandos, deberíamos abrir el menú de ayuda ejecutando
```
docker run --rm --name {{ networks.development.container_name }} \
purestake/moonbeam \
--help
```

Para continuar con el tutorial, la siguiente sección no sería necesaria ya que ya hemos activado un nodo con Docker. Podemos saltar a [Connecting Polkadot JS Apps to a Local Moonbeam Node](#connecting-polkadot-js-apps-to-a-local-moonbeam-node).

## Instalación y configuración 

Comenzamos clonando una etiqueta específica del repositorio Moonbeam que podemos encontrar aquí:

[https://github.com/PureStake/moonbeam/](https://github.com/PureStake/moonbeam/)

```
git clone -b {{ networks.development.build_tag }} https://github.com/PureStake/moonbeam
cd moonbeam
```

A continuación, instalamos Substrate y todos sus requisitos previos (incluido Rust) ejecutando:
```
--8<-- 'code/setting-up-local/substrate.md'
```

Una vez que haya seguido todos los procedimientos anteriores, es hora de construir el nodo de desarrollo ejecutando:
```
--8<-- 'code/setting-up-local/build.md'
```


Si  _cargo not found error_ aparece en la terminal, deberíamos agregar manualmente Rust a la ruta de su sistema (o reiniciar el sistema):

```
--8<-- 'code/setting-up-local/cargoerror.md'
```

!!! note
    La construcción inicial llevará un tiempo. Dependiendo de nuestro hardware, debiendo esperar aproximadamente 30 minutos para que se finalice el proceso de compilación.

Así es como debería verse el final de la salida de la compilación:

![End of build output](/images/setting-up-a-node/setting-up-node-3.png)

Luego, ejecutamos el nodo en modo dev usando el siguiente comando:

```
--8<-- 'code/setting-up-local/runnode.md'
```

!!! note
   Para las personas que no están familiarizadas con Substrate, la marca `--dev` es una forma de ejecutar un nodo basado en Substrate en una configuración de desarrollador de un solo nodo con fines de prueba. Puedes aprender más sobre`--dev` en [this Substrate tutorial](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/interact).

Deberíamos ver una salida similar a la siguiente, que muestra un estado inactivo a la espera de que se produzcan bloques:

![Output shows blocks being produced](/images/setting-up-a-node/setting-up-node-4.png)

Para obtener más información sobre algunas de las banderas y opciones utilizadas en el ejemplo, consultar en [Common Flags and Options](#common-flags-and-options). Si se desea ver una lista completa de todos los indicadores, opciones y subcomandos, abrir el menú de ayuda ejecutando:

```
./target/release/moonbeam --help
```
## Conexión de aplicaciones JS de Polkadot a un nodo Moonbeam local

El nodo de desarrollo es un nodo basado en Substrate, por lo que podemos interactuar con él utilizando herramientas de Substrate estándar. Los dos endpoints de RPC proporcionados son:

 - HTTP: `http://127.0.0.1:9933`
 - WS: `ws://127.0.0.1:9944` 

Comencemos por conectarnos con Polkadot JS Apps. Abrimos un navegador: [https://polkadot.js.org/apps/#/explorer](https://polkadot.js.org/apps/#/explorer). Esto abrirá Polkadot JS Apps, que se conecta automáticamente a Polkadot MainNet.

![Polkadot JS Apps](/images/setting-up-a-node/setting-up-node-5.png)

Hacemos clic en la esquina superior izquierda para abrir el menú para configurar las redes y luego naveguamos hacia abajo para abrir el submenú Desarrollo. Allí, podemos alternar la opción "Nodo local", que apunta Polkadot JS Apps a `ws: //127.0.0.1: 9944`. A continuación, seleccionamos el botón Cambiar y el sitio debería conectarse a nuestro nodo de desarrollo Moonbeam.


![Select Local Node](/images/setting-up-a-node/setting-up-node-6.png)

Con Polkadot JS Apps conectadas, podremos ver el nodo de desarrollo Moonbeam esperando que lleguen las transacciones para comenzar a producir bloques.

![Select Local Node](/images/setting-up-a-node/setting-up-node-7.png)

## Consultando el estado de la cuenta

Con el lanzamiento de [Moonbase Alpha v3](https://www.purestake.com/news/moonbeam-network-upgrades-account-structure-to-match-ethereum/), Moonbeam ahora funciona con un formato de cuenta única, que es el H160 de estilo Ethereum y ahora también es compatible con Polkadot JS Apps. Para verificar el saldo de una dirección, simplemente puede importar su cuenta a la pestaña Cuentas. Puede encontrar más información en la sección[Unified Accounts](/learn/unified-accounts/) .
 
Sin embargo, aprovechando las capacidades completas de RPC de Ethereum de Moonbeam, puede usar [MetaMask](/getting-started/local-node/using-metamask/) para comprobar el saldo de esa dirección también. Además, también puede utilizar otras herramientas de desarrollo, como [Remix](/getting-started/local-node/using-remix/) y [Truffle](/getting-started/local-node/using-truffle/).

## Opciones y banderas comunes

Las banderas no toman un argumento. Para usar una bandera, las agréguamos al final de un comando. Por ejemplo:

```
--8<-- 'code/setting-up-local/runnode.md'
```

- `--dev`:Especifica la cadena de desarrollo
- `--no-telemetry`:Deshabilita la conexión a la telemetría del servidor de Substrate. Para las cadenas globales, la telemetría está activada de forma predeterminada. La telemetría no está disponible si está ejecutando un nodo de desarrollo (`--dev`) .
- `--tmp`: 
Ejecuta un nodo temporal en el que toda la configuración se eliminará al final del proceso.
- `--rpc-external`: Escucha todas las interfaces RPC
- `--ws-external`: Escucha todas las interfaces de Websocket

Las opciones aceptan un argumento al lado derecho de la opción. Por ejemplo:
```
--8<-- 'code/setting-up-local/runnodewithsealinginterval.md'
```

- `-l <log pattern>` or `--log <log pattern>`: Establece un filtro de registro personalizado. La sintaxis del patrón de registro es `<target>=<level>`. Por ejemplo, para imprimir todos los registros de RPC, el comando se vería así: `-l rpc=trace`.
- `--sealing <interval>`: Cuando los bloques deben sellarse en el servicio de desarrollo. Argumentos aceptados para el intervalo: `instant`, `manual`, o un número que representa el intervalo del temporizador en milisegundos. El valor predeterminado es `instant`.
- `--rpc-port <port>`: Establece el puerto TCP del servidor HTTP RPC. Acepta un puerto como argumento.
- `--ws-port <port>`: Establece el puerto TCP del servidor WebSockets RPC. Acepta un puerto como argumento.

Para obtener una lista completa de banderas y opciones, active su nodo de desarrollo Moonbeam con `--help` agregado al final del comando.

## Cuentas de desarrollo prefinanciadas

Su nodo de desarrollo Moonbeam viene con diez cuentas prefinanciadas para el desarrollo. Las direcciones se derivan del mnemónico de desarrollo canónico de Substrate:

```
bottom drive obey lake curtain smoke basket hold race lonely fit walk
```

Ver la sección [Usando MetaMask](/getting-started/local-node/using-metamask/) s para comenzar a interactuar con sus cuentas.

--8<-- 'text/setting-up-local/dev-accounts.md'
