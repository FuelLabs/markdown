# Preparing a Script Transaction

Akin to Contracts, we can configure the [call parameters](../contracts/call-parameters.md) and [transaction parameters](../transactions/adding-parameters.md) for Scripts, as well as retrieve the entire transaction request prior to submission.

```
const myMainScript = new Script(ScriptSum.bytecode, ScriptSum.abi, wallet);

const tx = myMainScript.functions.main(argument);

// Set the call parameters
tx.callParams({ gasLimit: 7500 });

// Get the entire transaction request prior to
const txRequest = await tx.getTransactionRequest();

// Get the transaction ID
const chainId = await provider.getChainId();
const txId = txRequest.getTransactionId(chainId);

// Retrieve the value of the call and the actual gas used
const { waitForResult: waitForActualGasUsed } = await tx.call();
const { value: valueOfActualGasUsed, gasUsed: gasUsedOfActualGasUsed } =
  await waitForActualGasUsed();
```
