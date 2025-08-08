# Transaction Response

Once a transaction has been submitted, you may want to extract information regarding the result of the transaction. The SDK offers a `TransactionResponse` class with helper methods to expose the following information:

- The transaction ID
- The status (submitted, success, squeezed out, or failure)
- Receipts (return data, logs, mints/burns, transfers and panic/reverts)
- Operations (contract calls, transfers, withdrawals)
- Gas fees and usages
- Date and time of the transaction
- The block the transaction was included in

We can easily extract this information from a contract call:

```
// Call a contract function
const call = await contract.functions.increment_count(15).call();

// Wait for the result
const { transactionResponse } = await call.waitForResult();

// Retrieve the full transaction summary
const transactionSummary = await transactionResponse.getTransactionSummary({
  // Pass a Contract ID and ABI map to generate the contract operations
  [contract.id.toB256()]: Counter.abi,
});
```

We can also use the result of a transaction request to extract a transaction summary:

```
/**
 * Instantiate the transaction request using a ScriptTransactionRequest and
 * set the script main function arguments
 */
const request = new ScriptTransactionRequest({
  script: ScriptSum.bytecode,
});

request.setData(ScriptSum.abi, [1]);

// Estimate and funding the transaction
const { assembledRequest } = await provider.assembleTx({
  request,
  feePayerAccount: wallet,
  accountCoinQuantities: [
    {
      amount: 1000,
      assetId: await provider.getBaseAssetId(),
      account: wallet,
      changeOutputAccount: wallet,
    },
  ],
});

// Submit the transaction
const response = await wallet.sendTransaction(assembledRequest);

// Generate the transaction summary
const transactionSummary = await response.getTransactionSummary();
```

Or we can build a transaction summary from a stored transaction ID:

```
// Take a transaction ID from a previous transaction
const transactionId = previouslySubmittedTransactionId;
// 0x...

// Retrieve the transaction response from the transaction ID
const transactionResponse = await TransactionResponse.create(
  transactionId,
  provider
);

// Generate the transaction summary
const summary = await transactionResponse.getTransactionSummary();
```
