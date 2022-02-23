# How to connect a custom ERC-20 token to a virtual account

---

Tatum Virtual Accounts can support ERC-20 tokens whether they are existing or not out-of-the-box. To support your own ERC-20 token and utilize all the off-chain features, you must create your own virtual currency inside a virtual account. When you produce your own virtual currency, you can create accounts and send virtual account and blockchain transactions.

---

<!-- theme: info -->

> A virtual currency is an entity in the virtual account distributed database. It is created with an initial supply of coins. This supply can be extended or reduced at any time.

Every virtual currency inside virtual accounts is pegged to a certain currency from the outside world - a blockchain asset or fiat currency. This means that one unit of the virtual currency is equal to some amount of the pegged currency.

<!-- theme: info -->

>When you create a virtual currency <i>MY_OWN_TOKEN</i> with the base pair <i>USD</i>, you can set your custom base rate. For instance, base rate 2 means that 1 MY_OWN_TOKEN = 2 USD.

When you want to connect a custom ERC-20 token to virtual accounts and utilize off-chain features, there are two scenarios:
- Connecting an existing ERC-20 token to Tatum.
- Creating your own ERC-20 token and connecting it to Tatum.

---
## Connecting an existing ERC-20 token

If you want to support an existing ERC-20 token, you only need to create a virtual currency representing the ERC-20 token inside virtual accounts. You need to enter the symbol of the existing ERC-20 (`symbol`); supply which will be credited to the virtual account inside Tatum (`supply`); a base pair (`basePair`); and the blockchain address where the initial supply was transferred (`address`). Let's use the [Tatum Test Token](https://ropsten.etherscan.io/token/0xb3858430b7ed404747b9561027d2c01a72610f43), an ERC-20 token issued for testing purposes by Tatum.

```request
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/ETH/erc20/deploy \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --header 'x-testnet-type: ' \
  --data '{
  "symbol": "ERC_SYMBOL",
  "name": "My_ERC20",
  "totalCap": "10000000",
  "supply": "10000000",
  "digits": 18,
  "address": "0xa0Ca9FF38Bad06eBe64f0fDfF279cAE35129F5C6",
  "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
  "nonce": 0,
  "fee": {
    "gasLimit": "40000",
    "gasPrice": "20"
  }
}'
```
```response
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}
```
</div>

The response includes the virtual `txId` with the account balance of the supply at the blockchain address. The account is frozen unless the virtual currency is connected to a specific ERC-20 on the blockchain.

To unfreeze the account and connect the virtual currency to a blockchain ERC-20 token, you need to provide account ID (`id`) of the virtual currency as seen in the request below.

**Request example**
```json
curl --request PUT \
  --url https://api-eu1.tatum.io/v4/tatum/account/id/unfreeze \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```

If the request is processed successfully, the response is empty. Internally, the account created in the previous step [is unfrozen](../virtualAccounts/b3A6MzEwNDI1NDk-unfreeze-account) and automatic synchronization of incoming transactions for connected accounts and addresses is started. It is also possible to create a virtual account of your virtual currency and utilize all the instant feeless virtual account transactions.

---
## Creating and connecting a new ERC-20 token

In this step, you will create a new ERC-20 on the blockchain, create a virtual currency in virtual accounts, and connect this virtual currency to the instance of the ERC-20 token. This can be done using two API calls. In the first call, you both [deploy your ERC-20 token](../virtualAccounts/b3A6MzA5MTQyMTI-create-new-virtual-currency) and create the virtual account currency.

<!-- theme: warning -->
> #### Security
>
> Blockchain transactions are signed using a private key via API, which is not a secure way of signing transactions. Your private keys and mnemonics should never leave your security perimeter. To correctly and securely sign a transaction, you can use the [Tatum CLI](https://github.com/tatumio/tatum-cli); a specific language library like [Tatum JS](https://github.com/tatumio/tatum-js); local [middleware API](https://github.com/tatumio/tatum-middleware); or our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).


<div class='tabbed-code-blocks'>
```Request
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/token \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "name": "VC_VIRTUAL",
  "supply": "1000000",
  "basePair": "AED",
  "baseRate": 1,
  "customer": {
    "accountingCurrency": "USD",
    "customerCountry": "US",
    "externalId": "123654",
    "providerCountry": "US"
  },
  "description": "My Virtual Token description.",
  "accountCode": "AC_1011_B",
  "accountNumber": "1234567890",
  "accountingCurrency": "AED"
}'
```
```Response
{
  "id": "5e68c66581f2ee32bc354087",
  "balance": {
    "accountBalance": "1000000",
    "availableBalance": "1000000"
  },
  "currency": "BTC",
  "frozen": true,
  "active": false,
  "customerId": "5e68c66581f2ee32bc354087",
  "accountCode": "03_ACC_01",
  "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka"
}
```
</div>

The response includes the `id` of the virtual account, with `accountBalance` being the supply at the blockchain address. The account is frozen (`"frozen": true`) unless the virtual currency is connected to a specific ERC-20 on the blockchain.

The second property is the `transaction ID` of the blockchain transaction that created the ERC-20 token. To obtain the contract address, you need to [get the details of the transaction](https://dxh.stoplight.io/docs/blockchain/b3A6MjgzNjM1MTY-get-transaction-by-hash-or-address). You can see the property `contractAddress`, which is the address of the ERC-20 token.

<div class='tabbed-code-blocks'>
```Request
curl --request GET \
  --url https://api-eu1.tatum.io/v4/blockchain/chainId/transaction/id \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: '
```
```Response
[
  {
    "block": {
      "hash": "000ca231a439a5c0a86a5a5dd6dc1918a8",
      "height": 500124
    },
    "hash": 1,
    "index": "d1c75a84e4bdf0dd9bf1bcd0ce4fb25f89e2ed3c5e9574dbca2760b52c428717",
    "fee": {
      "price": 50,
      "limit": 50000,
      "currency": "CELO"
    },
    "addressFrom": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "addressTo": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "token": {
      "contractAddress": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
      "currencySymbol": "BTC",
      "amount": 0
    },
    "status": "true",
    "timestamp": "2021-21-10 07:36:05"
  }
]
```
</div>


To unfreeze the account and connect the virtual currency to a blockchain ERC-20 token, you need to [provide the address of the ERC-20 token](https://tatum.io/apidoc.php#operation/storeTokenAddress) and the name of the virtual currency as seen in the request below.

<div class='tabbed-code-blocks'>
```request
curl --location --request POST 'https://api-eu1.tatum.io/v3/offchain/token/TestToken/0x5a14C4ebc2e20eEB820C5197fc408a3CB9E27B75' \
--header 'x-api-key: YOUR_API_KEY '
```
```response
curl --request PUT \
  --url https://api-eu1.tatum.io/v4/tatum/account/id/unfreeze \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```

If the request is processed successfully, the response is empty. Internally, the account created in the previous step [is unfrozen](../virtualAccounts/b3A6MzEwNDI1NDk-unfreeze-account) and automatic synchronization of incoming transactions for connected accounts and addresses is started. It is also possible to create a virtual account of your virtual currency and utilize all the instant feeless virtual account transactions.