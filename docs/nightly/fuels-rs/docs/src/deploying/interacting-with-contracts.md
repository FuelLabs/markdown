# Interacting with contracts

If you already have a deployed contract and want to call its methods using the SDK, but without deploying it again, all you need is the contract ID of your deployed contract. You can skip the whole deployment setup and call `::new(contract_id, wallet)` directly. For example:

```rust,ignore
        abigen!(Contract(
            name = "MyContract",
            // Replace with your contract ABI.json path
            abi = "e2e/sway/contracts/contract_test/out/release/contract_test-abi.json"
        ));
        let wallet_original = launch_provider_and_get_wallet().await?;

        let wallet = wallet_original.clone();
        let contract_id: ContractId =
            "0x65b6a3d081966040bbccbb7f79ac91b48c635729c59a4c02f15ae7da999b32d3".parse()?;

        let connected_contract_instance = MyContract::new(contract_id, wallet);
```
