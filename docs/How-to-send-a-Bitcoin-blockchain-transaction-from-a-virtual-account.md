# How to send a Bitcoin blockchain transaction from a virtual account

When you work with Tatum Virtual Accounts, you can perform instant transactions between virtual accounts which are not written to the underlying blockchain. But every once and a while, you need to synchronize some of the transactions to the blockchain. In this case, you have to perform a virtual account-to-blockchain transaction. As a prerequisite, you must have a virtual account with credited blockchain transactions available. 

This type of transaction consists of 3 steps:
- [Create a withdrawal transaction](https://docs.tatum.io/rest/virtual-accounts/store-withdrawal) - this will perform a virtual account transaction from the source account. It will debit the amount from the source account.
- [Perform a blockchain transaction](https://docs.tatum.io/rest/blockchain/send-bitcoin-to-blockchain-addresses) - in this step, the crypto assets are sent to the recipient's address. Any blockchain address from your blockchain wallet can be used as the source address of the blockchain transaction.
- [Complete the withdrawal transaction](https://docs.tatum.io/rest/virtual-accounts/complete-withdrawal) - mark the withdrawal as successful and store the transaction ID of the blockchain transaction to the withdrawal operation. This step must be completed; otherwise, there will be inconsistencies within the virtual account state. If the blockchain transaction fails, the [withdrawal request must be canceled](https://docs.tatum.io/rest/virtual-accounts/complete-withdrawal), and the funds will be credited to the originating virtual account.

All of these actions can be performed as [one API call](https://docs.tatum.io/rest/virtual-accounts/cancel-withdrawal) for a specific blockchain. The following example uses Bitcoin but the process is applicable for other blockchains as well.
¨
#### Security
<div class="toolbar-warning">
In this guide, blockchain transactions are being signed using a private key via API.
This is fine for testing and demo purposes, but for production use, it is not a secure way of signing transactions. 
**Your private keys and mnemonics should never leave your security perimeter**. To correctly and securely sign a transaction, you can use [Tatum CLI](https://github.com/tatumio/tatum-cli); a specific language library like [Tatum JS](https://github.com/tatumio/tatum-js); the local [middleware API](https://github.com/tatumio/tatum-middleware); or our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).
</div>

---
## Sending Bitcoin from a virtual account to the blockchain


This operation will [withdraw BTC from a virtual account to a blockchain address](https://docs.tatum.io/rest/virtual-accounts/send-bitcoin-from-tatum-account-to-address).


<div class='tabbed-code-blocks'>
```SDK
import {sendBitcoinOffchainTransaction} from '@tatumio/tatum';
/**
 * Send Bitcoin from Tatum account to address.
 * This will create Tatum internal withdrawal request with ID.
 * @param body - request body with data filter - https://tatum.io/apidoc.php#operation/BtcTransfer
 * @param testnet - true if testnet, false if mainnet
 */
const body = {
  senderAccountId: "5fbaca3001421166273b3779",
  address: "mpTwPdF8up9kidgcAStriUPwRdnE9MRAg7",
  amount: "0.00195",
  fee: "0.00005",
  mnemonic: "behave season capable ridge repair creek seat rescue potato divide fox expose wrestle asthma luggage rack afford pistol ridge modify direct picnic magic cannon",
  xpub: "tpubDF1sYuDKCJr6mGietaVzqGmF2dqdKVBa1DtLJGBX8HXhtHZPv5UBz3WNWU22tiVAYSjqfvfFxMnDs3vM11iQrKej6dq33UCevhiPW9EQAS2"
  }
const tx = sendBitcoinOffchainTransaction(false, body);
```
```REST API call
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
</div>

**Response:**

<div class='tabbed-code-blocks'>
```json
{
  "reference": "0c64cc04-5412-4e57-a51c-ee5727939bcb"
}
```
</div>

We can see that the parameters `chainId`, `sender` and `receiver` are required in this call. The `reference` parameter in the response is a unique identifier within the Tatum ledger. In case of failure, use this value to search for problems.

---
## Getting virtual account transactions

For a withdrawal, a [virtual account transaction](https://docs.tatum.io/rest/virtual-accounts/b3A6MzA2MjE3NTY-find-transactions-for-account) will be created for the source virtual account. To look up the details of this withdrawal transaction, use the [find transactions for account](https://docs.tatum.io/rest/virtual-accounts/find-transactions-for-account) endpoint:


<div class='tabbed-code-blocks'>
```SDK
import {getTransactionsByAccount} from '@tatumio/tatum';
/**
 * Finds transactions for the account identified by the given account ID.
 * @param filter - request body with data filter - https://tatum.io/apidoc.php#operation/getTransactionsByAccountId
 * @param pageSize - max number of items per page is 50.
 * @param offset - optional Offset to obtain next page of the data.
 */
const filter = {
  id: "5fbc208c99a159b4e9120c30",
  }
const tx = getTransactionsByAccount(filter,50,0);
```
```REST API call
curl --request POST \
  --url https://api-eu1.tatum.io/v3/ledger/transaction/account \
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
</div>

The response will contain the details of all transactions from the given account.

**Response:**
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

Using the transaction ID of the withdrawal transaction, you can [get the details of the blockchain transaction](https://docs.tatum.io/rest/blockchain/get-transaction-by-hash).


<div class='tabbed-code-blocks'>
```SDK
import { btcGetTransaction } from '@tatumio/tatum';
/**
 * @param hash - transaction hash
 * @returns - transaction detail
 */
const transaction = btcGetTransaction('97bc1c3c23b179cba837e4060c0d07aa399f7ac7d34d91a7405cb5f801b93c8a');
```
```REST API call
curl --request GET \
  --url https://api-eu1.tatum.io/v3/blockchain/bitcoin/transaction/{hash} \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: '
```
</div>

**Response:**
<div class='tabbed-code-blocks'>
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
</div>

Let's take a look at this blockchain transaction. You can see that the transaction consumed two deposit transactions - vin array  - these are the two blockchain transactions credited to the account. As an output of the transaction - vout array - there are also two recipients. The first one is the recipient address you entered in the request. The second one is the address from your blockchain wallet with index 0. By default, Tatum uses address 0 of the blockchain wallet as an internal system address, where all the leftovers from the transactions are being acquired. They are used again in the next transaction.

<div class="toolbar-note">
In blockchain transactions, unspent deposits from every blockchain address in the same blockchain wallet are automatically collected at the address 0. This is necessary to fully utilize Tatum's off-chain features.
</div>

The logic is the same for the [Litecoin](https://docs.tatum.io/rest/virtual-accounts/send-litecoin-from-tatum-account-to-address) and [Bitcoin Cash](https://docs.tatum.io/rest/virtual-accounts/send-bitcoin-cash-from-tatum-account-to-address) blockchains. Ethereum is based on different principles and will be described in a separate article.