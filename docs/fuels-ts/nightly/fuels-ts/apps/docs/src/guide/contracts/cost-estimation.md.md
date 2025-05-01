# Estimating Contract Call Cost

The [`FunctionInvocationScope.getTransactionCost`](DOCS_API_URL/classes/_fuel_ts_program.FunctionInvocationScope.html#getTransactionCost) method allows you to estimate the cost of a specific contract call. The return type, `TransactionCost`, is an object containing relevant information for the estimation:

```
export type TransactionCost = {
  gasPrice: BN;
  gasUsed: BN;
  minGas: BN;
  minFee: BN;
  maxFee: BN;
  maxGas: BN;
  rawReceipts: TransactionReceiptJson[];
  receipts: TransactionResultReceipt[];
  outputVariables: number;
  missingContractIds: string[];
  estimatedPredicates: TransactionRequestInput[];
  requiredQuantities: CoinQuantity[];
  addedSignatures: number;
  dryRunStatus?: DryRunStatus;
  updateMaxFee?: boolean;
  transactionSummary?: TransactionSummaryJsonPartial;
};
```

The following example demonstrates how to get the estimated transaction cost for:

## 1. Single contract call transaction:

```
const cost = await contract.functions
  .return_context_amount()
  .callParams({
    forward: [100, baseAssetId],
  })
  .getTransactionCost();

console.log('costs', cost);
```

## 2. Multiple contract calls transaction:

```
const scope = contract.multiCall([
  contract.functions.return_context_amount().callParams({
    forward: [100, baseAssetId],
  }),
  contract.functions.return_context_amount().callParams({
    forward: [300, baseAssetId],
  }),
]);

const txCost = await scope.getTransactionCost();

console.log('costs', txCost);
```

You can use the transaction cost estimation to set the gas limit for an actual call or display the estimated cost to the user.
