# Working with Contract Logs

When you log a value within a contract method, you can generate a log entry that is added to the log receipt, and the variable type is recorded in the contract's ABI. The SDK enables you to parse these values into TypeScript types.

## Simple Logs

Consider the following example contract:

```
contract;

use log_simple_abi::LogSimple;

impl LogSimple for Contract {
    fn log_simple(val: str[9]) {
        log(val);
    }
}
```

To access the logged values in TypeScript, use the `logs` (typed as `Array<any>`) property from the response of a contract call.

```
const deploy = await LogSimpleFactory.deploy(wallet);
const { contract } = await deploy.waitForResult();

const { waitForResult } = await contract.functions
  .log_simple('ContractA')
  .call();

const { logs } = await waitForResult();
// logs = [
//   'ContractA'
// ]
```

## Grouped Logs

We also provide a `groupedLogs` property that groups the logs by their program identifier. This is particularly useful when working with inter-contract or multi-calls.

We will use the same [`LogSimple`](#simple-logs) contract as in the previous example.

### Multi-call

We can make a multi-call to the contract

```
const deploy = await LogSimpleFactory.deploy(wallet);
const { contract: contractA } = await deploy.waitForResult();

const { waitForResult } = await contractA
  .multiCall([
    contractA.functions.log_simple('Contract1'),
    contractA.functions.log_simple('Contract2'),
  ])
  .call();

const { groupedLogs } = await waitForResult();
// groupedLogs = {
//   [contractA.id.toB256()]: ["Contract1", "Contract2"]
// }
```

### Inter-contract

Consider the following example contract:

In this example, we are making a call an inter-contract call to the [`LogSimple`](#simple-logs) contract from the previous example.

```
contract;

// Interface from the `LogSimple` contract
abi LogSimple {
    fn log_simple(val: str[9]);
}

// Interface for the contract
abi LogInterCalls {
    fn log_inter_call(contract_id: b256, simple_log_message: str[9]);
}

impl LogInterCalls for Contract {
    fn log_inter_call(contract_id: b256, simple_log_message: str[9]) {
        log("Starting inter-call");

        let logger = abi(LogSimple, contract_id);
        logger.log_simple(simple_log_message);

        log("Inter-call completed");
    }
}
```

The `log_inter_call` function makes a call to the `log_simple` function of the [`LogSimple`](#simple-logs) contract.

```
// First we make a simple contract that logs a value
const deploySimpleContract = await LogSimpleFactory.deploy(wallet);
const { contract: simpleContract } = await deploySimpleContract.waitForResult();

// Then we make an inter-contract that makes a multi-call to the simple contract
const deployInterContract = await LogInterCallsFactory.deploy(wallet);
const { contract: interContract } = await deployInterContract.waitForResult();

// We can then call the inter-contract function that makes call out to the simple contract
const { waitForResult } = await interContract.functions
  .log_inter_call(simpleContract.id.toB256(), 'ContractB')
  .call();

// We can then wait for the result and get the grouped logs
const { groupedLogs } = await waitForResult();
// groupedLogs = {
//   [simpleContract.id.toB256()]: ['ContractB'],
//   [interContract.id.toB256()]: ['Starting inter-call', 'Inter-call completed'],
// };
```
