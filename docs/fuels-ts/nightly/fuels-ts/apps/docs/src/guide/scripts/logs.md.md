# Working with Script Logs

When you log a value within a script, you can generates a log entry that is added to the log receipt, and the variable type is recorded in the script's ABI. The SDK enables you to parse these values into TypeScript types.

## Simple

Consider the following example script:

```
script;

fn main(log_value: str[7]) -> str[7] {
    log(log_value);
    log_value
}
```

To access the logged values in TypeScript, use the `logs` property in the response of a script call.

```
import { Provider, Wallet } from 'fuels';

import { WALLET_PVT_KEY, LOCAL_NETWORK_URL } from '../../../env';
import { ScriptLogSimple } from '../../../typegend';

const provider = new Provider(LOCAL_NETWORK_URL);
const wallet = Wallet.fromPrivateKey(WALLET_PVT_KEY, provider);

const script = new ScriptLogSimple(wallet);
const { waitForResult } = await script.functions.main('ScriptA').call();

const { logs } = await waitForResult();
// logs: ['ScriptA']
```

## Grouped logs

Consider the following example script:

```
script;

use log_simple_abi::LogSimple;

fn main(contract_id: b256) {
    log("Script started");

    let log_contract = abi(LogSimple, contract_id);
    log_contract.log_simple(__to_str_array("ContractA"));

    log("Script finished");
}
```

### With Contract

To access the fine-grained logs for each contract, use the `groupedLogs` property from the script call response.

```
import { Provider, Wallet, ZeroBytes32 } from 'fuels';

import { WALLET_PVT_KEY, LOCAL_NETWORK_URL } from '../../../env';
import { LogSimpleFactory, ScriptLogWithContract } from '../../../typegend';

const provider = new Provider(LOCAL_NETWORK_URL);
const wallet = Wallet.fromPrivateKey(WALLET_PVT_KEY, provider);

// Create a contract instance
const { waitForResult: waitForDeploy } = await LogSimpleFactory.deploy(wallet);
const { contract } = await waitForDeploy();

// Create a script instance
const script = new ScriptLogWithContract(wallet);

// Call the script
const { waitForResult } = await script.functions
  .main(contract.id.toB256())
  .addContracts([contract])
  .call();

// Wait for the script to finish and get the logs
const { groupedLogs } = await waitForResult();
// groupedLogs = {
//   [ZeroBytes32]: ['Script started', 'Script finished'],
//   [contract.id.toB256()]: ['ContractA', 'ContractB'],
// }
```

Although Scripts don't have IDs/addresses, they can still call contracts and generate logs, so we use a zeroed-out (hexadecimal) address as their key instead.
