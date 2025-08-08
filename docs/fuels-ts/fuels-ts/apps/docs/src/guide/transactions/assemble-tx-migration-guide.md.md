# AssembleTx Migrating Guide

The old methods of estimating and funding transaction were deprecated in favor of a more robust `provider.assembleTx`. This guide will help you migrate your code to use the new method.

## Migrating From `getTransactionCost` and `fund`

### Old Approach (Deprecated)

```
const transferAmount = 100;

const request = new ScriptTransactionRequest();

// Add a coin output to transfer 100 base asset to account
request.addCoinOutput(accountB.address, transferAmount, baseAssetId);

const txCost = await accountA.getTransactionCost(request);

request.maxFee = txCost.maxFee;
request.gasLimit = txCost.gasUsed;

await accountA.fund(request, txCost);

const tx = await accountA.sendTransaction(request);
const { isStatusSuccess } = await tx.waitForResult();
```

### New Approach

```
const transferAmount = 100;

const request = new ScriptTransactionRequest();
request.addCoinOutput(accountB.address, transferAmount, baseAssetId);

const { assembledRequest } = await provider.assembleTx({
  request,
  feePayerAccount: accountA,
  accountCoinQuantities: [
    {
      amount: transferAmount,
      assetId: baseAssetId,
      account: accountA,
      changeOutputAccount: accountA,
    },
  ],
});

const tx = await accountA.sendTransaction(assembledRequest);
const { isStatusSuccess } = await tx.waitForResult();
```

### More Complex Transactions

Consider the following Sway script

```
script;

use std::asset::transfer;

fn main(
    contract_address: b256,
    asset_a: AssetId,
    amount_asset_a: u64,
    asset_b: AssetId,
    amount_asset_b: u64,
) -> bool {
    let wrapped_contract = ContractId::from(contract_address);
    let contract_id = Identity::ContractId(wrapped_contract);
    transfer(contract_id, asset_a, amount_asset_a);
    transfer(contract_id, asset_b, amount_asset_b);
    true
}
```

This script transfers two asset amounts to the same contract address. To ensure the transaction succeeds, it must be properly funded with the required amounts. This means we need to explicitly define how the transaction should be funded.

### Old Approach (Deprecated)

```
const transferAmountA = 400;
const transferAmountB = 600;

const script = new ScriptTransferToContract(account);

const request = await script.functions
  .main(
    contractId,
    { bits: TestAssetId.A.value },
    transferAmountA,
    { bits: TestAssetId.B.value },
    transferAmountB
  )
  .addContracts([contract])
  .getTransactionRequest();

const txCost = await account.getTransactionCost(request, {
  quantities: [
    {
      amount: bn(transferAmountA),
      assetId: TestAssetId.A.value,
    },
    {
      amount: bn(transferAmountB),
      assetId: TestAssetId.B.value,
    },
  ],
});

request.maxFee = txCost.maxFee;
request.gasLimit = txCost.gasUsed;

await account.fund(request, txCost);

const tx = await account.sendTransaction(request);
const { isStatusSuccess } = await tx.waitForResult();
```

### New Approach

```
const transferAmountA = 400;
const transferAmountB = 600;

const script = new ScriptTransferToContract(account);

const scope = script.functions
  .main(
    contractId,
    { bits: TestAssetId.A.value },
    transferAmountA,
    { bits: TestAssetId.B.value },
    transferAmountB
  )
  .assembleTxParams({
    feePayerAccount: account,
    accountCoinQuantities: [
      {
        amount: transferAmountA,
        assetId: TestAssetId.A.value,
        account,
        changeOutputAccount: account,
      },
      {
        amount: transferAmountB,
        assetId: TestAssetId.B.value,
        account,
        changeOutputAccount: account,
      },
    ],
  });

const { waitForResult } = await scope.call();

const {
  transactionResult: { isStatusSuccess },
} = await waitForResult();
```

By specifying which parameters `assembleTx` should use, we gain control over how the script call is estimated and funded.

## Migrating from `estimateAndFund`

### Old Code

```
const request = new ScriptTransactionRequest();

// Add a coin output to transfer 100 base asset to accountB
request.addCoinOutput(accountB.address, 100, baseAssetId);

// Estimate and fund the request
await request.estimateAndFund(accountA);

// Send the transaction
const tx = await accountA.sendTransaction(request);
await tx.waitForResult();
```

### New Code

```
const request = new ScriptTransactionRequest();

// Add a coin output to transfer 100 base asset to accountB
request.addCoinOutput(accountB.address, 100, baseAssetId);

const { assembledRequest } = await provider.assembleTx({
  request,
  feePayerAccount: accountA,
  accountCoinQuantities: [
    {
      amount: 100,
      assetId: baseAssetId,
      account: accountA,
      changeOutputAccount: accountA,
    },
  ],
});

const tx = await accountA.sendTransaction(assembledRequest);
await tx.waitForResult();
```

### Key Differences

1. **More Explicit Control**: `assembleTx` provides clearer control over which account pays fees and which accounts provide resources.

2. **Better Resource Management**: The new method allows you to specify exactly which accounts should provide which quantities of assets.

3. **Controlling Change Output**: When informing each account coin quantity you have control over determining who is going to receive a the change for that specific asset ID.

## Notes

- The methods `getTransactionCost` and `estimateAndFund` are deprecated and are going to be removed on futures updates
- The new method provides more predictable transaction assembly

## Additional Options

The new `assembleTx` method provides several additional options:

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

You can read more about the `assembleTx` [here](./index.md).
