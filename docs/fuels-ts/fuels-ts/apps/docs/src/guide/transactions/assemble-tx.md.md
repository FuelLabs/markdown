# AssembleTx

The `assembleTx` method is a crucial part of the Fuel TypeScript SDK that helps prepare and assemble transactions with the correct inputs, outputs, and policies. It is used by all higher-level APIs in the SDK, including account transfers, contract and blob deployments, and contract calls. This guide provides a comprehensive overview of how to use `assembleTx` effectively.

## Overview

The `assembleTx` method takes a transaction request and assembles it with the necessary inputs, outputs, and policies based on the provided parameters. It handles:

- Coin quantity for different assets
- Fee payer account
- Gas and fee estimation
- Predicate estimation
- Resource exclusion (specific resources to ignore)

## Parameters

The [AssembleTxParams](DOCS_API_URL/types/_fuel_ts_account.AssembleTxParams.html) interface includes the following parameters:

```
export type AssembleTxParams<T extends TransactionRequest = TransactionRequest> = {
  // The transaction request to assemble
  request: T;
  // Coin quantities required for the transaction, optional if transaction only needs funds for the fee
  accountCoinQuantities?: AccountCoinQuantity[];
  // Account that will pay for the transaction fees
  feePayerAccount: Account;
  // Block horizon for gas price estimation (default: 10)
  blockHorizon?: number;
  // Whether to estimate predicates (default: true)
  estimatePredicates?: boolean;
  // Resources to be ignored when funding the transaction (optional)
  resourcesIdsToIgnore?: ResourcesIdsToIgnore;
  // Amount of gas to reserve (optional)
  reserveGas?: BigNumberish;
};

export type AssembleTxResponse<T extends TransactionRequest = TransactionRequest> = {
  assembledRequest: T;
  gasPrice: BN;
  receipts: TransactionResultReceipt[];
  rawReceipts: TransactionReceiptJson[];
};
```

### Parameter Details

- `request`: The transaction request to be assembled.
- `blockHorizon`: The number of blocks to look ahead for gas price estimation. Defaults to `10` blocks.
- `feePayerAccount`: The account that will pay for the transaction fees
- `accountCoinQuantities`: An array of coin quantities needed for the transaction. This is optional if the transaction only requires funds to cover the fee. The parameters are:
  - `amount`: The amount of coins needed (fee value does need to be included)
  - `assetId`: The asset ID of the coins
  - `account`: The account providing the coins (optional, defaults to `feePayerAccount`)
  - `changeOutputAccount`: The account to receive change (optional, defaults to the `account` or `feePayerAccount` properties, respectively)
- `resourcesIdsToIgnore`: Resources to be ignored when funding the transaction (UTXOs or messages)
- `estimatePredicates`: Whether to estimate gas for predicates
- `reserveGas`: Additional Amount of gas to be set for the transaction

### Default Behaviors for Account Properties

The `accountCoinQuantities` entries have two optional properties with specific default behaviors:

1. `account` property:

   - If not provided, defaults to the root `feePayerAccount`
   - Used to specify which account provides the resources for the transaction

2. `changeOutputAccount` property:
   - If not provided, defaults to the `account` property
   - Used to specify which account receives the change from all spent resources from a specific `assetId`

Example of default behaviors:

```
import { Provider, Wallet, type AccountCoinQuantity } from 'fuels';
import { TestAssetId } from 'fuels/test-utils';

import {
  LOCAL_NETWORK_URL,
  WALLET_PVT_KEY,
  WALLET_PVT_KEY_2,
  WALLET_PVT_KEY_3,
} from '../../../../env';

// #region transaction-request-3
const provider = new Provider(LOCAL_NETWORK_URL);
const accountA = Wallet.fromPrivateKey(WALLET_PVT_KEY, provider);
const accountB = Wallet.fromPrivateKey(WALLET_PVT_KEY_2, provider);
const accountC = Wallet.fromPrivateKey(WALLET_PVT_KEY_3, provider);
const baseAssetId = await provider.getBaseAssetId();
const accountCoinQuantities: AccountCoinQuantity[] = [
  {
    amount: 100,
    assetId: baseAssetId,
    // account defaults to feePayerAccount
    // changeOutputAccount defaults to feePayerAccount
  },
  {
    amount: 200,
    assetId: TestAssetId.A.value,
    account: accountA,
    // changeOutputAccount defaults to accountA
  },
  {
    amount: 300,
    assetId: TestAssetId.B.value,
    account: accountB,
    changeOutputAccount: accountC,
    // Both account and changeOutputAccount are explicitly set
  },
];
```

## Return Value

The method returns an object of the type [AssembleTxResponse](DOCS_API_URL/types/_fuel_ts_account.AssembleTxResponse.html):

- `assembledRequest`: The fully assembled transaction request with all necessary inputs, outputs, and policies
- `gasPrice`: The estimated gas price for the transaction
- `receipts`: Parsed receipts returned from the transaction dry run.
- `rawReceipts`: Unparsed receipts returned from the transaction dry run.

## Usage Example

```
const request = new ScriptTransactionRequest();

request.addCoinOutput(accountB.address, transferAmount, baseAssetId);

const accountCoinQuantities: AccountCoinQuantity[] = [
  {
    amount: transferAmount,
    assetId: baseAssetId, // Asset ID
    account: accountA,
    changeOutputAccount: accountA, // Optional
  },
];

// Assemble the transaction
const { assembledRequest, gasPrice, receipts } = await provider.assembleTx({
  request,
  accountCoinQuantities,
  feePayerAccount: accountA,
  blockHorizon: 10,
  estimatePredicates: true,
});

// The assembledRequest is now ready to be signed and sent
const submit = await accountA.sendTransaction(assembledRequest);
await submit.waitForResult();
```

## Attention to `account` and `changeOutputAccount` Fields

As mentioned earlier, the `account` and `changeOutputAccount` fields are optional in the `accountCoinQuantities` entries. However, **special attention must be paid to** `changeOutputAccount` due to how change from spent resources works on Fuel.

In Fuel, only one `OutputChange` is allowed per `assetId` in a transaction. But what exactly is an `OutputChange`? In simple terms, it designates the recipient of the leftover funds ("change") after all UTXO resources for a specific `assetId` have been spent.

### Understanding Change in Fuel's UTXO Model

Because Fuel uses a **UTXO-based model** (unlike Ethereum’s account-based model), transactions will spend all included UTXOs, even if only a small portion is actually required. For example, suppose you have a single UTXO worth 10 ETH. If you create a transaction to send just 1 Gwei, the entire 10 ETH UTXO will be consumed. The transaction will then:

- Create a UTXO with 1 Gwei to the recipient.
- Create another UTXO for the remaining amount (i.e., 10 ETH - 1 Gwei - fee), sent to the address defined in the `OutputChange`.

In this context, the `OutputChange` ensures that **you receive the change** from your transaction.

### Fuel's Constraint: One Change Output per `assetId`

A key rule in Fuel is that only one `OutputChange` per `assetId` is allowed per transaction. This becomes particularly important when a transaction includes resources from **multiple accounts** that hold the same `assetId` (e.g., ETH). In such a case, **only one account can receive the change**, whichever one is defined on the `OutputChange`.

If not managed correctly, this can lead to unintended behavior, such as one account spending its funds while another receives the leftover balance.

### How This Relates to `assembleTx`

The `changeOutputAccount` field in `assembleTx` explicitly specifies **which account should receive the change** for a particular `assetId`. This is critical when you're using resources from multiple accounts in the same transaction.

Here's an example:

```
let { assembledRequest } = await provider.assembleTx({
  request,
  feePayerAccount: accountA,
  accountCoinQuantities: [
    {
      amount: transferAmount,
      assetId: baseAssetId,
      account: accountB,
      /**
       * accountB will receive the change. Although it is explicitly set here,
       * if it were not set, it would default to the account property,
       * which in this case is also accountB.
       */
      changeOutputAccount: accountB,
    },
  ],
});
```

In the example above:

- Resources from `accountB` are being explicitly requested in `accountCoinQuantities`.
- `accountA` is listed as the `feePayerAccount`, meaning it will also contribute resources to cover fees.
- `changeOutputAccount` is explicitly set to `accountB`.

As a result, `accountB` will receive the change, even if UTXOs from `accountA` are spent in the transaction.

This means that if `accountA` contributes a 10 ETH UTXO, the transaction will spend the full 10 ETH. The leftover amount (change) will be sent to `accountB`, not back to `accountA`, because `changeOutputAccount`is set to `accountB`.

## Best Practices

1. Always provide the correct `feePayerAccount` that has sufficient funds for the transaction fees
2. Pay special attention to change outputs when dealing with multiple accounts and the same asset ID:
   - In Fuel, only one change output is allowed per asset ID
   - If a transaction includes resources from the same asset ID for multiple accounts, only one account will receive the change from all spent resources
   - Make sure to coordinate with all parties involved to determine which account should receive the change output

## Notes

- The method automatically handles base asset requirements for fees
- It performs a dry run to validate the transaction before returning
- The assembled transaction will include all necessary inputs and outputs based on the provided coin quantities
- Gas and fee estimation is performed using the specified block horizon
