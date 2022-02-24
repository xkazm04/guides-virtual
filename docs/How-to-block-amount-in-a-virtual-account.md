# How to block amounts in a virtual account

---

Most blockchains do not allow the blocking or freezing of assets at blockchain addresses. But in real life, many applications must block blockchain assets before the actual transaction. It is the same as with a debit card and a real bank account. First, funds are blocked, and then the transaction is processed a couple of days later.

When you work with Tatum Virtual Accounts, there are two types of balances:
- **account balance** - a sum of all funds that are present in the account
- **available balance** - a sum of all funds that are available for spending

<div class="toolbar-note">
The available balance can even be negative when there are blockages in the account. When the balance is below zero, transactions cannot be performed, but new blockages can be made.
</div>

Blockages affect the available balance of the account. Every new blockage has its unique identifier and details, like its type or custom description.

---

## Blocking funds in a virtual account

[To block funds in an account](https://developer.tatum.io/rest/virtual-accounts/block-an-amount-in-an-account), the account ID, amount, and type are required parameters. 


<div class='tabbed-code-blocks'>
```SDK
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
```REST API call
curl --location --request POST 'https://api-eu1.tatum.io/v3/ledger/account/block/5fbaca3001421166273b3779' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "amount": "5",
    "type": "DEBIT_CARD_OP",
    "description": "Card payment in the shop."
}'
```
</div>

The response is the identifier of the blockage.

```Response
{
    "id": "5fbe2b9b99166cb792cba6f2"
}
```

---

## Getting an account's details

You can now [get details about the account](https://developer.tatum.io/rest/virtual-accounts/get-account-by-id) where we made the blockage to see what has happened with the balance.

<div class='tabbed-code-blocks'>
```SDK
import {getAccountById} from '@tatumio/tatum';
/**
 * Gets active account by ID.
 * Displays all information regarding the given account - https://tatum.io/apidoc.php#operation/getAccountByAccountId
 * @param id - ledger account ID
 */
const account = getAccountById("5fbaca3001421166273b3779");
```
```json
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/account/5fbaca3001421166273b3779' \
--header 'x-api-key: YOUR_API_KEY'
```
</div>


The response will contain the details of the virtual account associated with the specified account ID.

**Response:**
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

## Getting all blockages in the account

As you can see, there were 5 bitcoins blocked in the account, so the available balance is negative. Now you can [get all blockages in the specific account](https://developer.tatum.io/rest/virtual-accounts/get-blocked-amounts-in-an-account).


<div class='tabbed-code-blocks'>
```SDK
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
```REST API call
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/account/block/5fbaca3001421166273b3779?pageSize=50' \
--header 'x-api-key: YOUR_API_KEY'
```
</div>

The response will contain a list of all current blockages for the account ID.


**Response:**
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

When you want to unblock an amount from the account, you have several options:
- **unblock a specific blockage** - this will unblock the whole amount, and the account's available balance will be increased.
- **unblock a blockage and perform a virtual account transaction** - this will unblock the amount from the account and perform a transaction to a different account.
- **unblock all blocked amounts in the account** - this will unblock all blockages in the account.

---

## Unblocking an amount in the account

Let's [unblock a blockage without a transaction](https://developer.tatum.io/rest/virtual-accounts/unblock-a-blocked-amount-in-an-account). You'll need to pass the ID of the specific blockage you want to unblock.

<div class='tabbed-code-blocks'>
```SDK
import {deleteBlockedAmount} from '@tatumio/tatum';
/**
 * Unblocks a previously blocked amount in an account.
 * Increases the available balance in the account where the amount was blocked.
 * @param id - transaction ID of blocked amount
 */
const account = deleteBlockedAmount ("5fbe2b9b99166cb792cba6f2");
```
```json
curl --location --request DELETE 'https://api-eu1.tatum.io/v3/ledger/account/block/5fbe2b9b99166cb792cba6f2' \
--header 'x-api-key: YOUR_API_KEY'
```
</div>

There will be an empty response if the request is successful. Now, when you check the details of the account, you will see the **availableBalance** is the same as the **accountBalance**.

<div class='tabbed-code-blocks'>
```SDK
import {getAccountById} from '@tatumio/tatum';
/**
 * Gets active account by ID.
 * Displays all information regarding the given account - https://tatum.io/apidoc.php#operation/getAccountByAccountId
 * @param id - ledger account ID
 */
const account = getAccountById("5fbaca3001421166273b3779");
```
```REST API call
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/account/5fbaca3001421166273b3779' \
--header 'x-api-key: YOUR_API_KEY'
```
```Response
{
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0.000001",
        "availableBalance": "0.000001"
    },
    "accountCode": null,
    "accountNumber": null,
    "frozen": false,
    "xpub": "tpubDF1sYuDKCJr6mGietaVzqGmF2dqdKVBa1DtLJGBX8HXhtHZPv5UBz3WNWU22tiVAYSjqfvfFxMnDs3vM11iQrKej6dq33UCevhiPW9EQAS2",
    "accountingCurrency": "EUR",
    "id": "5fbaca3001421166273b3779"
}
```
</div>

You can test other unblocking methods or read more about the specific endpoints in the [API Reference](https://developer.tatum.io/rest/virtual-accounts/ block-an-amount-in-an-account). 
























































































































































