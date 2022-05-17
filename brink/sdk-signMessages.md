# Órdenes condicionales con Brink SDK

Esta guía cubre cómo firmar mensajes con **Brink** para que algún ejecutor pueda realizar la transacción *onChain*. Se espera que el lector tenga conocimientos básicos de programación en javascript, blockchain y entienda conceptos sobre Finanzas Descentralizadas (DEFI).

## Órdenes condicionales
Una orden condicional, es un tipo especial de intercambio que sólo se puede ejecutar cuando se cumple una condición. Por ejemplo, podemos pensar un orden de 1 ETH por 2100 DAI (Al día de escribir este artículo, 1 ETH tiene un valor aproximado de 2030 DAI). Cuándo el valor del mercado llegue a los valores apropiados, un ejecutor puede tomar la orden y realizar el intercambio.

## Configuración
El primer paso para esta aventura es configurar un nuevo proyecto de node. Utilizando el comando [yarn init](https://classic.yarnpkg.com/lang/en/docs/cli/init/) o [npm init](https://docs.npmjs.com/cli/v6/commands/npm-init) se crea el nuevo directorio que contiene **package.json**. Durante todo el artículo se utiliza yarn

Una vez creado el proyecto, se accede  al directorio y se instalan las dependencias necesarias (*dotenv*, *ethers* y *@brinkninja/sdk*).

```bash
$ yarn add dotenv ethers @brinkninja/sdk
```

### dotenv
Dotenv es un módulo que carga variables de entorno de un archivo llamado **.env**. 

La idea general de dotenv, es extraer las variables de configuración del código mejorando la usabilidad y evitando hacer públicos datos sensibles, como una clave privada. Para esto, es importante no agregar al sistema de control de versiones (Posiblemente git).

Para este script, vamos a crear un archivo con una entrada, *WALLET_MNEMONIC*, la cual vamos a cargar con una frase de recuperación. En caso de no tener una, para realizar pruebas, se puede genera [acá](https://iancoleman.io/bip39/).

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
El primer paso es crear una instancia de accountSigner. Para esto necesitamos una signer de ethers y la red sobre la que vamos a ejecutar ordenes. Para estas pruebas vamos a usar la testnet de [rinkeby](https://www.rinkeby.io/).

```javascript
const signer = await new ethers.Wallet.fromMnemonic(WALLET_MNEMONIC)
const brinkAccountSigner = brinkSDK.accountSigner(signer, 'rinkeby')
```

Con la nueva instancia de account signer, podemos crear un primer mensaje
```javascript
const signedEthToToken = await brinkAccountSigner.signEthToTokenSwap(
  BN(0), // bitmapIndex
  BN(1), // bit
  '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenAddress
  BN('100000000000000000000'), // ethAmount
  BN('2'), // tokenAmount
  undefined // expiryBlock
)
```
**bitmap** y **bitmapIndex** se utilizan para no ejecutar la misma transacción 2 veces. 

**tokenAddress** es la dirección del smart contract del token que queremos intercambiar.

**ethAmount** es la cantidad de Ether que va a salir de nuestra cuenta de Brink

**tokenAmount** es el mínimo de tokens que vamos a recibir.

**expiryBlock** es el bloque hasta el cual es válida la transacción. Lo dejamos como *undefined* para indicar que siempre es válida.

Esta función devuelve un JSON con formato
```json
{
  "message": "0x7e954b39fac51f053c48ec40812c43bd1ccb8e43989b4295f476a8876d9f1011",
  "EIP712TypedData": {
    "types": {
      "MetaDelegateCall": [
        {
          "name": "to",
          "type": "address"
        },
        {
          "name": "data",
          "type": "bytes",
          "calldata": true
        }
      ]
    },
    "domain": {
      "name": "BrinkAccount",
      "version": "1",
      "chainId": 4,
      "verifyingContract": "0xF787295481f3005A065D742E092a001a797396F4"
    },
    "value": {
      "to": "0x53D468E719694f3e542Dda96a237Af08eb394f2C",
      "data": "0xdc0ed0fe000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000004d224452801aced8b2f0aebe155379bb5d5943810000000000000000000000000000000000000000000000056bc75e2d631000000000000000000000000000000000000000000000000000000000000000000002ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"
    }
  },
  "signature": "0x9271fb25da1891e7daf114b87a65f6e0eefabb12d1802a19566d767514fdf1d365adfd37cef24cd5ffa36dd8dcb64b93cde8a93a6afaf0bd7a2378fcbace15cb1c",
  "signer": "0x3bc8dE4CF6c075Fb8e24A954EC1D1B12bDcbF336",
  "accountAddress": "0xF787295481f3005A065D742E092a001a797396F4",
  "functionName": "metaDelegateCall",
  "signedParams": [
    {
      "name": "to",
      "type": "address",
      "value": "0x53D468E719694f3e542Dda96a237Af08eb394f2C"
    },
    {
      "name": "data",
      "type": "bytes",
      "value": "0xdc0ed0fe000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000004d224452801aced8b2f0aebe155379bb5d5943810000000000000000000000000000000000000000000000056bc75e2d631000000000000000000000000000000000000000000000000000000000000000000002ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "callData": {
        "functionName": "ethToToken",
        "params": [
          {
            "name": "bitmapIndex",
            "type": "uint256",
            "value": "0"
          },
          {
            "name": "bit",
            "type": "uint256",
            "value": "1"
          },
          {
            "name": "token",
            "type": "address",
            "value": "0x4d224452801aced8b2f0aebe155379bb5d594381"
          },
          {
            "name": "ethAmount",
            "type": "uint256",
            "value": "100000000000000000000"
          },
          {
            "name": "tokenAmount",
            "type": "uint256",
            "value": "2"
          },
          {
            "name": "expiryBlock",
            "type": "uint256",
            "value": "115792089237316195423570985008687907853269984665640564039457584007913129639935"
          },
          {
            "name": "to",
            "type": "address"
          },
          {
            "name": "data",
            "type": "bytes"
          }
        ]
      }
    }
  ]
}
```



```javascript
const signedTokenToEth = await brinkAccountSigner.signTokenToEthSwap(
  BN(0), // bitmapIndex
  BN(2), // bit
  '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenAddress
  BN('100000000000000000'), // tokenAmount
  BN('2'), // ethAmount
  undefined // expiryBlock
)
```
Notar que para el método **signTokenToEthSwap**, **ethAmount** y **tokenAmount** aparecen en distinto orden.

El último mensaje a firmar es un intercambio de tokens por otros
```javascript
const signedTokenToToken = await brinkAccountSigner.signTokenToTokenSwap(
  BN(0), // bitmapIndex
  BN(3), // bit
  '0x4d224452801aced8b2f0aebe155379bb5d594381', // tokenInAddress
  '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // tokenOutAddress
  BN('100000000000000000'), // TokenInAmount
  BN('2'), // TokenOutAmount
  undefined // expiryBlock
)
```
En este mensaje, *tokenInAddress* es el token a enviar y *tokenOutAddress* el token a recibir. 

Con esto último, el script completo es
```javascript
require('dotenv').config()
const ethers = require('ethers')
const brinkSDK = require('@brinkninja/sdk')

const BN = ethers.BigNumber.from

const { WALLET_MNEMONIC } = process.env

const run = async () => {
  const signer = await new ethers.Wallet.fromMnemonic(WALLET_MNEMONIC)
  const brinktSigner = brinkSDK.accountSigner(signer, 'rinkeby')

  const signedEthToToken = await brinkSigner.signEthToTokenSwap(
    BN(0), // bitmapIndex
    BN(1), // bit
    '0x1f9840a85d5af5bf1d1762f925bdaddc4201f984', // tokenAddress
    BN('100000000000000000000'), // ethAmount
    BN('2'), // tokenAmount
    undefined // expiryBlock
  )
  console.log('')
  console.log('signedEthToToken')
  console.log(JSON.stringify(signedEthToToken))

  const signedTokenToEth = await brinkSigner.signTokenToEthSwap(
    BN(0), // bitmapIndex
    BN(2), // bit
    '0x1f9840a85d5af5bf1d1762f925bdaddc4201f984', // tokenAddress
    BN('100000000000000000'), // tokenAmount
    BN('2'), // ethAmount
  )
  console.log('')
  console.log('signedTokenToEth')
  console.log(JSON.stringify(signedTokenToEth))

  const signedTokenToToken = await brinkSigner.signTokenToTokenSwap(
    BN(0), // bitmapIndex
    BN(3), // bit
    '0x1f9840a85d5af5bf1d1762f925bdaddc4201f984', // tokenInAddress
    '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // tokenOutAddress
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












