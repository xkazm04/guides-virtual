# How to send a Bitcoin blockchain transaction from a virtual account

---

When you work with Tatum Virtual Accounts, you can perform instant transactions between virtual accounts which are not written to the underlying blockchain. But every once in a while, you need to synchronize some of the transactions to the blockchain. In this case, you have to perform a virtual-account-to-blockchain transaction. As a prerequisite, you must have a virtual account with credited blockchain transactions available. 

This type of transaction consists of 3 steps:
- [Create a withdrawal transaction](../virtualAccounts/b3A6MjgwOTI1Njk-create-withdrawal) - this will perform a virtual account transaction from the source account. It will debit the amount from the source account.
- [Perform a blockchain transaction](../virtualAccounts/b3A6MjgxMjcyNTM-blockchain-transfer) - in this step, the crypto assets are sent to the recipient's address. Any blockchain address from your blockchain wallet can be used as the source address of the blockchain transaction.
- [Complete the withdrawal transaction](../virtualAccounts/b3A6MjgwOTI1NzE-complete-withdrawal) - mark the withdrawal as successful and store the transaction ID of the blockchain transaction to the withdrawal operation. This step must be completed; otherwise, there will be inconsistencies within the virtual account state. If the blockchain transaction fails, the [withdrawal request must be canceled](../virtualAccounts/b3A6MjgwOTI1NzI-cancel-withdrawal), and the funds will be credited to the originating virtual account.

All of these actions can be performed as [one API call](../virtualAccounts/b3A6MjgwOTI1NzI-cancel-withdrawal) for a specific blockchain. The following example uses Bitcoin but the process is applicable for other blockchains as well.

<!-- theme: warning -->
> #### Security
>
> Blockchain transactions are signed using a private key via API, which is not a secure way of signing transactions. Your private keys and mnemonics should never leave your security perimeter. To correctly and securely sign a transaction, you can use [Tatum CLI](https://github.com/tatumio/tatum-cli); a specific language library like [Tatum JS](https://github.com/tatumio/tatum-js); the local [middleware API](https://github.com/tatumio/tatum-middleware); or our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).

---
## Sending Bitcoin from a virtual account to the blockchain

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/transaction/BTC \
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
We can see that the parameters `chainId`, `sender` and `receiver` are required in this call. The `reference` parameter in the response is a unique identifier within the Tatum ledger. In case of failure, use this value to search for problems.

---
## Getting virtual account transactions

For a withdrawal, a [virtual account transaction](../virtualAccounts/b3A6MjgwOTcwNDg-list-account-transactions) will be created for the source virtual account.

**Request example**
```json
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
## Getting a Bitcoin transaction

Below is an example of [getting details of a blockchain transaction](../blockchain/b3A6MjgzNjM1MTY-get-transaction-by-hash-or-address).

**Request example**
```json
curl --request GET \
  --url https://api-eu1.tatum.io/v4/blockchain/BTC/transaction/id \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: '
```
**Response example**
```json
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

The required parameters for this call are `chainId`, which represents a supported blockchain, and `id`, which is the transaction hash or the user address related with the searched transaction.
Optional parameters are `from`, `offset`, `sort` and `to`.
The response provides any available details about the transaction.

<!-- theme: info -->
>In blockchain transactions, unspent deposits from every blockchain address in the same blockchain wallet are automatically collected at the address 0. This is necessary to fully utilize Tatum's off-chain features.

---

Thanks to the unified Tatum API calls, the logic presented above is applicable for all supported blockchains.