# How to set up virtual accounts with XRP, BNB, and XLM

Create multiple virtual accounts from the same xpub and send assets between them instantly and feelessly.

---


Tatum Virtual Accounts offer unique capabilities to send cryptocurrencies between accounts instantly and without incurring blockchain fees. For each type of blockchain, virtual accounts must be set up in a specific way so that they will integrate and function seamlessly with your app's backend. In this guide, you will learn how to set up virtual accounts to work with the following blockchains:

- Ripple (XRP)
- Binance Chain (BNB)
- Stellar (XLM)

---

## Generate a wallet

In this guide, we will be using XRP to demonstrate how to set up virtual accounts. However, the exact same process applies to BNB and XLM.

<!-- theme: warning -->
>All of the virtual accounts for XRP, BNB, or XLM within your app must be connected to the same wallet address on each respective blockchain. If virtual accounts are connected to deposit addresses generated from different addresses from different wallets, they will not function properly.

Use the following API endpoint to [generate an XRP wallet](https://tatum.io/apidoc.php#operation/XrpWallet):

**Request example**

```JavaScript
import { generateXrpWallet } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const generateWallet = () => {
  const wallet = generateXrpWallet();
  console.log(wallet);
};

generateWallet();
```
```cURL
curl --request GET \
  --url https://api-eu1.tatum.io/v3/xrp/account \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```
```KMS
tatum-kms storemanagedwallet XRP --testnet
```

The response will contain your wallet's xpub in the "address" field and private key in the "secret" field.

**Response example**

```json
{
  address: 'rDWpnpN3KHUz49WZJpwNe2GQLHeSZ62B1d',
  secret: 'ssKPqjZ9VPg79rEsrHjydRLMM8xgG'
}
```

---

## Create virtual accounts


Now, you can generate XRP virtual accounts connected to the xpub of the wallet you have just generated. You can generate as many virtual accounts as you want, but they must all be connected to the same xpub in order to function properly.

Use the following API endpoint to [generate XRP virtual accounts](https://tatum.io/apidoc.php#operation/createAccount). In the "xpub" field, enter the xpub of the wallet you generated in the previous step.

**Request example**

```JavaScript
import { Account, Currency, CreateAccount, createAccount } from "@tatumio/tatum";
import { config } from "dotenv";

config();

const createNewAccount = async () => {
  const createAccountData: CreateAccount = {
    currency: Currency.XRP,
    xpub: wallet.address
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
      "currency":"XRP",
      "xpub":"rDWpnpN3KHUz49WZJpwNe2GQLHeSZ62B1d",
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

The response will contain details about your newly created virtual account.

**Response example**
```json
{
  currency: 'XRP',
  active: true,
  balance: { accountBalance: '0', availableBalance: '0' },
  frozen: false,
  xpub: 'rDWpnpN3KHUz49WZJpwNe2GQLHeSZ62B1d',
  accountingCurrency: 'USD',
  id: '61afba85997b887f543fa7f0'
}
```

---

## Create deposit addresses

Finally, we create deposit addresses for each virtual account. Again, these deposit addresses MUST be generated from the xpub of the wallet we generated in the first step. If they are not generated from the same xpub, they will not function properly.

<!-- theme: info-->
>On XRP and XLM, you must freeze a specific amount of assets for each deposit address you create.  If you are creating 1,000,000 deposit addresses, this will cost you a ridiculous amount of money. For this reason, every virtual account uses the same deposit address, which is the same address as the wallet's xpub. 

Use the following API endpoint to [generate deposit addresses for each virtual account](https://tatum.io/apidoc.php#operation/generateDepositAddress):

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
curl --request POST \
  --url 'https://api-eu1.tatum.io/v3/offchain/account/{id}/address?index=1' \
  --header 'x-api-key: REPLACE_KEY_VALUE'
```
The response will contain your newly generated `address` which is connected to the virtual account.

**Response example**

```json
{
  xpub: 'rDWpnpN3KHUz49WZJpwNe2GQLHeSZ62B1d',
  derivationKey: 1,
  address: 'rDWpnpN3KHUz49WZJpwNe2GQLHeSZ62B1d',
  currency: 'XRP',
  destinationTag: 1
}
```
<!-- theme: info-->
>When users receive deposits from external sources to the deposit address, the user is identified by unique identifiers specific to XRP, BNB, and XLM.
>
>**The fields used to identify users for each blockchain are as follows:**
>
>- XRP - `destinationTag`
>
>- BNB - `memo`
>
>- XLM - `message`

<!-- theme: warning-->
>If you try to create a virtual account with a different xpub, or try to create a deposit address for a virtual account with a different xpub, you will encounter the following error:
```json
{
  errorCode: 'account.xpub.bnb',
  message: 'For BNB, XLM and XRP, only 1 xpub is allowed for API ?Key. For your ledger account, use already defined xpub and memo field for address differentiation. Please contact support at support@tatum.io if you need more.'
}
```


This is not a bug, it is the expected behavior, and is telling you that you must use the same xpub to create different virtual accounts and deposit addresses on the Ripple blockchain.

---

# And that's it!

Now your XRP virtual accounts should work perfectly and your users can send feeless, instant transactions to each other. All of your assets are in one blockchain address and you can send as many transactions as you want without having to worry about which blockchain address holds how much crypto or how much transaction fees will cost to send funds between accounts. 

