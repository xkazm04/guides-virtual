# How to support fiat currencies

---
<!-- theme: info -->

>A fiat currency is a government-issued currency that isn't backed by a commodity such as gold.

In addition to crypto accounts, virtual accounts also support private virtual currencies. A virtual currency does not have to be connected to the blockchain. It can live within the virtual accounts alone. When you create your own virtual currency, you can then create accounts and send virtual account transactions.

<!-- theme: info -->

>A virtual currency is an entity in the virtual account distributed database. It is created with an initial supply of coins. This supply can be extended or reduced at any time.

Every virtual currency inside virtual accounts is pegged to a certain currency from the outside world - a blockchain asset or fiat currency. This means that one unit of the virtual currency is equal to some amount of the pegged currency.

<!-- theme: info -->

>When you create a virtual currency <i>VC_USD</i> with the base pair <i>USD</i>, you can set your custom base rate. For instance, base rate 2 means that 1 VC_USD = 2 USD.

---
## Creating a virtual currency

To support fiat currencies, you can [create your own virtual currency](../virtualAccounts/b3A6MzA5MTQyMTI-create-new-virtual-currency) with the base pair as the currency you want to support.

**Request example**
```JavaScript
import {Currency, Fiat, createVirtualCurrency } from '@tatumio/tatum';
/**
 * Create new virtual currency with given supply stored in account.
 * This will create Tatum internal virtual currency.
 * @param data - body of request - https://tatum.io/apidoc.php#operation/createCurrency
 * @returns virtual currency detail
 */
const body = {
  name: "VC_USD",
  supply: "100000",
  basePair: USD
  }
const vc = createVirtualCurrency (body);
```
```cURL
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
The result of the call is a virtual account of the given virtual currency. The initial supply of the virtual currency is already credited to the account.

---

## Increasing the supply of a virtual currency


When you want to [increase the supply of a virtual currency](../virtualAccounts/b3A6MzA5MTQyMTU-create-new-supply), you have to mint new units. Minted units are credited to a specific virtual account, and the operation is visible as a new virtual account transaction for that account.

**Request example**
```JavaScript
import { mintVirtualCurrency } from '@tatumio/tatum';
/**
 * Create new supply of virtual currency linked on the given accountId.
 * @param data - body of request - https://tatum.io/apidoc.php#operation/mintCurrency
 * @returns transaction internal reference
 */
const body = {
  accountId: "5fbe46739045a09adbc3f590",
  amount: "100",
  }
const mint = mintVirtualCurrency (body);
```
```cURL
curl --request PUT \
  --url https://api-eu1.tatum.io/v4/vc/token/mint \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "accountId": "5e68c66581f2ee32bc354087",
  "amount": "1.5",
  "paymentId": "My Payment",
  "reference": "akjsndakjsdn-asd-kjasnd-asdkn-asdjnasjkdn",
  "transactionCode": "1_01_EXTERNAL_CODE",
  "recipientNote": "Private note",
  "counterAccount": "5e6645712b55823de7ea82f1",
  "senderNote": "Sender note"
}'
```
**Response example**
```json
{
  "reference": "0c64cc04-5412-4e57-a51c-ee5727939bcb"
}
```
A `reference` to the virtual account transaction is given in the response.

---
## Destroying the supply of a virtual currency

When you want to [decrease the supply of a virtual currency](../virtualAccounts/b3A6MzA5MTQyMTY-burn-supply), you have to revoke some units. The revoked units are debited from the virtual account they were first credited to, and the operation is visible as a new virtual account transaction for the account.

**Request example**
```JavaScript
import { revokeVirtualCurrency } from '@tatumio/tatum';
/**
 * Destroy supply of virtual currency linked on the given accountId.
 * @param data - body of request - https://tatum.io/apidoc.php#operation/revokeCurrency
 * @returns transaction internal reference
 */
const body = {
  accountId: "5fbe46739045a09adbc3f590",
  amount: "100",
  }
const burn = revokeVirtualCurrency (body);
```
```cURL
curl --request PUT \
  --url https://api-eu1.tatum.io/v4/vc/token/revoke \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "accountId": "5e68c66581f2ee32bc354087",
  "amount": "1.5",
  "paymentId": "My Payment",
  "reference": "akjsndakjsdn-asd-kjasnd-asdkn-asdjnasjkdn",
  "transactionCode": "1_01_EXTERNAL_CODE",
  "recipientNote": "Private note",
  "counterAccount": "5e6645712b55823de7ea82f1",
  "senderNote": "Sender note"
}'
```
**Response example**
```json
{
  "reference": "0c64cc04-5412-4e57-a51c-ee5727939bcb"
}
```
A `reference` to the virtual account transaction is given in the response.

---
## Listing account transactions

When we get the [list of account transactions](../virtualAccounts/b3A6MjgwOTcwNDg-list-account-transactions) now, we can see two operations - mint and revoke.

**Request example**
```JavaScript
import {getTransactionsByAccount} from '@tatumio/tatum';
/**
 * Finds transactions for the account identified by the given account ID.
 * @param filter - request body with data filter - https://tatum.io/apidoc.php#operation/getTransactionsByAccountId
 * @param pageSize - max number of items per page is 50.
 * @param offset - optional Offset to obtain next page of the data.
 */
const filter = {
  id: "5fbe46739045a09adbc3f590",
  }
const tx = getTransactionsByAccount(filter,50,0);
```
```cURL
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/transaction/account \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "id": "5e6be8e9e6aa436299950c41",
  "counterAccount": "5e6be8e9e6aa436299950c41",
  "from": 1571833231000,
  "to": 1571833231000,
  "currency": "BTC",
  "amount": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "currencies": [
    "BTC"
  ],
  "transactionType": "FAILED",
  "transactionTypes": [
    "FAILED"
  ],
  "opType": "PAYMENT",
  "transactionCode": "1_01_EXTERNAL_CODE",
  "paymentId": "65426",
  "recipientNote": "65426",
  "senderNote": "65426"
}'
```

In the response, we can see the two transactions we have just performed - **mint** and **revoke**.

**Response example**
```json
[
  {
    "accountId": "5e6645712b55823de7ea82f1",
    "counterAccountId": "5e6645712b55823de7ea82f1",
    "currency": "BTC",
    "amount": "0.1",
    "anonymous": false,
    "created": 1572031674384,
    "marketValue": {
      "amount": "1235.56",
      "currency": "EUR",
      "sourceDate": 1572031674384,
      "source": "fixer.io"
    },
    "operationType": "PAYMENT",
    "transactionType": "CREDIT_PAYMENT",
    "reference": "5e6be8e9e6aa436299950c41",
    "transactionCode": "1_01_EXTERNAL_CODE",
    "senderNote": "Sender note",
    "recipientNote": "Private note",
    "paymentId": "65426",
    "attr": "123",
    "address": "qrppgud79n5h5ehqt9s7x8uc82pcag82es0w9tada0",
    "txId": "c6c176e3f6705596d58963f0ca79b34ffa5b78874a65df9c974e22cf86a7ba67"
  }
]
```
---
## Typical integration steps
Due to legislative restrictions, it is impossible to integrate Tatum directly into a specific bank or card processor to read and perform bank transactions. The bank integration must be done in your application and reflect your bank operations into Tatum.

The typical flow is as follows:

- When you receive a payment into your bank account or via a card transaction, you can mint a new supply on the connected user account.
- When you send a payment from your bank account, you can revoke the supply from the connected user account.
- You will then perform all internal transactions within the Tatum virtual accounts and your application ecosystem.

For more information on how to support virtual accounts and currencies in a crypto exchange, please refer to the following workshop:

https://www.youtube.com/watch?v=CGgyyTTv0yw&t=209s&ab_channel=Tatum
