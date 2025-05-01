# Setting up test wallets

You'll often want to create one or more test wallets when testing your contracts. Here's how to do it.

## Create a single wallet

```
import type { WalletLocked, WalletUnlocked } from 'fuels';
import { Provider, Wallet } from 'fuels';

import { LOCAL_NETWORK_URL } from '../../../env';

// We can use the `generate` to create a new unlocked wallet.
const provider = new Provider(LOCAL_NETWORK_URL);
const myWallet: WalletUnlocked = Wallet.generate({ provider });

// or use an Address to create a wallet
const someWallet: WalletLocked = Wallet.fromAddress(myWallet.address, provider);
```

## Setting up multiple test wallets

You can set up multiple test wallets using the `launchTestNode` utility via the `walletsConfigs` option.

To understand the different configurations, check out the [walletsConfig](./test-node-options.md#walletsconfig) in the test node options guide.

```
using launched = await launchTestNode({
  walletsConfig: {
    count: 3,
    assets: [TestAssetId.A, TestAssetId.B],
    coinsPerAsset: 5,
    amountPerCoin: 100_000,
  },
});

const {
  wallets: [wallet1, wallet2, wallet3],
} = launched;
```
