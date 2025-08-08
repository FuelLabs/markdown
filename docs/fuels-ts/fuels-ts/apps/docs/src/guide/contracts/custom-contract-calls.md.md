# Custom Contract Call

In certain scenarios, your use case may require more control over how contract calls are prepared and submitted.

For instance, imagine a liquidity pool contract where users can deposit a specific asset to receive some form of benefit. To enhance the experience and make it more attractive, one could use a predicate to cover the transaction fees. This way, users only need to provide the asset they wish to deposit without worrying about the fees.

There are two main ways to customize a contract call:

## Approach 1: Customize `assembleTx` Parameters

In most cases, this isn’t necessary. However, if you need precise control, you can specify the parameters passed to `assembleTx`, which is used internally to estimate and fund the transaction.

Here’s how it works:

```
// Create a new contract instance using the contract id
const liquidityPoolContract = new LiquidityPool(contractId, wallet);

// Execute the contract call manually specifying the assembleTx parameters
const { waitForResult } = await liquidityPoolContract.functions
  .deposit({ bits: wallet.address.toB256() })
  .callParams({
    forward: [1000, TestAssetId.A.value],
  })
  .assembleTxParams({
    feePayerAccount: predicate, // Using predicate as fee payer
    accountCoinQuantities: [
      {
        amount: 1000,
        assetId: TestAssetId.A.value,
        account: wallet,
        changeOutputAccount: wallet,
      },
    ],
  })
  .call();

const {
  transactionResult: { isStatusSuccess },
} = await waitForResult();
```

## Approach 2: Manually Call `assembleTx`

You can also retrieve the transaction request from the invocation scope and manually call `assembleTx` on it. Just like the approach 1 this gives you full control over how the transaction is assembled and funded.

```
// Create a new contract instance using the contract id
const liquidityPoolContract = new LiquidityPool(contractId, wallet);

// Create invocation scope to call the deposit function
const scope = liquidityPoolContract.functions
  .deposit({ bits: wallet.address.toB256() })
  .callParams({
    forward: [1000, TestAssetId.A.value],
  });

// Get the transaction request
const request = await scope.getTransactionRequest();

/**
 * Using "assembleTx" to estimate and fund the transaction.
 */
await provider.assembleTx({
  request,
  feePayerAccount: predicate, // Using predicate as fee payer
  accountCoinQuantities: [
    {
      amount: 1000,
      assetId: TestAssetId.A.value,
      account: wallet,
      changeOutputAccount: wallet,
    },
  ],
});

// Use the "call" method to submit the transaction, skipping the "assembleTx" step
const response = await scope
  .fromRequest(request)
  .call({ skipAssembleTx: true });

const {
  transactionResult: { isStatusSuccess },
} = await response.waitForResult();
```

There are 2 details here that there are essential in this flow

1. `fromRequest`:
   This sets the transaction request extracted from the invocation scope. It ensures that any manual changes you’ve applied persist and are used when the transaction is executed.

1. `{ skipAssembleTx: true }` option:
   It tells the SDK to skip the automatic `assembleTx` step during the `.call()` execution, since we’ve already manually assembled and funded the transaction using `provider.assembleTx()`. Skipping this step prevents the SDK from re-estimating the transaction and ensures the custom logic remains intact, such as using a predicate as the fee payer.
