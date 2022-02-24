# How to send a feeless instant transaction

---

Not every transaction should be recorded on the blockchain. To be more precise, almost no transaction is significant enough in and of itself to be stored on the blockchain. Why? Every transaction incurs a transaction fee. And it makes no sense to pay, for example, $1 for a transaction worth $5 in bitcoin.

To avoid these types of transactions and the fees that invariably come with them, virtual accounts are an ideal solution.

It's both extremely easy and highly practical to utilize virtual accounts to send instant transactions. As a prerequisite, you must have at least two virtual accounts with assets on at least one of them.

[How to automatically scan blockchain addresses and set up webhook notifications](url)

---
## Performing a transaction between virtual accounts

A [virtual account transaction](https://developer.tatum.io/rest/virtual-accounts/send-payment) is a transaction that is settled immediately with no blockchain fee incurred. It is a transaction between two virtual accounts with the same account currency, e.g. Bitcoin accounts, Ethereum accounts, or your custom fiat accounts.


<div class="toolbar-note">
#### Tatum Virtual Accounts
Tatum supports microtransactions. You can send as little as 1/1000000 of a Satoshi (Bitcoin denomination) or 1/1000000 of a Wei (Ethereum denomination) between virtual accounts.
</div>


<div class='tabbed-code-blocks'>
```SDK
import {storeTransaction} from '@tatumio/tatum';
/**
 * Sends a payment from one virtual account to another
 * @param transaction - body of the request - https://tatum.io/apidoc.php#operation/sendTransaction
 */
const body = {
  senderAccountId: "5fad2aa1cac7f2e8aeac0e6b",
  recipientAccountId: "5fbaca3001421166273b3779",
  amount: "0.000001",
  anonymous: false,
  compliant: false,
  transactionCode: "1_01_EXTERNAL_CODE",
  paymentId: "9625",
  recipientNote: "Private note",
  baseRate: 1,
  senderNote: "Sender note"
};
const txReference = storeTransaction(body);
```
```REST API call
curl --location --request 
POST 'https://api-eu1.tatum.io/v4/ledger/transaction' \
--header 'x-api-key: YOUR_API_KEY ' \ 
--header 'Content-Type: application/json' \
--data-raw '{
  "senderAccountId": "5fad2aa1cac7f2e8aeac0e6b",
  "recipientAccountId": "5fbaca3001421166273b3779",    
  "amount": "0.000001"
}'
```
</div>

The response contains a transaction `reference` ID that can be used to get the details of the transaction.

**Response:**
```json
{
  "reference": "9e179f90-0221-480f-adb4-28bd1937bb20"
}
```

## Getting the details of the transaction

When you perform a [virtual account transaction](https://tatum.io/apidoc#operation/getTransactionsByReference), there are actually two transactions written inside the virtual accounts - one for the sender and one for the recipient of the transaction, each with slightly different data.

- the accountID and counterAccountID for the transaction are opposite
- the transactionType is DEBIT or CREDIT based on the sender and recipient account
- the marketValue might be different based on the settings of the accounts in the transaction

To get the details of both transactions, enter the reference ID of the virtual account transaction from the previous step into the following API call:

<div class='tabbed-code-blocks'>
```SDK
import {getTransactionsByReference} from '@tatumio/tatum';
/**
 * Finds transactions for all accounts with the given reference.
 * @param reference - transaction reference within Tatum Private Ledger
 */
 
const tx = getTransactionsByReference("9e179f90-0221-480f-adb4-28bd1937bb20");
```
```REST API call
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/transaction/reference/9e179f90-0221-480f-adb4-28bd1937bb20' \
--header 'x-api-key: YOUR_API_KEY '
```
</div>

**Response:**
```json
[
    {
        "amount": "-0.000001",
        "operationType": "PAYMENT",
        "currency": "BTC",
        "transactionType": "DEBIT_PAYMENT",
        "accountId": "5fad2aa1cac7f2e8aeac0e6b",
        "anonymous": false,
        "reference": "9e179f90-0221-480f-adb4-28bd1937bb20",
        "counterAccountId": "5fbaca3001421166273b3779",
        "senderNote": null,
        "recipientNote": null,
        "paymentId": null,
        "transactionCode": null,
        "marketValue": {
            "currency": "EUR",
            "source": "CoinGecko",
            "sourceDate": 1606076765607,
            "amount": "-0.01566745999999999889"
        },
        "created": 1606076989819
    },
    {
        "amount": "0.000001",
        "operationType": "PAYMENT",
        "currency": "BTC",
        "transactionType": "CREDIT_PAYMENT",
        "accountId": "5fbaca3001421166273b3779",
        "anonymous": false,
        "reference": "9e179f90-0221-480f-adb4-28bd1937bb20",
        "counterAccountId": "5fad2aa1cac7f2e8aeac0e6b",
        "senderNote": null,
        "recipientNote": null,
        "paymentId": null,
        "transactionCode": null,
        "marketValue": {
            "currency": "EUR",
            "source": "CoinGecko",
            "sourceDate": 1606076765607,
            "amount": "0.01566745999999999889"
        },
        "created": 1606076989819
    }
]
```

<div class="toolbar-note">
#### Tatum Virtual Accounts
The market value in the transaction is part of the compliance engine built into Tatum Virtual Accounts. It records the exact value of the transaction in fiat currencies - US Dollars, Euros, etc. Currency is obtained from the account property `accountingCurrency`.
</div>

---

## And that's it!

You can easily perform instant transactions and get details about them with 2 API calls. You can also obtain a [list of transactions for the specific account](https://developer.tatum.io/rest/virtual-accounts/find-transactions-for-account) or [find customer transactions across multiple accounts](https://developer.tatum.io/rest/virtual-accounts/find-transactions-for-a-customer-across-all-of-the-customer-s-accounts).