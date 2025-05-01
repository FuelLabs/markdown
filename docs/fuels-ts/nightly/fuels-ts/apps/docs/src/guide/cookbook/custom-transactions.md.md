# Custom Transactions

There may be scenarios where you need to build out transactions that involve multiple program types and assets; this can be done by instantiating a [`ScriptTransactionRequest`](DOCS_API_URL/classes/_fuel_ts_account.ScriptTransactionRequest.html). This class allows you to a append multiple program types and assets to a single transaction.

Consider the following script that transfers multiple assets to a contract:

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

This script can be executed by creating a [`ScriptTransactionRequest`](DOCS_API_URL/classes/_fuel_ts_account.ScriptTransactionRequest.html), appending the resource and contract inputs/outputs and then sending the transaction, as follows:

```
// 1. Create a script transaction using the script binary
const request = new ScriptTransactionRequest({
  ...defaultTxParams,
  gasLimit: 3_000_000,
  script: ScriptTransferToContract.bytecode,
});

// 2. Instantiate the script main arguments
const scriptArguments = [
  contract.id.toB256(),
  { bits: ASSET_A },
  new BN(1000),
  { bits: ASSET_B },
  new BN(500),
];

// 3. Populate the script data and add the contract input and output
request
  .setData(ScriptTransferToContract.abi, scriptArguments)
  .addContractInputAndOutput(contract.id);

// 4. Get the transaction resources
const quantities = [
  coinQuantityfy([1000, ASSET_A]),
  coinQuantityfy([500, ASSET_B]),
];

// 5. Estimate and fund the transaction
await request.estimateAndFund(wallet, { quantities });

// 6. Send the transaction
const tx = await wallet.sendTransaction(request);
await tx.waitForResult();

const contractFinalBalanceAssetA = await contract.getBalance(ASSET_A);
const contractFinalBalanceAssetB = await contract.getBalance(ASSET_B);
```

## Full Example

For a full example, see below:

```
import { BN, ScriptTransactionRequest, coinQuantityfy } from 'fuels';
import { ASSET_A, ASSET_B, launchTestNode } from 'fuels/test-utils';

import { EchoValuesFactory } from '../../../typegend/contracts/EchoValuesFactory';
import { ScriptTransferToContract } from '../../../typegend/scripts/ScriptTransferToContract';

using launched = await launchTestNode({
  contractsConfigs: [{ factory: EchoValuesFactory }],
});
const {
  contracts: [contract],
  wallets: [wallet],
} = launched;

const defaultTxParams = {
  gasLimit: 10000,
};

// #region custom-transactions-2

// 1. Create a script transaction using the script binary
const request = new ScriptTransactionRequest({
  ...defaultTxParams,
  gasLimit: 3_000_000,
  script: ScriptTransferToContract.bytecode,
});

// 2. Instantiate the script main arguments
const scriptArguments = [
  contract.id.toB256(),
  { bits: ASSET_A },
  new BN(1000),
  { bits: ASSET_B },
  new BN(500),
];

// 3. Populate the script data and add the contract input and output
request
  .setData(ScriptTransferToContract.abi, scriptArguments)
  .addContractInputAndOutput(contract.id);

// 4. Get the transaction resources
const quantities = [
  coinQuantityfy([1000, ASSET_A]),
  coinQuantityfy([500, ASSET_B]),
];

// 5. Estimate and fund the transaction
await request.estimateAndFund(wallet, { quantities });

// 6. Send the transaction
const tx = await wallet.sendTransaction(request);
await tx.waitForResult();

const contractFinalBalanceAssetA = await contract.getBalance(ASSET_A);
const contractFinalBalanceAssetB = await contract.getBalance(ASSET_B);
// #endregion custom-transactions-2
```
