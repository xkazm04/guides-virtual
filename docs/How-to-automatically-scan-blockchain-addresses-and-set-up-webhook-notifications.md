# How to automatically scan blockchain addresses and set up webhook notifications


Create virtual accounts that update their balances to reflect incoming transactions at blockchain addresses and set up webhook notifications for a variety of blockchain events.

---

## Automatic blockchain address scanning

Tatum's virtual accounts enable a variety of features otherwise unavailable on most blockchains. Their most crucial feature, and the one that opens the doors for all of the rest, is automatic scanning of blockchain addresses for incoming transactions.

Normally, to get the balances of blockchain addresses, you'd have to query each address every time you want to see if any transactions have been received. However, when you connect a virtual account to a blockchain address (or multiple addresses), it automatically detects incoming transactions at every connected address, and updates its balance accordingly.

This enables the other powerful features built-in to Tatum's virtual accounts, including...

## Webhook notifications

If you’re building an exchange, marketplace, payment app, or any other kind of fintech solution, you’ll definitely want to be able to notify users about various types of blockchain events. Nobody wants to have to hit “refresh” in their PayPal account to know if they’ve received funds, and your app’s end-users shouldn’t have to either.

Blockchains do not natively support webhook notifications. However, since virtual accounts automatically scan blockchain addresses for incoming transactions, they are the perfect tool for setting up webhook notifications within your app.

<!-- theme: info -->
>In this guide, you'll learn to how to:
>1. Create a virtual account
>2. Create a blockchain address connected to the account
>3. View all addresses connected to a virtual account
>4. Set up webhook notifications for the account
>5. Test your webhook notifications on testnet

---

# How to automatically scan for incoming transactions

## Create a virtual account

First we will create a virtual account to which we will connect a blockchain address (or addresses). To [create a virtual account](https://tatum.io/apidoc#operation/createAccount), we must enter the xpub from a blockchain wallet into the body of the API endpoint. This is the first connection between the blockchain and the virtual account.

To create a BTC blockchain wallet, please refer to [this guide](https://docs.tatum.io/guides/blockchain/how-to-create-a-blockchain-wallet). 

To create wallets on any other supported blockchain, please refer to the blockchain's respective section in our [API documentation](https://tatum.io/apidoc.php).

<!-- theme: warning -->
>Every virtual account is of a specific currency, which must be defined when it is created. This currency cannot be changed in the future (i.e. a BTC virtual account can only be connected to BTC addresses and will always be a BTC account).


We are going to set up a virtual account on the Bitcoin blockchain. For more information on how to set up guides on [UTXO blockchains](https://docs.tatum.io/guides/ledger-and-off-chain/how-to-set-up-virtual-accounts-with-btc-ltc-doge-and-bch), [blockchains like XRP, BNB, and XLM](https://docs.tatum.io/guides/ledger-and-off-chain/how-to-set-up-virtual-accounts-with-xrp-bnb-and-xlm), and [account-based blockchains](https://docs.tatum.io/guides/ledger-and-off-chain/how-to-set-up-virtual-accounts-with-eth-tron-celo-matic-bsc-and-one), please refer to the respective guides.

Use the following API endpoint to generate a virtual account from the xpub of a Bitcoin wallet.

**Request example**
```JavaScript
import { Account, Currency, CreateAccount, createAccount } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const createNewAccount = async () => {
  const createAccountData: CreateAccount = {
    currency: Currency.BTC,
    xpub: wallet.address
  };
  const accoun: Account = await createAccount(createAccountData);
  console.log(accoun);
};

createNewAccount();
```
```cURL
curl --location --request POST 'https://api-eu1.tatum.io/v3/ledger/account' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "currency": "BTC",
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb"
}'
```
The response contains information about your newly generated virtual account.

**Response example**
```json
{
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0",
        "availableBalance": "0"
    },
    "frozen": false,
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "accountingCurrency": "EUR",
    "id": "5fb7bdf6e96d9ab593e191a5"
}
```

<!-- theme: info -->
>Virtual accounts can also be assigned to customers in Tatum. A customer inside Tatum is an entity containing information about a specific user of your application, such as their country of residence, accounting currency, etc. 
To create a customer, you simply need to include an `externalId` field when creating a virtual account. This is an identifier of the user in your external system. More details are available in the [API Reference](https://tatum.io/apidoc.php).

## Generate a blockchain address and connect it to the virtual account

Now that you've created a virtual account, you must synchronize it with the blockchain. Until you do, there is no blockchain address connected to the account, only a blockchain wallet from which addresses will be chosen. 

Use the [create new deposit address](https://tatum.io/apidoc#operation/generateDepositAddress) endpoint to generate an address for the account.

**Request example**
```JavaScript
import { generateDepositAddress, Address } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const createNewDepositAddress = async () => {
    const id = account.id;
    const address: Address = await generateDepositAddress(id);
    console.log(address);
};

createNewDepositAddress();
```
```cURL
curl --location --request POST 'https://api-eu1.tatum.io/v3/offchain/account/5fb7bdf6e96d9ab593e191a5/address' \
--header 'x-api-key: YOUR_API_KEY'
```

The response will contain the blockchain address you have just generated.

**Response example**
```json
{
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "derivationKey": 1,
    "address": "mgSXLa5sJHvBpYTKZ62aW9z2YWQNTJ59Zm",
    "currency": "BTC"
}
```

Any incoming blockchain transaction to this address will be automatically synchronized to the virtual account.

## View a list of blockchain addresses connected to an account

It is possible to work with only one blockchain address connected to the account, but you can also connect as many addresses as you want. The virtual account's balance will reflect the sum of all connected blockchain addresses.

Your app's users will most likely want to be able to see all of the addresses connected to their account. You can view all a list of them using the off-chain method [Get all deposit addresses](https://tatum.io/apidoc#operation/getAllDepositAddresses).

**Request example**

```JavaScript
import { getDepositAddressesForAccount } from '@tatumio/tatum';
/**
 * Get all deposit addresses generated for account.
 * @param id - account ID
 * @returns addresses detail - - 
 https://tatum.io/apidoc#operation/getAllDepositAddresses
 */
const addresses = getDepositAddressesForAccount ("5fb7bdf6e96d9ab593e191a5");
```
```cURL
curl --location --request GET 'https://api-eu1.tatum.io/v3/offchain/account/5fb7bdf6e96d9ab593e191a5/address' \
--header 'x-api-key: YOUR_API_KEY'
```

The response will contain a list of all deposit addresses connected to the virtual account.

**Response example**

```json
[
    {
        "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
        "derivationKey": 1,
        "address": "mgSXLa5sJHvBpYTKZ62aW9z2YWQNTJ59Zm",
        "currency": "BTC"
    }
]
```

---

# How to set up webhook notifications

Now that we've created a virtual account and connected a blockchain address to it, we can set up webhook notifications for the address with just one more API call.

In Tatum virtual accounts, webhook notifications work as **subscriptions** to the accounts. When you set up a subscription, the account specified in the `id` field will be monitored for the blockchain event specified in the `type` field.

To set up webhook notifications for incoming blockchain transactions, use the following API call:

**Request example**

```JavaScript
import { createNewSubscription, CreateSubscription, SubscriptionType } from '@tatumio/tatum'

export const createAccountIncomingBlockchainTransactionSubscription = async () => {
  const data: CreateSubscription = {
    "type": SubscriptionType.ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION,
    "attr": {
      "id": "61eed3362a90644ef54a5018",
      "url": "https://webhook.site/"
    }
  }
  return await createNewSubscription(data)
}
```
```cURL
curl --request POST \
  --url https://api-eu1.tatum.io/v3/subscription \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
     "type":"ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION",
     "attr":{
        "id":"5e6be8e9e6aa436299950c41",
        "url":"https://webhook.tatum.io/account"
        }
     }'
```

The required parameters are for an incoming transactions subscription are:

- `type` - the type of subscription you are creating
- `id` - the ID of the virtual account or customer for which you are creating the subscription
- `url` - the URL of the webhook listener you are using

**Types of subscriptions:**

- Incoming transactions: `ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION`
- Pending transactions: `ACCOUNT_PENDING_BLOCKCHAIN_TRANSACTION`
- Customer trade matches (in exchanges): `CUSTOMER_TRADE_MATCH`, `CUSTOMER_PARTIAL_TRADE_MATCH`
- Transactions in specific blocks (outgoing): `TRANSACTION_IN_THE_BLOCK`
- Tatum KMS completed/failed transactions: `KMS_COMPLETED_TX` , `KMS_FAILED_TX`
- Outgoing transactions in the whole account: `OFFCHAIN_WITHDRAWAL`
- Account balances above a certain limit: `ACCOUNT_BALANCE_LIMIT`
- Transaction history reports: `TRANSACTION_HISTORY_REPORT`

<!-- theme: info -->

>Different types of subscriptions have different required parameters. For a full list of each type with example API endpoint bodies, please refer to our [API documentation](https://tatum.io/apidoc.php).

---

# How to test your webhook notifications

To help you get the hang of how webhooks work with virtual accounts, we'd suggest giving it a try on a blockchain testnet (as you should with anything in blockchain development).

Once you've followed the above steps, you can test out your webhook notifications as follows:

1. Send some crypto from a testnet faucet on your blockchain of choice to the blockchain address connected to your virtual account (e.g. 0.001 BTC).
2. Check the blockchain address (either in the blockchain's explorer or using a **get account balance** endpoint in Tatum) to confirm that the transaction has arrived. This could take a little bit of time, so we'd suggest waiting a few minutes, then trying again in another 5 minutes if it hasn't arrived yet.
3. Check your virtual account's balance using [this endpoint](https://tatum.io/apidoc.php#operation/getAccountBalance).
4. Now, check your webhook listener (the URL you entered when creating the subscription) to confirm that the webhook has correctly been sent.

# And that's it!

You've successfully created a virtual account, connected a blockchain address to it, and set up webhook notifications. Now, anytime you send a funds to the blockchain address, it will be synced and visible in the account, and you will receive a webhook notification about the transaction.
