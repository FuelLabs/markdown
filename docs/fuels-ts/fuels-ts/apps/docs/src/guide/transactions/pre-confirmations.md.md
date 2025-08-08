# Pre-Confirmations

## What is a Pre-Confirmation?

A **pre-confirmation** is an intermediate transaction status that occurs after a transaction has been **submitted** and **accepted** by the blockchain, but **before** it is fully **processed and included** in a new block.

At this stage, the transaction is pre-executed and assigned one of two possible statuses:

- `PreconfirmationSuccessStatus`: The transaction is expected to be successfully included in a future block.

- `PreconfirmationFailureStatus`: The transaction will **not** be included in any future block.

## Why are Pre-Confirmations important?

Pre-confirmations allow applications to **react earlier** by providing immediate feedback about a transaction's expected outcome without waiting for full block finalization.

Additionally, pre-confirmations expose **processed outputs** (such as `OutputChange` and `OutputVariable`) that can be **immediately reused** in new transactions.

## Available Outputs for Pre-Confirmations

When a transaction reaches the **pre-confirmation** stage, certain `resolvedOutputs` become available:

- `OutputChange`: Represents the change UTXO generated from unspent inputs, grouped by `assetId` (one per asset).

- `OutputVariable`: Similar to `OutputCoin`, but only created if the transaction succeeds.

These outputs can be:

- Extracted directly from the pre-confirmation response.
- **Used immediately** to fund new transactions, without waiting for block confirmation.

This significantly improves the ability to build transaction sequences or reactive transaction flows.

This is the `ResolvedOutput` interface structure:

```
export type ResolvedOutput = {
  utxoId: string;
  output: OutputChange | OutputVariable;
};
```

## Example Workflow

Suppose you send a transaction that will send funds to another wallet.

As soon as you receive a `PreconfirmationSuccessStatus`, you can:

- Use the `OutputChange` in a new transaction.
- Submit the next transaction **without waiting** for block finalization.

This reduces wait times and accelerates transaction chaining.

## Code Examples

### Using Pre-Confirmations when Submitting a Transfer

The following example sends a transfer, waits for the pre-confirmation success, and then submits another transfer using the resolved outputs from the first:

```
const provider = new Provider(LOCAL_NETWORK_URL);

const wallet = Wallet.fromPrivateKey(WALLET_PVT_KEY, provider);
const recipient1 = Wallet.fromPrivateKey(WALLET_PVT_KEY_2, provider);
const recipient2 = Wallet.fromPrivateKey(WALLET_PVT_KEY_3, provider);
const baseAssetId = await provider.getBaseAssetId();

// Send a transfer and retrieve the pre-confirmation callback
const { waitForPreConfirmation } = await wallet.transfer(
  recipient1.address,
  1000
);

// Wait for the transaction to reach a pre-confirmation status
const { resolvedOutputs, isStatusPreConfirmationSuccess } =
  await waitForPreConfirmation();

// Check if the pre-confirmation status indicates success
if (isStatusPreConfirmationSuccess) {
  // Find the change output associated with the base asset ID
  const resolvedChangeOutput = resolvedOutputs?.find(
    (resolved) =>
      resolved.output.type === OutputType.Change &&
      resolved.output.assetId === baseAssetId
  );

  // If we find the change output, we can use it to create a new transaction
  if (resolvedChangeOutput) {
    const { output, utxoId } = resolvedChangeOutput;

    // Create a new transaction request
    const newTransaction = new ScriptTransactionRequest({
      maxFee: 1000,
      gasLimit: 1000,
    });

    // Add the change output as an input resource for the new transaction
    newTransaction.addResource({
      id: utxoId,
      assetId: output.assetId,
      amount: output.amount,
      owner: new Address(output.to),
      blockCreated: bn(0),
      txCreatedIdx: bn(0),
    });

    // Define the transfer recipient of the new transaction
    newTransaction.addCoinOutput(recipient2.address, 1000, baseAssetId);

    // Send the new transaction
    await wallet.sendTransaction(newTransaction);
  }
}
```

### Using Pre-Confirmations with a Contract Call

This example performs a contract call, waits for pre-confirmation success, and then uses the resolved output to execute another contract call:

```
// Send a contract call and retrieve the pre-confirmation callback
const { waitForPreConfirmation } = await contract.functions
  .increment_count(1)
  .call();

const {
  transactionResult: { resolvedOutputs, isStatusPreConfirmationSuccess },
} = await waitForPreConfirmation();

// Check if the pre-confirmation status indicates success
if (isStatusPreConfirmationSuccess) {
  // Find the change output associated with the base asset ID
  const resolvedChangeOutput = resolvedOutputs?.find(
    (resolved) =>
      resolved.output.type === OutputType.Change &&
      resolved.output.assetId === baseAssetId
  );

  // If we find the change output, we can use it to create a new transaction
  if (resolvedChangeOutput) {
    const { output, utxoId } = resolvedChangeOutput;

    // Creates a new scope invocation for another contract call
    const scope = contract.functions.increment_count(1).txParams({
      maxFee: 100_000,
      gasLimit: 100_000,
    });

    // Get the transaction request from the scope invocation
    const request = await scope.getTransactionRequest();

    // Add the change output as an input resource for the new transaction
    request.addResource({
      id: utxoId,
      assetId: output.assetId,
      amount: output.amount,
      owner: new Address(output.to),
      blockCreated: bn(0),
      txCreatedIdx: bn(0),
    });

    /**
     * Call the scope invocation and skip the assembleTx step since the transaction
     * request is already funded
     */
    await scope.call({ skipAssembleTx: true });
  }
}
```
