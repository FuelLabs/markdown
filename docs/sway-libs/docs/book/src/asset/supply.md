# Supply Functionality

For implementation details on the Asset Library supply functionality please see the [Sway Libs Docs](https://fuellabs.github.io/sway-libs/master/sway_libs/asset/asset/supply/index.html).

## Importing the Asset Library Supply Functionality

In order to use the supply functionality of the Asset Library, the Asset Library and the [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) Standard must be added to your `Forc.toml` file and then imported into your Sway project.

To add the Asset Library and the [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) Standard as a dependency to your `Forc.toml` file in your project, use the `forc add` command.

```bash
forc add asset@0.8.0
forc add src3@0.8.0
```

> **NOTE:** Be sure to set the version to the latest release.

To import the Asset Library Supply Functionality and [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) Standard to your Sway Smart Contract, add the following to your Sway file:

```sway
use asset::supply::*;
use src3::*;
```

## Integration with the SRC-3 Standard

The [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) definition states that the following abi implementation is required for any Native Asset on Fuel which mints and burns tokens:

```sway
abi SRC3 {
    #[storage(read, write)]
    fn mint(recipient: Identity, sub_id: Option<SubId>, amount: u64);
    #[payable]
    #[storage(read, write)]
    fn burn(vault_sub_id: SubId, amount: u64);
}
```

The Asset Library has the following complimentary functions for each function in the `SRC3` abi:

- `_mint()`
- `_burn()`

> **NOTE** The `_mint()` and `_burn()` functions will mint and burn assets *unconditionally*. External checks should be applied to restrict the minting and burning of assets.

## Setting Up Storage

Once imported, the Asset Library's supply functionality should be available. To use them, be sure to add the storage block below to your contract which enables the [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) standard.

```sway
storage {
    total_assets: u64 = 0,
    total_supply: StorageMap<AssetId, u64> = StorageMap {},
}
```

## Implementing the SRC-3 Standard with the Asset Library

To use either function, simply pass the `StorageKey` from the prescribed storage block. The example below shows the implementation of the [SRC-3](https://docs.fuel.network/docs/sway-standards/src-3-minting-and-burning/) standard in combination with the Asset Library with no user defined restrictions or custom functionality. It is recommended that the [Ownership Library](../ownership/index.md) is used in conjunction with the Asset Library's supply functionality to ensure only a single user has permissions to mint an Asset.

The `_mint()` and `_burn()` functions follows the SRC-20 standard for logging and will emit the `TotalSupplyEvent` when called.

```sway
use asset::supply::{_burn, _mint};
use src3::SRC3;

storage {
    total_assets: u64 = 0,
    total_supply: StorageMap<AssetId, u64> = StorageMap {},
}

// Implement the SRC-3 Standard for this contract
impl SRC3 for Contract {
    #[storage(read, write)]
    fn mint(recipient: Identity, sub_id: Option<SubId>, amount: u64) {
        // Pass the StorageKeys to the `_mint()` function from the Asset Library.
        _mint(
            storage
                .total_assets,
            storage
                .total_supply,
            recipient,
            sub_id
                .unwrap_or(b256::zero()),
            amount,
        );
    }

    // Pass the StorageKeys to the `_burn_()` function from the Asset Library.
    #[payable]
    #[storage(read, write)]
    fn burn(sub_id: SubId, amount: u64) {
        _burn(storage.total_supply, sub_id, amount);
    }
}
```

> **NOTE** The `_mint()` and `_burn()` functions will mint and burn assets *unconditionally*. External checks should be applied to restrict the minting and burning of assets.
