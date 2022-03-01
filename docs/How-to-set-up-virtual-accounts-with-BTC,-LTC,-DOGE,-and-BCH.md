# How to set up virtual accounts with BTC, LTC, DOGE, and BCH

Create multiple virtual accounts from the same xpub for UTXO blockchains to allow your users to send instant, feeless transactions.

---

Tatum Virtual Accounts offer unique capabilities to send cryptocurrencies between accounts instantly and without incurring blockchain fees. For each type of blockchain, virtual accounts must be set up in a specific way so that they will integrate and function seamlessly with your app's backend. 

In this guide, you will learn how to set up virtual accounts to work with the UTXO blockchains we support: 

- Bitcoin (BTC)
- LiteCoin (LTC)
- Doge (DOGE)
- Bitcoin Cash (BCH)

---

## Generate a wallet

<div class="toolbar-warning">
In this guide, we will generate just one blockchain wallet for all of our virtual accounts. All of the virtual accounts within your app must be connected to the same wallet. If virtual accounts are connected to deposit addresses generated from different extended public keys (xpubs) from different wallets, they will not function properly.
</div>

In this guide, we will use examples for the Bitcoin blockchain, but the process is the same for all of the other blockchains listed above.

To [generate a Bitcoin wallet](https://docs.tatum.io/rest/blockchain/generate-bitcoin-wallet), use the following API endpoint:

**Request example**
<div class='tabbed-code-blocks'>
```SDK
// You need to install the Javascript library
// https://github.com/tatumio/tatum-js
const {generateWallet, Currency} = require("@tatumio/tatum");
const btcWallet = generateWallet(Currency.BTC, false);
console.log(btcWallet);
```
```REST API call
curl --request GET \
  --url 'https://api-eu1.tatum.io/v3/bitcoin/wallet?mnemonic=SOME_STRING_VALUE' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```
```KMS
tatum-kms storemanagedwallet BTC --testnet
```
</div>

The response will contain the wallet's mnemonic and xpub.

**Response:**

```json
{
  mnemonic: 'north today grit rate brand atom swift act surround stool lumber neither salon solution labor buyer celery coach clarify poet clown next culture melt',
  xpub: 'tpubDEvtqDzBrGVtGJ3yvSV8khBNFa5FVTFu25YQTKfDQUvSp2zFcQzy1dPgKXKHBzKgygHeefpp4ASmJCi5x9bjiJzbJ9XQczd76NgrS1tzZY1'
}
```

---

## Create virtual accounts

Now, you can generate BTC virtual accounts connected to the xpub of the wallet you have just generated. You can generate as many virtual accounts as you want, but they must all be connected to the same xpub in order to function properly.

<div class="toolbar-note">
The reason virtual accounts for UTXO blockchains must be generated from the same xpub, is that UTXO blockchains are capable of performing multiple transactions at once. If the virtual accounts were generated from different xpubs, it would be impossible to keep track of which assets were being sent from which wallet.
</div>

Use the following API endpoint to [generate BTC virtual accounts](https://docs.tatum.io/rest/virtual-accounts/create-new-account). In the "xpub" field, enter the xpub of the wallet you generated in the previous step.

**Request example**
<div class='tabbed-code-blocks'>
```SDK
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
```REST API call
curl --request POST \
  --url https://api-eu1.tatum.io/v3/ledger/account \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
      "currency":"BTC",
"xpub":"tpubDEvtqDzBrGVtGJ3yvSV8khBNFa5FVTFu25YQTKfDQUvSp2zFcQzy1dPgKXKHBzKgygHeefpp4ASmJCi5x9bjiJzbJ9XQczd76NgrS1tzZY1",
      "customer":{
          "accountingCurrency":"USD",
          "customerCountry":"US",
          "externalId":"123654",
          "providerCountry":"US"
          },
      "compliant":false,
      "accountCode":"AC_1011_B",
      "accountingCurrency":"USD",
      "accountNumber":"123456"
      }'
```
</div>

**Response:**

```json
{
  currency: 'BTC',
  active: true,
  balance: { accountBalance: '0', availableBalance: '0' },
  frozen: false,
  xpub: 'tpubDEvtqDzBrGVtGJ3yvSV8khBNFa5FVTFu25YQTKfDQUvSp2zFcQzy1dPgKXKHBzKgygHeefpp4ASmJCi5x9bjiJzbJ9XQczd76NgrS1tzZY1',
  accountingCurrency: 'USD',
  id: '61afba85997b887f543fa7f0'
}
```

---

## Create deposit addresses

Finally, we create deposit addresses for each virtual account. Again, these deposit addresses MUST be generated from the xpub of the wallet we generated in the first step. If they are not generated from the same xpub, they will not function properly.

Use the following API endpoint to [generate deposit addresses for each virtual account](https://docs.tatum.io/rest/virtual-accounts/create-new-deposit-address):

**Request example**
<div class='tabbed-code-blocks'>
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
curl --request POST \
  --url 'https://api-eu1.tatum.io/v3/offchain/account/{id}/address?index=1' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```
</div>

**Response example**
```json
{
  address: 'mfX9GBTff8mBBskwFPfGjyAGiTi3eS61aN',
  currency: 'BTC',
  derivationKey: 0,
  xpub: 'tpubDEvtqDzBrGVtGJ3yvSV8khBNFa5FVTFu25YQTKfDQUvSp2zFcQzy1dPgKXKHBzKgygHeefpp4ASmJCi5x9bjiJzbJ9XQczd76NgrS1tzZY1'
}
```

<div class="toolbar-note">
If you want to generate virtual accounts from different xpubs, you won't be able to perform instant transactions between them. Only virtual accounts generated from the same xpub will be able to perform transactions between one another.
</div>

---

## And that's it! 

Now your virtual accounts should work perfectly and your users can send feeless, instant transactions to each other.



