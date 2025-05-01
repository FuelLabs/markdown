# Fake signer (impersonating another account)

To facilitate account impersonation, the Rust SDK provides the `FakeSigner`. We can use it to simulate ownership of assets held by an account with a given address. This also implies that we can impersonate contract calls from that address. A wallet with a `FakeSigner` will only succeed in unlocking assets if the network is set up with `utxo_validation = false`.

```rust,ignore
        let node_config = NodeConfig {
            utxo_validation: false,
            ..Default::default()
        };
        let provider = setup_test_provider(coins, vec![], Some(node_config), None).await?;
```

```rust,ignore
        let provider = setup_test_provider(coins, vec![], Some(node_config), None).await?;
```

```rust,ignore
        // create impersonator for an address
        let fake_signer = FakeSigner::new(some_address);
        let impersonator = Wallet::new(fake_signer, provider.clone());

        let contract_instance = MyContract::new(contract_id, impersonator.clone());

        let response = contract_instance
            .methods()
            .initialize_counter(42)
            .call()
            .await?;

        assert_eq!(42, response.value);
```
