# How to block amount in a virtual account

---

Most blockchains do not allow blocking or freezing assets at blockchain addresses. But in real life, many applications must block blockchain assets before the actual transaction. It is the same as with a debit card and a real bank account. At first, funds are only blocked and the transaction is processed a couple of days later.

There are two types of balances in Tatum Virtual Accounts:

- **Account balance** - a sum of all funds that are present in the account

- **Available balance** - a sum of all funds that are not blocked and are therefore available for spending

<!-- theme: info -->
>Available balance can be negative due to blockages in an account. In such cases transactions cannot be performed, but new blockages can still be made.

Every blockage has its unique identifier and details, like type or custom description.

---
## Blocking funds in a virtual account

To block funds in an account identified by `id`, you are required to state the `amount` and `type`. The response is the `id` of the newly created blockage.

**Request example**
```JavaScript
import {blockAmount} from '@tatumio/tatum';
/**
 * Blocks an amount in an account.
 * Any number of distinct amounts can be blocked in one account.
 * @param id - ledger account ID
 * @param body - request body with blocked amount - https://tatum.io/apidoc.php#operation/blockAmount
 */
const body = {
  "amount": "5",
  "type": "DEBIT_CARD_OP",
  "description": "Card payment in the shop."
  }
const txId = blockAmount("5fbaca3001421166273b3779", body);
```
```cURL
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/account/block/id \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "amount": "5",
  "type": "DEBIT_CARD_OP",
  "description": "Card payment in the shop."
}'
```
**Response example**
```json
{
  "id": "5e68c66581f2ee32bc354087"
}
```
---
## Getting balances

You can now send a request to get the available balance of the account where funds have just been blocked.

**Request example**
```JavaScript
import {getAccountById} from '@tatumio/tatum';
/**
 * Gets active account by ID.
 * Displays all information regarding the given account - https://tatum.io/apidoc.php#operation/getAccountByAccountId
 * @param id - ledger account ID
 */
const account = getAccountById("5fbaca3001421166273b3779");
```
```cURL
curl --request GET \
  --url https://api-eu1.tatum.io/v4/blockchain/BTC/account/address/balance \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: '
```
**Response example**
```json
{
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0.000001",
        "availableBalance": "-4.999999"
    },
    "accountCode": null,
    "accountNumber": null,
    "frozen": false,
    "xpub": "tpubDF1sYuDKCJr6mGietaVzqGmF2dqdKVBa1DtLJGBX8HXhtHZPv5UBz3WNWU22tiVAYSjqfvfFxMnDs3vM11iQrKej6dq33UCevhiPW9EQAS2",
    "accountingCurrency": "EUR",
    "id": "5fbaca3001421166273b3779"
}
```
---

## Getting all blockages in an account

As you can see above, there are five bitcoins blocked in the account, so the available balance is now negative. Let's get all blockages in this account to see if there are any more.

**Request example**
```JavaScript
import { getBlockedAmountsByAccountId  } from '@tatumio/tatum';
/**
 * Gets blocked amounts for an account.
 * @param id - account ID
 * @param pageSize - max number of items per page is 50
 * @param offset - optional Offset to obtain next page of the data
 * @returns - detail of blocked amounts - https://tatum.io/apidoc.php#operation/getBlockAmount
 */
 
const amounts = getBlockedAmountsByAccountId("5fbaca3001421166273b3779?pageSize=50");
```
```cURL
curl --request GET \
  --url https://api-eu1.tatum.io/v4/tatum/account/block/id \
  --header 'Content-Type: application/json'
```
**Response example**
```json
[
    {
        "amount": "5",
        "type": "DEBIT_CARD_OP",
        "description": "Card payment in the shop.",
        "accountId": "5fbaca3001421166273b3779",
        "id": "5fbe2b9b99166cb792cba6f2"
    }
]
```
In this case, there is just one blockage in the account.

---
## Unblocking an amount in an account

When you want to unblock an amount from an account, you have several options:
- **unblock a specific blockage** - this will unblock the whole amount and the available balance will be increased
- **unblock a blockage and perform a virtual account transaction** - this will unblock the amount from the account and perform a transaction to a different account right away
- **unblock all blocked amounts in an account** - this will unblock all blockages in the account

Let's unblock a blockage without a transaction. You'll need to provide the ID of the specific blockage you want to unblock.

**Request example**
```JavaScript
import {deleteBlockedAmount} from '@tatumio/tatum';
/**
 * Unblocks a previously blocked amount in an account.
 * Increases the available balance in the account where the amount was blocked.
 * @param id - transaction ID of blocked amount
 */
const account = deleteBlockedAmount ("5fbe2b9b99166cb792cba6f2");
```
```cURL
curl --request DELETE \
  --url https://api-eu1.tatum.io/v4/tatum/account/block/id \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```

The response will be empty if the request is processed successfully. 

<!-- theme: info -->
>After deleting all blockages in an account, the available balance is equal to the account balance. You can check this  by getting the account details.

---

Now you know how to block and unblock funds in an account.
