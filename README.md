# Getting started

The DeversiFi API allows trading of cryptocurrency tokens from any Ethereum wallet or smart-contract.

**Note: DeversiFi currently uses version 2.1 of the 0x protocol for settlement on the Ethereum blockchain.**

The available endpoints allow access to submit, cancel and query placed orders onto the DeversiFi order book, whilst keeping full custody of funds and authenticating only using an Ethereum account. By using this API anyone is able to create and integrate their own interfaces, or run trading algorithms whilst keeping secure control of their funds in a personal Ethereum wallet.

If you are new to interacting with the Ethereum blockchain and DeversiFi API usage, be sure to go through the [Beginner API DeversiFi guide](#e4a19166-5710-406a-a856-485a217124e0).

This documentation set is actively maintained and updated. If you would like to suggest any changes or find there is something missing, please reach out to us via email -  feedback@deversifi.com - or leave a suggestion as a comment.

The base URL for requests is `https://api.deversifi.com`

- Trading base url: `https://api.deversifi.com/v1/trading/`
- Public volume data base url: `https://api.deversifi.com/v1/pub/`
- Price data base url (bitfinex proxy): `https://api.deversifi.com/bfx/v2/`
- Price data websocket proxy base url (bitfinex proxy): `https://api.deversifi.com/bfx/ws/2/`

---

# Libraries and Examples

A node.js client is available to interface with this API at https://github.com/ethfinex/efx-api-node

`npm i efx-api-node`

---

# Public Endpoints

To request public data, for example order books, candles, or trade history, the format is identical to the Bitfinex Rest and Websocket APIs.

For an example of requesting the full Orderbook, please see [Get full Orderbook](#f3471e15-5e30-428c-83b3-da2dc63184e6).

For price data endpoints, the documentation for these is also available at https://docs.bitfinex.com/docs with the only change required being to switch `api.bitfinex.com` with `api.deversifi.com/bfx/` as the base url.

Websocket is recommended to stream orders, and a [javascript client library](https://github.com/bitfinexcom/bitfinex-api-node) makes this easy (please make sure to switch your base-url to the deversifi endpoints).

---

# Authentication

No API key pair is required to be issued for the authenticated endpoints.

Instead authentication is done using an Ethereum private key to sign messages.
This signing is done using the Ethereum web3 library and a code example is shown below.

```javascript
web3.eth.sign(toSign, address, (err, res) => {
      if (err) { reject(err) }
      resolve(res)
    })
```

The same basic method of authentication is used to sign different payloads in order to view the user's open orders, cancel orders, and place new ones.

Note that if using Metamask this should be `web3.eth.personal.sign` instead of `web3.eth.sign`. Examples are available in the node client: https://github.com/ethfinex/efx-api-node/blob/master/src/api/sign/sign.js

---

# Smart Contracts

### Mainnet

The following are the key contracts deployed on the Ethereum mainnet, as well as examples of some of the token Wrappers:

| Contract        | Address          |
| ------------- |:-------------:|
| Exchange0xV2.1 Contract    | [0x080bf510FCbF18b91105470639e9561022937712](https://etherscan.io/address/0x080bf510FCbF18b91105470639e9561022937712) |
| ETHWrapper            | [0x50cB61AfA3F023d17276DCFb35AbF85c710d1cfF](https://etherscan.io/address/0x50cB61AfA3F023d17276DCFb35AbF85c710d1cfF)     |
| USDTWrapper    | [0x33d019eb137b853f0cdf555a5d5bd2749135ac31](https://etherscan.io/address/0x33d019eb137b853f0cdf555a5d5bd2749135ac31)     |
| MKRWrapper     | [0x91cf769a44e9d09b8e249886d0de4dbe0aa998f9](https://etherscan.io/address/0x91cf769a44e9d09b8e249886d0de4dbe0aa998f9) |

Full configuration information can be fetched using the [getConfig](#3d74c943-f274-4e92-a051-a9682a262de1) api request.


### Kovan

The following contracts are deployed on the Kovan test network:

| Contract        | Address          |
| ------------- |:-------------:|
| Wrapper Registry         | [0x750DeaE872619eb2Cf6c65FD07FCbc60E8D98b73](https://kovan.etherscan.io/address/0x750DeaE872619eb2Cf6c65FD07FCbc60E8D98b73) |
| Exchange Contract vEFX     | [0x77525f7db94116677dbd02c93829aba0c86d9768](https://kovan.etherscan.io/address/0x77525f7db94116677dbd02c93829aba0c86d9768) |
| Exchange Contact 0xv2    | [0xf1ec01d6236d3cd881a0bf0130ea25fe4234003e](https://kovan.etherscan.io/address/0xf1ec01d6236d3cd881a0bf0130ea25fe4234003e)     |

This information and remaining token wrappers can be found by calling [getConfig](#3d74c943-f274-4e92-a051-a9682a262de1) api request using the `https://test.ethfinex.com/trustless/v1/` base url. Wrapper Registry also enables a lookup of different tokens for their wrappers.


# Time Lock Wrapper Contracts

The DeversiFi non-custodial exchange API uses wrapper token contracts to ensure that traders will always be able to settle all of their open orders if required. These 'wrapper tokens' lock the tokens access for a specified duration, and during that time the tokens are reserved and can only be transferred as a result of a successfully executed order. Early unlock can be requested from DeversiFi using the `/releaseTokens` endpoint, provided the user does not have any open orders.

While the tokens are locked, they can only be used by DeversiFi to fill the owner's submitted trades. Below an example code snippet of locking/unlocking is shown.

The process for locking ETH is slightly different than that for tokens.

---

# Locking ETH

1. Call `deposit(amount, duration)` on the ETH Lock Contract. Amount is in `wei` and duration is in `hours`. This should also have the value of ETH you wish to deposit sent with the contract call.

---

# Locking ERC20 Tokens

1. Call `approve(lockTokenContract, MAX_UINT)` . MAX_UINT = `2 ** 256 - 1`. This will allow the lockTokenContract to transfer on the user's behalf. This step only needs to be carried out the first time interacting with each new ERC20 token.

2. Call `deposit(amount, duration)`. This will transfer the specific token into the lockTokenContract.

These methods can be handled by the api-node library https://github.com/ethfinex/efx-api-node#locking-tokens and examples can be found in that library.

```javascript
const EFX = require('efx-api-node')
const web3 = new EFX.Web3(/*your web3 provider*/)
const efx = await EFX( web3 )

const token = 'ZRX'
const amount = 15 // Number of tokens to lock
const forTime = 48 // Time after which unlocking does not require permission

const allowance = await efx.contract.isApproved(token)
if (allowance === 0 || allowance < amount) {
    await efx.contract.approve(token)
}

const response = await efx.contract.lock(token, amount, forTime)

```

The above function can be used to lock tokens. The efx-api-node library takes care of mapping details from a particular token 3-letter code to its token address, lock wrapper address, and decimals.

The process for unlocking tokens requires either:
1. The time lock has expired already,
2. You have no open orders and request an early release signature from DeversiFi.

```javascript
const token = 'ZRX'
const amount = 15 // Number of tokens to lock

const response = await efx.contract.unlock(token, amount)

```

**Please note, when unlocking any token the whole amount of the token is unlocked, and so cannot be unlocked in part.**

---

# USD Tether Markets



The XXX/USDT markets on DeversiFi build on the liquidity of XXX/USD markets on the centralised Bitfinex exchange. However since there is often not a direct 1:1 rate between USD and USDT, a shift must be applied to the order books.

The configuration for DeversiFi returns a `settleSpread` parameter:

```json
      "USD":{
          "decimals":6,
          "wrapperAddress":"0x83e42e6d1ac009285376340ef64bac1c7d106c89",
          "tokenAddress":"0x0736d0c130b2ead47476cc262dbed90d7c4eeabd",
          "minOrderSize":10,
          "settleSpread": 0.02
      }
```

This `settleSpread` is indicative of the current USDT/USD exchange rate. When orders are placed on USDT markets, the settlement price in the signed order must be shifted by the settleSpread parameter before the order is accepted.

For example, if placing a buy order on the ETH/USD(T) market at a price of 100 USD relative to the centralised exchange the order will be settled on DeversiFi at a price of 102 USDT. Equally a sell order at 100 USD would receive 102 USDT when settled on DeversiFi.

```javascript
efx.submitOrder(symbol, amount, price)
// => settlementPrice = price * (1 + settleSpread)
```

The `settleSpread` parameter is set dynamically as a 30 minute rolling mean of the USDT/USD market exchange rate. When placing orders using submitOrder or generating them with createOrder the shift is applied for you.

---

# Error Codes

Error codes are returned to indicate when submitted orders, or other requests are invalid.

The node.js library for the API contains a list of errors, along with [human readable explanations](https://github.com/ethfinex/efx-api-node/blob/master/src/lib/error/reasons.js).

Many of these codes relate to an incorrect match between the fields in the submitted [0x Order format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) and those specified in the API order. Orders once submitted via API are checked to ensure all the parameters meet the requirements before they can be added to the order books.

These checks include amongst others:
1. Exchange address is specified as DeversiFi's address. This is returned by the [/r/get/conf](#3d74c943-f274-4e92-a051-a9682a262de1) .
2. Maker and taker token addresses and amounts match the pair, amount and price of the submitted order. Note that addresses here are the wrapper addresses, not the base token addresses.
3. The address placing the order has sufficient locked funds, and that the remaining expiry time of the lock is greater than the expiry time of the order.
4. The signature of the hashed order is valid.

For more information on preparing this order format please see [0x Order format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format).

---

# Troubleshooting

`ERR_MAKERTOKEN_ADDRESS_INVALID`

If you receive this error the 'maker' token address provided in the 0x order format did not match the pair specified by the API call.

The DeversiFi setup requires that you have temporarily 'time-locked' the tokens you will trade into a wrapper contract. You should verify that in the field `makerToken` of the [0x order JSON](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) you have specified this wrapper contract address rather than the original token address. Make sure you are using an up to date token wrapper mapping from DeversiFi for mainnet.

The `makerToken` is always the token *which you will send* when the order is successfully executed.

---

`ERR_TAKERTOKEN_ADDRESS_INVALID`

If you receive this error the 'taker' token address provided in the 0x order format did not match the pair specified by the API call.

Make sure you are using the correct token address in the field  `takerToken` of the [0x order format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) for the decentralized order. This token depends on the pair of the trade and whether you are placing a BUY or SELL. The `takerToken` is always the token *which you will receive* when the order is successfully executed.

This error may also be given if placing orders on USDT markets, without applying the `settleSpread` conversion. [See here for more](#usd-tether-markets).

---

`ERR_MAKERTOKEN_AMOUNT_INVALID` or  `ERR_TAKERTOKEN_AMOUNT_INVALID`

The specified 0x order JSON maker amount did not match with the amount and price specified in the API call.

Make sure that `makerTokenAmount` and `takerTokenAmount` in the [0x order format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) match the amounts in the normal API order. Note that the 0x order takes the amounts in base units (wei) while the normal order takes them in tokens. Make sure you are using the correct number of decimals for this conversion.

This error may also be given if placing orders on USDT markets, without applying the `settleSpread` conversion. [See here for more](#usd-tether-markets).

---

`ERR_0X_SIGNATURE_INVALID`

The 0x signed order hash was invalid.

Check the documentation to make sure you are correctly signing the 0x order, and make sure you are placing it in the submitted order object before posting a new order.

---

`ERR_0X_FEE_RECIPIENT_INVALID`

The fee recipient address, `feeRecipient` in the [0x order format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format), was not specified as DeversiFi. This setup requires that each order placed specifies DeversiFi as the recipient of any fees.

If you receive this error, make sure you are using the address specified in the documentation as the fee recipient for the DeversiFi order.

---

`ERR_0X_EXCHANGE_INVALID`

The exchange contract address for the trade was not specified correctly. The field `exchangeContractAddress` in the [0x order JSON](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) must specify DeversiFi's contract address.

If you get this error, make sure you are using the exchange contract address specified by DeversiFi's documentation.

---

`ERR_0X_TAKER_INVALID`

The 0x order format allows taker address to be left blank and therefore for anyone to fill an order. In order for the order to be accepted onto the DeversiFi order book it must specify the `taker` field to be DeversiFi.

If you receive this error, make sure you are using the address specified by DeversiFi as the taker for the order.

---

`ERR_0X_EXPIRED`

The order expiration date was too soon, or already passed. Make sure the expiration date, which is specified by `expirationTimestampInSec` is in the future and at least 1 hour away.

---

`ERR_0X_BELOW_MIN_SIZE`

The order was below the minimum threshold. Because of the fees involved with settling trades over Ethereum, a minimum size is required.

Make sure the order is for an amount above the DeversiFi threshold, which is (TBD)

---

`ERR_0X_LOCK_TIME_INSUFFICIENT`

The order expires after the time lock on the tokens is released. Orders will only be accepted onto the DeversiFi order book if the tokens are locked into the wrapper for longer than the expiry time of the order. This ensures that the order book is always a true representation of the available liquidity.

Make sure that when generating orders you use an expiration which is sooner than the lock duration remaining on your tokens. Alternatively re-lock your tokens to increase the duration. You should always be able to release your tokens early if no longer used in active orders by requesting an unlock signature from DeversiFi.

---

`ERR_0X_MAKER_INSUFFICIENT_BALANCE`

Not enough balance of time locked tokens to place the order.

Make sure that you have locked a sufficient amount of tokens for the order into the wrapper contracts.

---

`ERR_UNLOCK_TOO_LONG`

The requested unlock signature is valid for an excessive time period.

As you will not be able to place orders during the unlocked period, there is a maximum limit to how long it is allowed to be. Please try a shorter unlock duration.


## How to Build

The generated SDK relies on [Node Package Manager](https://www.npmjs.com/) (NPM) being available to resolve dependencies. If you don't already have NPM installed, please go ahead and follow instructions to install NPM from [here](https://nodejs.org/en/download/).
The SDK also requires Node to be installed. If Node isn't already installed, please install it from [here](https://nodejs.org/en/download/)
> NPM is installed by default when Node is installed

To check if node and npm have been successfully installed, write the following commands in command prompt:

* `node --version`
* `npm -version`

![Version Check](https://apidocs.io/illustration/nodejs?step=versionCheck&workspaceFolder=Introduction%20-%20DeversiFi-Node)

Now use npm to resolve all dependencies by running the following command in the root directory (of the SDK folder):

```bash
npm install
```

![Resolve Dependencies](https://apidocs.io/illustration/nodejs?step=resolveDependency1&workspaceFolder=Introduction%20-%20DeversiFi-Node)

![Resolve Dependencies](https://apidocs.io/illustration/nodejs?step=resolveDependency2)

This will install all dependencies in the `node_modules` folder.

Once dependencies are resolved, you will need to move the folder `IntroductionDeversiFiLib ` in to your `node_modules` folder.

## How to Use

The following section explains how to use the library in a new project.

### 1. Open Project Folder
Open an IDE/Text Editor for JavaScript like Sublime Text. The basic workflow presented here is also applicable if you prefer using a different editor or IDE.

Click on `File` and select `Open Folder`.

![Open Folder](https://apidocs.io/illustration/nodejs?step=openFolder)

Select the folder of your SDK and click on `Select Folder` to open it up in Sublime Text. The folder will become visible in the bar on the left.

![Open Project](https://apidocs.io/illustration/nodejs?step=openProject&workspaceFolder=Introduction%20-%20DeversiFi-Node)

### 2. Creating a Test File

Now right click on the folder name and select the `New File` option to create a new test file. Save it as `index.js` Now import the generated NodeJS library using the following lines of code:

```js
var lib = require('lib');
```

Save changes.

![Create new file](https://apidocs.io/illustration/nodejs?step=createNewFile&workspaceFolder=Introduction%20-%20DeversiFi-Node)

![Save new file](https://apidocs.io/illustration/nodejs?step=saveNewFile&workspaceFolder=Introduction%20-%20DeversiFi-Node)

### 3. Running The Test File

To run the `index.js` file, open up the command prompt and navigate to the Path where the SDK folder resides. Type the following command to run the file:

```
node index.js
```

![Run file](https://apidocs.io/illustration/nodejs?step=runProject&workspaceFolder=Introduction%20-%20DeversiFi-Node)


## How to Test

These tests use Mocha framework for testing, coupled with Chai for assertions. These dependencies need to be installed for tests to run.
Tests can be run in a number of ways:

### Method 1 (Run all tests)

1. Navigate to the root directory of the SDK folder from command prompt.
2. Type `mocha --recursive` to run all the tests.

### Method 2 (Run all tests)

1. Navigate to the `../test/Controllers/` directory from command prompt.
2. Type `mocha *` to run all the tests.

### Method 3 (Run specific controller's tests)

1. Navigate to the `../test/Controllers/` directory from command prompt.
2. Type `mocha  Introduction - DeversiFiController`  to run all the tests in that controller file.

> To increase mocha's default timeout, you can change the `TEST_TIMEOUT` parameter's value in `TestBootstrap.js`.

![Run Tests](https://apidocs.io/illustration/nodejs?step=runTests&controllerName=Introduction%20-%20DeversiFiController)

## Initialization

### 

API client can be initialized as following:

```JavaScript
const lib = require('lib');


```



# Class Reference

## <a name="list_of_controllers"></a>List of Controllers

* [ReadEndpointsController](#read_endpoints_controller)
* [WriteEndpointsController](#write_endpoints_controller)
* [PriceDataEndpointsController](#price_data_endpoints_controller)
* [PublicDataEndpointsController](#public_data_endpoints_controller)

## <a name="read_endpoints_controller"></a>![Class: ](https://apidocs.io/img/class.png ".ReadEndpointsController") ReadEndpointsController

### Get singleton instance

The singleton instance of the ``` ReadEndpointsController ``` class can be accessed from the API Client.

```javascript
var controller = lib.ReadEndpointsController;
```

### <a name="create_get_config"></a>![Method: ](https://apidocs.io/img/method.png ".ReadEndpointsController.createGetConfig") createGetConfig

> - Several Ethereum addresses are required when constructing orders to sign and submit.
> - Tradable pairs can also be retrieved, with corresponding minimum order sizes and decimals for each token<a name="getConfig">.</a>


```javascript
function createGetConfig(accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'application/json';
    var contentType = 'application/json';

    controller.createGetConfig(accept, contentType, function(error, response, context) {

    
    });
```



### <a name="create_get_open_orders"></a>![Method: ](https://apidocs.io/img/method.png ".ReadEndpointsController.createGetOpenOrders") createGetOpenOrders

> TODO: Add a method description


```javascript
function createGetOpenOrders(symbol, accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| symbol |  ``` Required ```  | TODO: Add a parameter description |
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var symbol = '';
    var accept = 'application/json';
    var contentType = 'application/json';

    controller.createGetOpenOrders(symbol, accept, contentType, function(error, response, context) {

    
    });
```



### <a name="create_get_order_history"></a>![Method: ](https://apidocs.io/img/method.png ".ReadEndpointsController.createGetOrderHistory") createGetOrderHistory

> ```javascript
>   const token = ((Date.now() / 1000) + 60 * 60 * 24) + ''
>   const signature = await efx.sign(token.toString(16))
> ```
> 
> 


```javascript
function createGetOrderHistory(accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'Accept';
    var contentType = 'Content-Type';

    controller.createGetOrderHistory(accept, contentType, function(error, response, context) {

    
    });
```



[Back to List of Controllers](#list_of_controllers)

## <a name="write_endpoints_controller"></a>![Class: ](https://apidocs.io/img/class.png ".WriteEndpointsController") WriteEndpointsController

### Get singleton instance

The singleton instance of the ``` WriteEndpointsController ``` class can be accessed from the API Client.

```javascript
var controller = lib.WriteEndpointsController;
```

### <a name="create_submit_order"></a>![Method: ](https://apidocs.io/img/method.png ".WriteEndpointsController.createSubmitOrder") createSubmitOrder

> The submit order endpoint requires an order prepared in a certain message format. This message format matches that defined by [0xproject](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format).
> 
> ### Order Message Format
> 
> Below is an example 0x v2 order. This trading standard relies on the use of smart contracts to allow exchanges to use funds securely and strictly for filling orders from traders. 
> 
> Combined with the wrappers mentioned in the Token Wrapper Lock section, this allows Ethfinex to integrate the decentralized orders directly into the main orderbook while still ensuring the same promises of decentralized exchanges.
> 
> The wrapper token contracts main use is to guarantee that orders submitted to Ethfinex will be fillable, which allows seamless integration and trading against centralized users. The contracts however only allow Ethfinex to use the locked funds for the purpose of filling orders, which guarantees their safety for the trader.
> 
> **Warning:** Trustless orders will always be settled at the exact price you specify, and can never be adjusted by Ethfinex, even if it is at a worse price than the market.
> 
> **For example, when placing a sell order, if the price entered is below the highest bid available on the order book, the order will be executed instantly at market. However, the amount you receive will reflect only the price that you entered, and not the market price at the time of execution.**
> 
> Note: for orders on USDT markets there is an additional conversion which must be made between USD and USDT markets. [See here for more](#usd-tether-markets).
> 
> ---
> ```javascript
> let order = {
> makerAddress: userAddress, // Your ethereum address
> takerAddress: '0x0000000000000000000000000000000000000000',
> 
> feeRecipientAddress: ethfinexAddress, // ethfinex provided address
> 
> senderAddress: ethfinexAddress,
> 
> makerAssetAmount: web3.utils.toBN(sellAmount), // in the smallest unit of the token, e.g in wei for ether
> 
> takerAssetAmount: web3.utils.toBN(buyAmount), // as above
> 
> makerFee: web3.utils.toBN('0'), // constant
> 
> takerFee: web3.utils.toBN('0'), // constant
> 
> expirationTimeSeconds: web3.utils.toBN(Math.round((new Date()).getTime() / 1000) + ((defaultExpiry || 60) * 60)), // part after the plus can be replaced, first part is constant
>     
> salt: ZeroEx.generatePseudoRandomSalt(), // as is
> 
> makerAssetData: assetDataUtils.encodeERC20AssetData(sellCurrency.wrapperAddress.toLowerCase()), // token type and wrapper token address
>   
> takerAssetData: assetDataUtils.encodeERC20AssetData(buyCurrency.wrapperAddress.toLowerCase()),  // token type and wrapper token address
>   }
> ```
> 
> The `taker` and `feeRecipient` should be Ethfinex's address, the `takerFee` and `makerFee` should be 0 (Ethfinex takes the fee out of the traded amount), the token addresses involved are the wrappers, and the `exchangeContractAddress` is the deployed Exchange contract to which matched orders will be submitted to execute. This order then must be signed as demonstrated above and have this added to the object.
> 
> ### Making orders from smart contracts
> 
> Orders can be placed on behalf of a contract by its owner or manager, and must be signed by them.
> An additional field in the `meta` order object must be specified to enable this.
> 
> `meta.sigType = 'contract'`
> 
> When set this will ensure orders are validated to see if the signer has permission to sign trades on behalf of the smart contract.
> 
> `gid`, string, Group Order ID
> 
> `cid`, string, Client order ID
> 
> `symbol`, string, Trading Symbol Pair
> 
> `amount`, string, Positive for buying, negative for selling
> 
> `price`, string, Price
> 
> `meta`, object, The signed 0x order object used by the decentralized trading protocol
> 
> `protocol`, string, The protocol used for decentralized trading (0x format currently supported)


```javascript
function createSubmitOrder(accept, contentType, body, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |
| body |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'application/json';
    var contentType = 'text/html';
    var body = '{ \n\'gid\': 1,\n\'cid\': 1575844655218,\n\'type\': EXCHANGE LIMIT,\n\'symbol\':tETHUSD,\n\'amount\': -1,\n\'price\': 201,\n\'meta\': signedOrder,\n\'protocol\': 0x\n}\n';

    controller.createSubmitOrder(accept, contentType, body, function(error, response, context) {

    
    });
```



### <a name="create_cancel_order"></a>![Method: ](https://apidocs.io/img/method.png ".WriteEndpointsController.createCancelOrder") createCancelOrder

> To cancel an order, the order ID must be signed as follows, and included in the signature field:
> ```javascript
> const sig = await efx.sign(parseInt(orderId).toString(16))
> const sigConcat = ethUtils.toRpcSig(sig.v, ethUtils.toBuffer(sig.r), ethUtils.toBuffer(sig.s))
> ```
> N.B. the signature is a hex string here, and if using ecsign function from a library which returns v, r, s it must be concatenated.
> 
> 


```javascript
function createCancelOrder(accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'application/json';
    var contentType = 'application/json';

    controller.createCancelOrder(accept, contentType, function(error, response, context) {

    
    });
```



### <a name="create_release_tokens"></a>![Method: ](https://apidocs.io/img/method.png ".WriteEndpointsController.createReleaseTokens") createReleaseTokens

> Requests an early unlock signature for the specified addresses, allowing tokens to be withdrawn before the locktime is over. The response should include A) `releaseSignature` B) `unlockUntilBlockNumber`
> These are then submitted to the token wrapper lock contract (with releaseSignature broken down into v, r, s): `withdraw(amountToWithdraw, v, r, s,  unlockUntil )`
> NB: As long as there is a pending unlock for an address, no orders will be accepted from that address and no orders will be filled.
> 
> 


```javascript
function createReleaseTokens(accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'application/json';
    var contentType = 'application/json';

    controller.createReleaseTokens(accept, contentType, function(error, response, context) {

    
    });
```



[Back to List of Controllers](#list_of_controllers)

## <a name="price_data_endpoints_controller"></a>![Class: ](https://apidocs.io/img/class.png ".PriceDataEndpointsController") PriceDataEndpointsController

### Get singleton instance

The singleton instance of the ``` PriceDataEndpointsController ``` class can be accessed from the API Client.

```javascript
var controller = lib.PriceDataEndpointsController;
```

### <a name="get_ticker_data"></a>![Method: ](https://apidocs.io/img/method.png ".PriceDataEndpointsController.getTickerData") getTickerData

> - Requests ticker data for a pair or for all pairs if no symbol is specified
> - Response is formatted as per https://docs.bitfinex.com API v2
> 


```javascript
function getTickerData(symbols, accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| symbols |  ``` Required ```  | TODO: Add a parameter description |
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var symbols = 'tMKRETH';
    var accept = 'application/json';
    var contentType = 'application/json';

    controller.getTickerData(symbols, accept, contentType, function(error, response, context) {

    
    });
```



[Back to List of Controllers](#list_of_controllers)

## <a name="public_data_endpoints_controller"></a>![Class: ](https://apidocs.io/img/class.png ".PublicDataEndpointsController") PublicDataEndpointsController

### Get singleton instance

The singleton instance of the ``` PublicDataEndpointsController ``` class can be accessed from the API Client.

```javascript
var controller = lib.PublicDataEndpointsController;
```

### <a name="get_volume"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getVolume") getVolume

> - Requests the volume by day, with the above example returning details regarding the volume for 01/08/2019.
> 
> - Furthermore, requesting the following would return the volume details for all tokens over the last 24 hours:  
> 
> ``` {
> https://api.deversifi.com/v1/pub/last24HoursVolume
> ```


```javascript
function getVolume(accept, contentType, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| accept |  ``` Required ```  | TODO: Add a parameter description |
| contentType |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var accept = 'application/json';
    var contentType = 'application/json';

    controller.getVolume(accept, contentType, function(error, response, context) {

    
    });
```



### <a name="get_block_info"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getBlockInfo") getBlockInfo

> - Return information for block #8232895


```javascript
function getBlockInfo(callback)
```

#### Example Usage

```javascript


    controller.getBlockInfo(function(error, response, context) {

    
    });
```



### <a name="get_txid_info"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getTXIDInfo") getTXIDInfo

> - Return information for transaction hash: 0x6a502fab01e83e41b1f0dba0448800ccee7e8a379823b938ecf6e12e5491a110


```javascript
function getTXIDInfo(callback)
```

#### Example Usage

```javascript


    controller.getTXIDInfo(function(error, response, context) {

    
    });
```



### <a name="get_event_id_info"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getEventIDInfo") getEventIDInfo

> - Return information for event id: 0x6a502fab01e83e41b1f0dba0448800ccee7e8a379823b938ecf6e12e5491a110-45


```javascript
function getEventIDInfo(callback)
```

#### Example Usage

```javascript


    controller.getEventIDInfo(function(error, response, context) {

    
    });
```



### <a name="get_maker_event_info"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getMakerEventInfo") getMakerEventInfo

> - Return all events where maker address is 0xf63246f4df508eba748df25daa8bd96816a668ba


```javascript
function getMakerEventInfo(callback)
```

#### Example Usage

```javascript


    controller.getMakerEventInfo(function(error, response, context) {

    
    });
```



### <a name="get_maker_event_time_period"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getMakerEventTimePeriod") getMakerEventTimePeriod

> - Return all events where maker address is 0xf63246f4df508eba748df25daa8bd96816a668ba
> - Providing startDate unix timestamp in millieconds ) for Tue Aug 06 2019 14:26:18 GMT+0100
> - Providing endDate unix timestamp in millieconds ) for Wed Sep 08 2019 23:00:00 GMT+0100


```javascript
function getMakerEventTimePeriod(startDate, endDate, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| startDate |  ``` Required ```  | TODO: Add a parameter description |
| endDate |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var startDate = 1565097982081;
    var endDate = 1565184391511;

    controller.getMakerEventTimePeriod(startDate, endDate, function(error, response, context) {

    
    });
```



### <a name="get_taker_event_info"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getTakerEventInfo") getTakerEventInfo

> - Return all events where taker address is 0x61b9898c9b60a159fc91ae8026563cd226b7a0c1


```javascript
function getTakerEventInfo(callback)
```

#### Example Usage

```javascript


    controller.getTakerEventInfo(function(error, response, context) {

    
    });
```



### <a name="get_taker_event_time_period"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getTakerEventTimePeriod") getTakerEventTimePeriod

> - Return all events where taker address is 0x61b9898c9b60a159fc91ae8026563cd226b7a0c1
> - Providing startDate unix timestamp in millieconds ) for Tue Aug 06 2019 14:26:18 GMT+0100
> - Providing endDate unix timestamp in millieconds ) for Wed Aug 07 2019 14:26:28 GMT+0100


```javascript
function getTakerEventTimePeriod(startDate, endDate, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| startDate |  ``` Required ```  | TODO: Add a parameter description |
| endDate |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var startDate = 1565097982081;
    var endDate = 1565184391511;

    controller.getTakerEventTimePeriod(startDate, endDate, function(error, response, context) {

    
    });
```



### <a name="get_all_time_volume_ranking"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getAllTimeVolumeRanking") getAllTimeVolumeRanking

>  - Return all time volume ranking quoted in ETH


```javascript
function getAllTimeVolumeRanking(callback)
```

#### Example Usage

```javascript


    controller.getAllTimeVolumeRanking(function(error, response, context) {

    
    });
```



### <a name="get_time_period_volume_ranking"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getTimePeriodVolumeRanking") getTimePeriodVolumeRanking

> - Return volume ranking quoted in ETH
> - Providing startDate unix timestamp in millieconds ) for Tue Aug 06 2019 14:26:18 GMT+0100
> - Providing endDate unix timestamp in millieconds ) for Wed Aug 07 2019 14:26:28 GMT+0100


```javascript
function getTimePeriodVolumeRanking(startDate, endDate, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| startDate |  ``` Required ```  | TODO: Add a parameter description |
| endDate |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var startDate = 1565097982081;
    var endDate = 1565184391511;

    controller.getTimePeriodVolumeRanking(startDate, endDate, function(error, response, context) {

    
    });
```



### <a name="get_all_tokens_volume_ranking"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getAllTokensVolumeRanking") getAllTokensVolumeRanking

> - Return all time volume ranking ( for all tokens together ) quoted in USD


```javascript
function getAllTokensVolumeRanking(callback)
```

#### Example Usage

```javascript


    controller.getAllTokensVolumeRanking(function(error, response, context) {

    
    });
```



### <a name="get_all_tokens_volume_ranking_time_period"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.getAllTokensVolumeRankingTimePeriod") getAllTokensVolumeRankingTimePeriod

> - Return volume ranking ( for all tokens together ) quoted in USD
> - Providing startDate unix timestamp in millieconds ) for Tue Aug 06 2019 14:26:18 GMT+0100
> - Providing endDate unix timestamp in millieconds ) for Wed Aug 07 2019 14:26:28 GMT+0100


```javascript
function getAllTokensVolumeRankingTimePeriod(startDate, endDate, callback)
```
#### Parameters

| Parameter | Tags | Description |
|-----------|------|-------------|
| startDate |  ``` Required ```  | TODO: Add a parameter description |
| endDate |  ``` Required ```  | TODO: Add a parameter description |



#### Example Usage

```javascript

    var startDate = 1565097982081;
    var endDate = 1565184391511;

    controller.getAllTokensVolumeRankingTimePeriod(startDate, endDate, function(error, response, context) {

    
    });
```



### <a name="get30_day_volume_specific_address"></a>![Method: ](https://apidocs.io/img/method.png ".PublicDataEndpointsController.get30DayVolumeSpecificAddress") get30DayVolumeSpecificAddress

> - Return last 30 days volume for 0xf63246f4df508eba748df25daa8bd96816a668ba
> - Cached for 60 minutes


```javascript
function get30DayVolumeSpecificAddress(callback)
```

#### Example Usage

```javascript


    controller.get30DayVolumeSpecificAddress(function(error, response, context) {

    
    });
```



[Back to List of Controllers](#list_of_controllers)



