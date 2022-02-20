# How to withdraw assets from virtual accounts with gas pump wallets

---
This guide describes how to withdraw assets from virtual accounts connected to gas pump wallets. We will cover the whole flow of the user journey here - from gas pump wallet generation to virtual account creation to withdrawal from a gas pump wallet to a blockchain address with correct virtual account balance updates.

---
## Wallet generation !THIS API CALL IS DEPRECATED!

The first step is to [create a gas pump wallet](https://tatum.io/apidoc.php#operation/GenerateCustodialWallet) for every user. You can choose which ERC-* standards you want to support in the wallet - native assets like ETH or BSC are supported by default. In this example, we will support the native currency (MATIC) on Polygon and any ERC-20 tokens running on the Polygon network.

**Request example**

```json
curl --location --request POST 'https://api-eu1.tatum.io/v3/blockchain/sc/custodial' \
--header 'Content-Type: application/json' \
--header 'x-api-key: YOUR_API_KEY' \
--data-raw '{
    "enableFungibleTokens": true,
    "enableNonFungibleTokens": false,
    "enableSemiFungibleTokens": false,
    "enableBatchTransactions": true,
    "chain": "MATIC",
    "fromPrivateKey": "0x37b091fc4ce46a56da643f021254612551dbe0944679a6e09cb5724d3085c9ab"
}'
```

**Response example**

```json
{
    "txId": "0x2979a3d30f812b0d9ebcc66d01b620d3779380c38e8770a82fa4ea4f2dfa8f69"
}
```
The response gives us a hash of the transaction. We can [obtain the contract address](https://tatum.io/apidoc.php#operation/SCGetContractAddress) - our gas pump address - from this hash.

---
## Virtual account creation

Now we need to [create virtual accounts](../virtualAccounts/b3A6MzA5MTQyMTI-create-new-virtual-currency). Since we have decided to support two currencies, MATIC and USDC, we need to create two virtual accounts for every user. These accounts do not have xpubs because we will manually assign the address to them later.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/account \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "currency": "BTC",
  "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
  "customer": {
    "accountingCurrency": "USD",
    "customerCountry": "US",
    "externalId": "123654",
    "providerCountry": "US"
  },
  "compliant": false,
  "accountCode": "AC_1011_B",
  "accountingCurrency": "AED",
  "accountNumber": "123456"
}'
```
**Response example**
```json
{
  "id": "5e68c66581f2ee32bc354087",
  "balance": {
    "accountBalance": "1000000",
    "availableBalance": "1000000"
  },
  "currency": "BTC",
  "frozen": false,
  "active": true,
  "customerId": "5e68c66581f2ee32bc354087",
  "accountCode": "03_ACC_01",
  "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka"
}
```
Once an account has been created, we need to [assign the gas pump address](../virtualAccounts/b3A6MzEwNDI1NTU-asssign-address-to-account) to it. This process enables automatic incoming transaction synchronization for the specified address and currency.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/account/id/address/address \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```
**Response example**
```json
{
  "address": "7c21ed165e294db78b95f0f181086d6f",
  "currency": "BTC",
  "derivationKey": -2147483648,
  "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka",
  "destinationTag": 5,
  "memo": "5",
  "message": "5"
}
```
The response contains a blockchain `address` that has been connected to a virtual account. Any incoming blockchain transaction to this address will be automatically detected and the account balance will be updated.

---

## Withdrawing assets from an account

The following three steps are necessary to withdraw assets from a virtual account and send them to the blockchain:
- Create a request for withdrawal from the virtual account - the withdrawn balance will be debited to the virtual account
- Transfer the assets from the gas pump address - this will result in the blockchain transaction
- Mark the withdrawal from step one as completed.

Let's withdraw 1 USDC from an account to the blockchain.

#### Perform a withdrawal

First you need to send a [request for withdrawal](../virtualAccounts/b3A6MjgwOTI1Njk-create-withdrawal) from the virtual account so that, after performing the blockchain operation, the virtual account would be synchronized.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/withdrawal \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "senderAccountId": "5e68c66581f2ee32bc354087",
  "address": "mpTwPdF8up9kidgcAStriUPwRdnE9MRAg7",
  "amount": "0.001",
  "attr": "12345",
  "compliant": false,
  "fee": "0.0005",
  "multipleAmounts": [
    "0.1"
  ],
  "paymentId": "12345",
  "senderNote": "Sender note"
}'

```
**Response example**

```json
{
  "reference": "5e6be8e9e6aa436299950c41",
  "data": [
    {
      "address": {
        "address": "7c21ed165e294db78b95f0f181086d6f",
        "currency": "BTC",
        "derivationKey": 2147483647,
        "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka",
        "destinationTag": 5,
        "memo": "5",
        "message": "5"
      },
      "amount": 0,
      "vIn": "string",
      "vInIndex": 0,
      "scriptPubKey": "string"
    }
  ],
  "id": "5e68c66581f2ee32bc354087"
}
```
The response contains a virtual account transaction `reference` and a withdrawal `id`. The account balance has been debited with 1 USDC and the transaction is visible in the transaction list of the virtual account. So far, nothing has been done on the blockchain.

#### Perform a blockchain transaction

Now it's time to [move the assets to another address](../virtualAccounts/b3A6MjgxMjcyNTM-blockchain-transfer). You don't have to send any MATIC there to pay for gas. You only need to specify which assets should be sent, how much, and where.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/transaction/chainId \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "sender": {
    "keyPair": [
      {
        "address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
        "privateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
        "mnemonic": "urge pulp usage sister evidence arrest palm math please chief egg abuse"
      }
    ],
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "accountId": "5e68c66581f2ee32bc354087"
  },
  "receiver": [
    {
      "address": "0x89205A3A3b2A69De6Dbf7f0",
      "token": {
        "address": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
        "symbol": "LINK",
        "amount": "5"
      }
    }
  ],
  "paymentId": "9625",
  "note": "Hello there",
  "compliant": false,
  "fee": {
    "price": 50,
    "limit": 50000,
    "currency": "CELO"
  }
}'
```
**Response example**
```json
{
  "reference": "0c64cc04-5412-4e57-a51c-ee5727939bcb"
}
```
The response contains a `reference` property which represents a unique identifier within the Tatum ledger. In case of a failure, use this value to search for problems.
#### Complete or cancel the withdrawal

Once the transaction is successful, you need to [mark the withdrawal as completed](../virtualAccounts/b3A6MjgwOTI1NzE-complete-withdrawal). If the transaction fails and you want to refund the assets to the virtual account, you need to [cancel the original withdrawal request](../virtualAccounts/b3A6MjgwOTI1NzI-cancel-withdrawal).

---

These five simple steps are all you need to do to connect your gas pump wallets to virtual accounts and keep them synced.