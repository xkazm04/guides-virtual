# How to send a feeless instant transaction

---

Not every transaction should be recorded on the blockchain. To be more precise, almost no transaction is significant enough in and of itself to be stored on the blockchain. Why? Every transaction incurs a transaction fee. And it makes no sense to pay, for example, $1 for a transaction worth $5 in bitcoin.

To avoid these types of transactions and the fees that invariably come with them, virtual accounts are an ideal solution.

<!-- theme: success -->

> #### Tatum Virtual Accounts
>
> Tatum Virtual Accounts are a universal [Layer-2](https://academy.binance.com/en/glossary/layer-2) solution that can be plugged into any blockchain supported by Tatum.

It's both extremely easy and highly practical to utilize virtual accounts to send instant transactions. As a prerequisite, you must have at least two virtual accounts with assets on at least one of them.

---
## Performing a transaction between virtual accounts

A virtual account transaction is a transaction that is settled immediately with no blockchain fee incurred. It is a transaction between two virtual accounts with the same account currency, e.g. Bitcoin accounts, Ethereum accounts, or your custom fiat accounts.


<!-- theme: info -->

> #### Tatum Virtual Accounts
>Tatum supports microtransactions. You can send as little as 1/1000000 of a Satoshi (Bitcoin denomination) or 1/1000000 of a Wei (Ethereum denomination) between virtual accounts.

**Request example**
```json
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
**Response example**
```json
{
  "reference": "9e179f90-0221-480f-adb4-28bd1937bb20"
}
```


<!-- theme: info -->

> #### Tatum Virtual Accounts
>The market value in the transaction is part of the compliance engine built into Tatum Virtual Accounts. It records the exact value of the transaction in fiat currencies - US Dollars, Euros, etc. Currency is obtained from the account property `accountingCurrency`.

As you see, performing instant transactions with Tatum Virtual Accounts is really easy.