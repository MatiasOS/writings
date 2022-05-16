# Órdenes condicionales con Brink SDK

Esta guía cubre como firmar mensajes con Brink para que algún ejecutor pueda realizar la transacción onChain. se espera que el lector tenga conocimientos previos de programación en javascript, blockchain y entienda conceptos básicos sobre Finanzas descentralizadas (DEFI).

## Configuración
El primer paso para esta aventura es configurar un nuevo proyecto de node. Utilizando el comando [yarn init](https://classic.yarnpkg.com/lang/en/docs/cli/init/) o [npm init](https://docs.npmjs.com/cli/v6/commands/npm-init) se crea el nuevo directorio que contiene **package.json**.

Una vez creado el proyecto, se accede  al directorio y se instalan las dependencias necesarias (*dotenv*, *ethers* y *@brinkninja/sdk*). Como ejemplo se utiliza yarn

```bash
$ yarn add dotenv ethers @brinkninja/sdk
```

### dotenv
Dotenv es un módulo que carga variables de entorno de un archivo llamado **.env**. 

La idea general de dotenv, es extraer las variables de configuración del código. Mejorando aśí la usabilidad y evitando hacer públicos datos sensibles, como una clave privada. Para esto, es importante no agregar al sistema de control de version (Posiblemente git).

Para este script, vamos a crear un archivo con una entrada, WALLET_MNEMONIC la cual vamos a cargar con una frase de recuperación. En caso de no tener una, para realizar pruebas, se puede genera [acá](https://iancoleman.io/bip39/).

```bash
 # .env file

WALLET_MNEMONIC="BIP39 Mnemonic"
```

### ethers
*Ethers* es una biblioteca de propósito general para interactuar con la blockchain Ethereum y su ecosistema.

Un *Provider* en el entorno de *ethers* es una abstracción de solo lectura para acceder a los datos en blockchain. 

Un *Signer* en ethers es una abstracción de una cuenta de Ethereum. A diferencia de un *Provider*, esta se utiliza para firmar mensajes y enviar transacciones.

Las operaciones en Ethereum se encuentran por fuera del rango de javascript. Para evitar problemas, *BigNumber* es un objeto que permite operaciones matemáticas con número de cualquier magnitud.

*Ethers*, en general utiliza como parámetros y tipos de retorno *BigNumber*. 

### Brink SDK
Esta biblioteca se utiliza para interactuar con las cuentas de Brink, como el dueño de la misma, o como un ejecutor de mensajes. 

En este post, solo veremos los métodos para firmar transacciones.

## Manos a la obra
```javascript
require('dotenv').config()
const ethers = require('ethers')
const brinkSDK = require('@brinkninja/sdk')

const BN = ethers.BigNumber.from

const { WALLET_MNEMONIC } = process.env

const run = async () => {
  const signer = await new ethers.Wallet.fromMnemonic(WALLET_MNEMONIC)
  const brinktSigner = brinkSDK.accountSigner(signer, 'rinkeby')

  const signedEthToToken = await brinktSigner.signEthToTokenSwap(
    BN(0), // bitmapIndex
    BN(1), // bit
    '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenAddress
    BN('100000000000000000000'), // ethAmount
    BN('2'), // tokenAmount
    undefined // expiryBlock
  )
  console.log('')
  console.log('signedEthToToken')
  console.log(JSON.stringify(signedEthToToken))

  const signedTokenToEth = await brinktSigner.signTokenToEthSwap(
    BN(0), // bitmapIndex
    BN(2), // bit
    '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenAddress
    BN('100000000000000000'), // tokenAmount
    BN('2'), // ethAmount
  )
  console.log('')
  console.log('signedTokenToEth')
  console.log(JSON.stringify(signedTokenToEth))

  const signedTokenToToken = await brinktSigner.signTokenToTokenSwap(
    BN(0), // bitmapIndex
    BN(3), // bit
    '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenInAddress
    '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenOutAddress
    BN('100000000000000000'), // TokenInAmount
    BN('2'), // TokenOutAmount
    undefined // expiryBlock
  )

  console.log('')
  console.log('signedTokenToToken')
  console.log(JSON.stringify(signedTokenToToken))

}

run()
```

Brink está construyendo una infraestructura crítica para órdenes condicionales en Ethereum. Si te interesa saber más, podés seguirnos en [Twitter](https://twitter.com/BrinkTrade) o [Discord](https://t.co/tyu5acJHz6)












