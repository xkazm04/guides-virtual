# How to connect a custom ERC-20 token to a virtual account

---

Tatum Virtual Accounts can support ERC-20 tokens whether they are existing or not out-of-the-box. To support your own ERC-20 token and utilize all the off-chain features, you must create your own virtual currency inside a virtual account. When you produce your own virtual currency, you can create accounts and send virtual account and blockchain transactions.

[How to create an ERC-20 token]()

<div class="toolbar-note">
A virtual currency is an entity in the virtual account distributed database. It is created with an initial supply of coins. This supply can be extended or reduced at any time.
</div>

Every virtual currency inside virtual accounts is pegged to a certain currency from the outside world - a blockchain asset or fiat currency. This means that one unit of the virtual currency is equal to some amount of the pegged currency.

<div class="toolbar-note">
When you create a virtual currency `MY_OWN_TOKEN` with the base pair `USD`, you can set your custom base rate. For instance, base rate 2 means that 1 MY_OWN_TOKEN = 2 USD.
</div>

When you want to connect a custom ERC-20 token to virtual accounts and utilize off-chain features, there are two scenarios:
- Connecting an existing ERC-20 token to Tatum.
- Creating your own ERC-20 token and connecting it to Tatum.

---
## Connecting an existing ERC-20 token

If you want to support an existing ERC-20 token, you only need to [create a virtual currency representing the ERC-20](https://docs.tatum.io/rest/virtual-accounts/register-new-erc-20-token-in-the-ledger) token inside virtual accounts. You need to enter the symbol of the existing ERC-20 (`symbol`); supply which will be credited to the virtual account inside Tatum (`supply`); a base pair (`basePair`); and the blockchain address where the initial supply was transferred (`address`). Let's use the [Tatum Test Token](https://ropsten.etherscan.io/token/0xb3858430b7ed404747b9561027d2c01a72610f43), an ERC-20 token issued for testing purposes by Tatum.

<div class='tabbed-code-blocks'>
```Request
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
```Response
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}
```
</div>

The call's result is the virtual account's ID, with an account balance of the supply at the blockchain address. The account is frozen unless the virtual currency is connected to a specific ERC-20 on the blockchain.

[Off-chain]()

To unfreeze the account and connect the virtual currency to a blockchain ERC-20 token, you need to [provide the address of the ERC-20 token](https://docs.tatum.io/rest/virtual-accounts/set-erc-20-bep-20-hrm-20-trc-20-kcs-20-token-contract-address) and the name of the virtual currency.


<div class='tabbed-code-blocks'>
```json
curl --request PUT \
  --url  https://api-eu1.tatum.io/v3/ledger/account/id/freeze \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```
</div>

The request does not have any response when successful. Internally, it will [unfreeze the account](https://docs.tatum.io/rest/virtual-accounts/unfreeze-account) created in the previous step and start the automatic synchronization of incoming transactions for connected accounts and addresses. It is also possible to create a virtual account of your virtual currency and utilize all the instant feeless virtual account transactions.

---
## Creating and connecting a new ERC-20 token

In this step, you will create a new ERC-20 on the blockchain, create a virtual currency in virtual accounts, and connect this virtual currency to the instance of the ERC-20 token. This can be done using two API calls. In the first call, you both [deploy your ERC-20 token](https://docs.tatum.io/rest/virtual-accounts/deploy-ethereum-erc-20-smart-contract-to-blockchain-and-ledger) and create the virtual account currency.

#### Security
<div class="toolbar-caution">
Blockchain transactions are signed using a private key via API, which is not a secure way of signing transactions. Your private keys and mnemonics should never leave your security perimeter. To correctly and securely sign a transaction, you can use the [Tatum CLI](https://github.com/tatumio/tatum-cli); a specific language library like [Tatum JS](https://github.com/tatumio/tatum-js); local [middleware API](https://github.com/tatumio/tatum-middleware); or our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).
</div>

<div class='tabbed-code-blocks'>
```Request
curl --request POST \
  --url https://api-eu1.tatum.io/v3/ledger/virtualCurrency \
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

The second property is the `transaction ID` of the blockchain transaction that created the ERC-20 token. To obtain the contract address, you need to [get the details of the transaction](https://docs.tatum.io/rest/blockchain/get-ethereum-transaction). You can see the property `contractAddress`, which is the address of the ERC-20 token.

To unfreeze the account and connect the virtual currency to a blockchain ERC-20 token, you need to [provide the address of the ERC-20 token](https://docs.tatum.io/rest/virtual-accounts/set-erc-20-bep-20-hrm-20-trc-20-kcs-20-token-contract-address) and the name of the virtual currency as seen in the request below.

<div class='tabbed-code-blocks'>
```Create currency
curl --location --request POST 'https://api-eu1.tatum.io/v3/offchain/token/TestToken/0x5a14C4ebc2e20eEB820C5197fc408a3CB9E27B75' \
--header 'x-api-key: YOUR_API_KEY '
```
</div>

The request does not have any response when successful. Internally, it will [unfreeze the account](https://docs.tatum.io/rest/virtual-accounts/unfreeze-account) created in the previous step and start the automatic synchronization of incoming transactions for connected accounts and addresses. It is also possible to create a virtual account of your virtual currency and utilize all the instant feeless virtual account transactions.