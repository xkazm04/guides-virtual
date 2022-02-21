# How to set up virtual accounts with ETH, TRON, CELO, MATIC, BSC, and ONE

Learn two different ways to set up virtual accounts with account-based blockchains.

---

Tatum Virtual Accounts offer unique capabilities to send cryptocurrencies between accounts instantly and without incurring blockchain fees. For each type of blockchain, virtual accounts must be set up in a specific way so that they will integrate and function seamlessly with your app's backend. 

In this guide, you will learn how to set up virtual accounts to work with the account-based blockchains we support: 

- Ethereum (ETH)
- Tron (TRON)
- Celo (CELO)
- Polygon (MATIC)
- Harmony (ONE)
- Binance Smart Chain (BSC)
- XinFin (XDC)

<!-- theme: info-->
>With other types of blockchains supported in Tatum, you must associate virtual accounts with addresses generated from a single wallet with the same extended public key (xpub) in order for them to work properly. 
>
>However, with the account-based blockchains we will be working with in this guide, you can associate virtual accounts with deposit addresses generated from different wallets with different xpubs. 

This guide will be divided into two main sections:

1. **First, we will show you how to create virtual accounts, blockchain addresses, and how to associate existing blockchain addresses with virtual accounts.**
2. **Then, we will show you two approaches to how to set up your application to work with virtual accounts.**

---

# Creating and connecting virtual accounts with blockchain addresses

First, we will show you how to generate a wallet and create virtual accounts and blockchain addresses from the wallet's xpub. Alternatively, you can associate existing blockchain addresses with the virtual accounts you create. In this case, you can skip the "**Generate a wallet**" and "**Create deposit addresses**" steps below.

<!-- theme: info-->
>The code snippets in this guide feature API endpoints for the Polygon blockchain, but the same process applies to all of the other blockchains listed above.

Use the following API endpoint to generate a wallet on Polygon:

**Request example**

```JavaScript
// You need to install the Javascript library
// https://github.com/tatumio/tatum-js
const {generateWallet, Currency} = require("@tatumio/tatum");

const maticWallet = generateWallet(Currency.MATIC, false);
console.log(maticWallet);
```
```cURL
curl --request GET \
  --url 'https://api-eu1.tatum.io/v3/polygon/wallet?mnemonic=SOME_STRING_VALUE' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```
```KMS
tatum-kms storemanagedwallet MATIC --testnet
```

The response will be the wallet's xpub and mnemonic.

**Response example**
```json
{
  "mnemonic": "urge pulp usage sister evidence arrest palm math please chief egg abuse",
  "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid"
}
```

## Create virtual accounts

Next, you can [create virtual accounts](https://tatum.io/apidoc.php#operation/createAccount) from the xpub of the wallet.

**Request example**
```JavaScript
import { Account, Currency, CreateAccount, createAccount } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const createNewAccount = async () => {
  const createAccountData: CreateAccount = {
    currency: Currency.MATIC,
    xpub: wallet.xpub
  };
  const accoun: Account = await createAccount(createAccountData);
  console.log(accoun);
};

createNewAccount();
```
```cURL
curl --request POST \
  --url https://api-eu1.tatum.io/v3/ledger/account \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
      "currency":"MATIC",
      "xpub":"xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid"      
      }'
```

The response will contain details about your newly created account, including the account `id`, which is be used to identify the account within Tatum.

**Response example**

```json
{
  currency: 'MATIC',
  active: true,
  balance: { accountBalance: '0', availableBalance: '0' },
  frozen: false,
  xpub: 'xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid',
  accountingCurrency: 'USD',
  id: '61afba85997b887f543fa7f0'
}
```

---

## Create blockchain addresses

Now, you can [create blockchain addresses](https://tatum.io/apidoc.php#operation/generateDepositAddress) for each virtual account.

**Request example**

```JavaScript
import { generateDepositAddress, Address } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const createNewAddress = async () => {
    const id = account.id;
    const address: Address = await generateDepositAddress(id);
    console.log(address);
};

createNewAddress();
```
```cURL
curl --request POST \
  --url 'https://api-eu1.tatum.io/v3/offchain/account/{id}/address' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```

The response will contain the blockchain address you have just generated.

**Response example**

```json
{
  address: '0x6d7abd9f164db455e5aabfb5a53307e8d5657555',
  currency: 'MATIC',
  derivationKey: 0,
  xpub: 'xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid'
}
```

## Connect virtual accounts to existing blockchain addresses

Alternatively, you can connect the virtual accounts you created above to existing blockchain addresses. To do so, use the [assign address to account](https://tatum.io/apidoc.php#operation/assignAddress) endpoint, and enter the account ID of a virtual account and the blockchain address you'd like to connect it to:

**Request example**
```JavaScript
import { assignDepositAddress, Address } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const assignAddress = async (address) => {
    const id = account.id;
    const address: Address = await assignDepositAddress(id, address);
    console.log(address);
};

assignAddress();
```
```cURL
curl --request POST \
  --url 'https://api-eu1.tatum.io/v3/offchain/account/{id}/address/{address}' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```

The response will contain the address that has been assigned to the virtual account.

**Response example**

```json
{
  address: '0x6d7abd9f164db455e5aabfb5a53307e8d5657555',
  currency: 'MATIC'
}
```
---
# Two ways to work with virtual accounts in your application

There are two primary approaches for how to work with virtual accounts on account-based blockchains:

1. Use each blockchain address associated with a virtual account as **both a deposit and withdrawal address**.
2. Use each blockchain address associated with a virtual account as a deposit address, but send all deposited funds to a **master blockchain address for withdrawals**.

Each approach has its own benefits, and we will show you how to set both of them up.

## Use each blockchain address as both a deposit and withdrawal address

<!-- theme: info-->
>This approach is the most cost-effective, but requires that you keep track of which funds are at which address associated with which private key. Thus, there is significantly more logistical work associated with this approach.

In this approach, your users receive incoming transactions from external sources to the deposit addresses associated with each virtual account. The assets remain at these deposit addresses until they are withdrawn to another blockchain address.

Here is an example of the workflow:

1. **User A** receives 3 MATIC from an external source to **deposit address A**, which is connected to **virtual account A**. 
2. The balance of **virtual account A** is updated to reflect the incoming transaction, and now contains 3 MATIC.
3. **User A** sends these 3 MATIC to **user B**  at **virtual account B**.
4. The balance of **virtual account A** is now 0, and the balance of **virtual account B** is 3 MATIC. However, **deposit address A** still holds the 3 MATIC on the blockchain.
5. When **user B** would like to withdraw these 3 MATIC to **deposit address B**, which is connected to **virtual account B**, and send them outside the application, they will be withdrawing them from **deposit address A**. 
6. As a result, they will need to use the `privateKey` of **deposit address A** to withdraw the assets.

This type of workflow is the most cost-effective because the only blockchain transactions you are paying gas fees for are for when users withdraw funds to their deposit addresses from their virtual accounts.

However, you, as the application owner, must keep track of which private key is associated with which assets at all times to allow your users to withdraw assets to their deposit addresses. This can be a lot of work to keep track of everything, which is why the second workflow, outlined below, might be more beneficial for some people.

## Use a master address for withdrawals

<!-- theme: info-->

>This approach incurs more blockchain fees, as each time a user receives assets to their deposit address, they are transferred to a master address which holds all funds to be withdrawn. 
>
>However, since there is only one address from which assets are withdrawn, you don't need to keep track of which address holds which assets, and the workflow is greatly simplified in comparison to the previous approach.

### Create a master address for all withdrawals

In this approach, you should create a deposit address on the blockchain to which you will send every incoming transaction received to every deposit address. Use the following API endpoint to [generate a deposit address on Polygon](https://tatum.io/apidoc.php#operation/PolygonGenerateAddress):

**Request example**

```JavaScript
const {generateAddressFromXPub, generateWallet, Currency} = require("@tatumio/tatum");

const wallet = generateWallet(Currency.MATIC, false);

const maticWallet = generateAddressFromXPub(Currency.MATIC, false, wallet.xpub, 0);
console.log(maticWallet);
```
```cURL
curl --request GET \
  --url https://api-eu1.tatum.io/v3/polygon/address/{xpub}/{index} \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```

The response will contain the address you have just generated.

**Response example**

```json
{
  "address": "0xa7673161CbfE0116A4De9E341f8465940c2211d4"
}
```

### Example workflow

Now that you've created a master address from which your users can withdraw assets to their deposit addresses to send them outside of your application, let's take a look at an example of how this workflow would look in practice.

1. **User A** receives 3 MATIC to **deposit address A** connected with **virtual account A.**
2. The balance of **virtual account A** is now 3 MATIC, and these 3 MATIC are transferred to the **master blockchain address**. The balance of **virtual account A** remains 3 MATIC.
3. **User A** transfers 3 MATIC from **virtual account A** to **user B** at **virtual account B**.
4. The balance of **virtual account A** is now 0 MATIC, and the balance of **virtual account B** is 3 MATIC.
5. Now, when **user B** withdraws the 3 MATIC from **virtual account B** to their own **deposit address B**, the assets are withdrawn from the **master blockchain address**. 

There is no need to keep track of which address holds which assets, as all incoming assets to all deposit addresses are immediately transferred to the master blockchain address. You don't need to figure out which private key to use for withdrawals, because all withdrawals will come from the master blockchain address and use the private key for this address. 

Although this will incur more blockchain transaction fees, as you are immediately sending all deposits to a master address, it significantly simplifies your application's workflow, and can save a lot of time and headache in the future. 

---

# And that's it!

So there you have it. Two approaches for how to set up your application to work with virtual accounts, each with their own benefits. Now it's up to you to decide which approach will work best for your purposes. We hope this has been helpful, and we hope you enjoy all of the flexibility and features virtual accounts will open up for you!