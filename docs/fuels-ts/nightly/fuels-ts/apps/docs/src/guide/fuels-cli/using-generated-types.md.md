<!-- TODO: Replace plan-text by code-snippets -->

# Using Generated Types

After generating types via:

```console
pnpm fuels typegen -i ./abis/*-abi.json -o ./types
```

We can use these files like so:

```
    // #context import { DemoContract } from './sway-programs-api';

    const contractInstance = new DemoContract(contractId, wallet);
    const call2 = await contractInstance.functions.return_input(1337).call();
    const { value: v2 } = await call2.waitForResult();
```

## Contract

Let's use the Contract class to deploy a contract:

```
    // #context import { DemoContract } from './sway-programs-api';

    // Deploy
    const deploy = await DemoContractFactory.deploy(wallet);
    const { contract } = await deploy.waitForResult();
```

### Autoloading of Storage Slots

Typegen tries to resolve, auto-load, and embed the [Storage Slots](../contracts/storage-slots.md) for your Contract within the `MyContract` class. Still, you can override it alongside other options from [`DeployContractOptions`](https://github.com/FuelLabs/fuels-ts/blob/a64b67b9fb2d7f764ab9151a21d2266bf2df3643/packages/contract/src/contract-factory.ts#L19-L24), when calling the `deploy` method:

```
    // #context import { DemoContractFactory } from './sway-programs-api';

    const { waitForResult } = await DemoContractFactory.deploy(wallet, {
      storageSlots,
    });

    const { contract } = await waitForResult();
```

## Script

After generating types via:

```console
pnpm fuels typegen -i ./abis/*-abi.json -o ./types --script
```

We can use these files like so:

```
  // #context import { Script } from './sway-programs-api';

  const script = new DemoScript(wallet);
  const { waitForResult } = await script.functions.main().call();
  const { value } = await waitForResult();
```

## Predicate

After generating types via:

```console
pnpm fuels typegen -i ./abis/*-abi.json -o ./types --predicate
```

We can use these files like so:

```
  // #context import type { PredicateInputs } from './sway-programs-api';
  // #context import { Predicate } from './sway-programs-api';

  // In this exchange, we are first transferring some coins to the predicate
  using launched = await launchTestNode();

  const {
    provider,
    wallets: [wallet],
  } = launched;

  const receiver = Wallet.fromAddress(Address.fromRandom(), provider);

  const predicateData: DemoPredicateInputs = [];
  const predicate = new DemoPredicate({
    provider,
    data: predicateData,
  });

  const tx = await wallet.transfer(predicate.address, 200_000, await provider.getBaseAssetId());
  const { isStatusSuccess } = await tx.wait();

  // Then we are transferring some coins from the predicate to a random address (receiver)
  const tx2 = await predicate.transfer(receiver.address, 50_000, await provider.getBaseAssetId());
  await tx2.wait();

  expect((await receiver.getBalance()).toNumber()).toEqual(50_000);
  expect(isStatusSuccess).toBeTruthy();
```

See also:

- [Generating Types for Contracts](./generating-types.md#generating-types-for-contracts)
- [Generating Types for Scripts](./generating-types.md#generating-types-for-scripts)
- [Generating Types for Predicates](./generating-types.md#generating-types-for-predicates)
