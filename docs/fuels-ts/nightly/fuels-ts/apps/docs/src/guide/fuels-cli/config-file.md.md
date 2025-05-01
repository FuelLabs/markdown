# Config File

Here, you can learn more about all configuration options.

## `workspace`

Relative directory path to Forc workspace.

```
  workspace: './sway-programs',
```

> _The property `workspace` is incompatible with [`contracts`](#contracts), [`predicates`](#predicates), and [`scripts`](#scripts)._

## `contracts`

List of relative directory paths to Sway contracts.

```
  contracts: ['./sway-programs/contracts'],
```

> _The property `contracts` is incompatible with [`workspace`](#workspace)._

## `predicates`

List of relative directory paths to Sway predicates.

```
  predicates: ['./sway-programs/predicates'],
```

> _The property `predicates` is incompatible with [`workspace`](#workspace)._

## `scripts`

List of relative directory paths to Sway scripts.

```
  scripts: ['./sway-programs/scripts'],
```

> _The property `scripts` is incompatible with [`workspace`](#workspace)._

## `output`

Relative directory path to use when generating Typescript definitions.

```
  output: './src/sway-programs-api',
```

## `providerUrl`

The URL to use when deploying contracts.

```
  // Default: http://127.0.0.1:4000/v1/graphql
  providerUrl: 'http://network:port/v1/graphql',
```

> _When [`autostartFuelCore`](#autostartfuelcore) property is set to `true`, the `providedUrl` is overridden by that of the local short-lived `fuel-core` node started by the [`fuels dev`](./commands.md#fuels-dev) command._

## `privateKey`

Wallet private key, used when deploying contracts.

This property should ideally come from env — `process.env.MY_PRIVATE_KEY`.

```
  privateKey: '0xa449b1ffee0e2205fa924c6740cc48b3b473aa28587df6dab12abc245d1f5298',
```

> _When [`autostartFuelCore`](#autostartfuelcore) property is set to `true`, the `privateKey` is overridden with the `consensusKey` of the local short-lived `fuel-core` node started by the [`fuels dev`](./commands.md#fuels-dev) command._

## `snapshotDir`

> - _Used by [`fuels dev`](./commands.md#fuels-dev) only_.

Relative path to directory containing custom configurations for `fuel-core`, such as:

- `chainConfig.json`
- `metadata.json`
- `stateConfig.json`

This will take effect only when [`autoStartFuelCore`](#autostartfuelcore) is `true`.

```
  snapshotDir: './my/snapshot/dir',
```

## `autoStartFuelCore`

> - _Used by [`fuels dev`](./commands.md#fuels-dev) only_.

When set to `true`, it will automatically:

1. Starts a short-lived `fuel-core` node as part of the [`fuels dev`](./commands.md#fuels-dev) command
1. Override property [`providerUrl`](#providerurl) with the URL for the recently started `fuel-core` node

```
  autoStartFuelCore: true,
```

If set to `false`, you must spin up a `fuel-core` node by yourself and set the URL for it via [`providerUrl`](#providerurl).

## `fuelCorePort`

> - _Used by [`fuels dev`](./commands.md#fuels-dev) only_.
> - _Ignored when [`autoStartFuelCore`](#autostartfuelcore) is set to `false`._

Port to use when starting a local `fuel-core` node.

```
  // Default: first free port, starting from 4000
  fuelCorePort: 4000,
```

## `forcBuildFlags`

> - _Used by [`fuels build`](./commands.md#fuels-build) and [`fuels deploy`](./commands.md#fuels-deploy)_.

Sway programs are compiled in `debug` mode by default.

Here you can customize all build flags, e.g. to build programs in `release` mode.

```
  // Default: []
  forcBuildFlags: ['--release'],
```

Check also:

- [Forc docs](https://docs.fuel.network/docs/forc/commands/forc_build/#forc-build)

## `deployConfig`

You can supply a ready-to-go deploy configuration object:

```
  deployConfig: {},
```

Or use a function for crafting dynamic deployment flows:

- If you need to fetch and use configs or data from a remote data source
- If you need to use IDs from already deployed contracts — in this case, we can use the `options.contracts` property to get the necessary contract ID. For example:

```
  deployConfig: async (options: ContractDeployOptions) => {
    // ability to fetch data remotely
    await Promise.resolve(`simulating remote data fetch`);

    // get contract by name
    const { contracts } = options;

    const contract = contracts.find(({ name }) => {
      const found = name === MY_FIRST_DEPLOYED_CONTRACT_NAME;
      return found;
    });

    if (!contract) {
      throw new Error('Contract not found!');
    }

    return {
      storageSlots: [
        {
          key: '0x..',
          /**
           * Here we could initialize a storage slot,
           * using the relevant contract ID.
           */
          value: contract.contractId,
        },
      ],
    };
  },
```

## `onBuild`

A callback function that is called after a build event has been successful.

Parameters:

- `config` — The loaded config (`fuels.config.ts`)

```
  onBuild: (config: FuelsConfig): void | Promise<void> => {
    console.log('fuels:onBuild', { config });
  },
```

## `onDeploy`

A callback function that is called after a deployment event has been successful.

Parameters:

- `config` — The loaded config (`fuels.config.ts`)
- `data` — The data (an array of deployed contracts)

```
  onDeploy: (config: FuelsConfig, data: DeployedData): void | Promise<void> => {
    console.log('fuels:onDeploy', { config, data });
  },
```

## `onDev`

A callback function that is called after the [`fuels dev`](./commands.md#fuels-dev) command has successfully restarted.

Parameters:

- `config` — The loaded config (`fuels.config.ts`)

```
  onDev: (config: FuelsConfig): void | Promise<void> => {
    console.log('fuels:onDev', { config });
  },
```

## `onNode`

A callback function that is called after the [`fuels node`](./commands.md#fuels-node) command has successfully refreshed.

Parameters:

- `config` — The loaded config (`fuels.config.ts`)

```
  onNode: (config: FuelsConfig): void | Promise<void> => {
    console.log('fuels:onNode', { config });
  },
```

## `onFailure`

Pass a callback function to be called in case of errors.

Parameters:

- `config` — The loaded config (`fuels.config.ts`)
- `error` — Original error object

```
  onFailure: (config: FuelsConfig, error: Error): void | Promise<void> => {
    console.log('fuels:onFailure', { config, error });
  },
```

## `forcPath`

Path to the `forc` binary.

When not supplied, will default to using the `system` binaries (`forc`).

```
  // Default: 'forc',
  forcPath: '~/.fuelup/bin/forc',
```

## `fuelCorePath`

Path to the `fuel-core` binary.

When not supplied, will default to using the `system` binaries (`fuel-core`).

```
  // Default: 'fuel-core'
  fuelCorePath: '~/.fuelup/bin/fuel-core',
```

## Loading environment variables

If you want to load environment variables from a `.env` file, you can use the `dotenv` package.

First, install it:

::: code-group

```sh [pnpm]
pnpm install dotenv
```

```sh [npm]
npm install dotenv
```

```sh [bun]
bun install dotenv
```

:::

Then, you can use it in your `fuels.config.ts` file:

```
import { createConfig } from 'fuels';
import dotenv from 'dotenv';
import { providerUrl } from './src/lib';

dotenv.config({
  path: ['.env.local', '.env'],
});

// If your node is running on a port other than 4000, you can set it here
const fuelCorePort = +(process.env.VITE_FUEL_NODE_PORT as string) || 4000;

export default createConfig({
  workspace: './sway-programs', // Path to your Sway workspace
  output: './src/sway-api', // Where your generated types will be saved
  fuelCorePort,
  providerUrl,
  forcPath: 'fuels-forc',
  fuelCorePath: 'fuels-core',
});
```
