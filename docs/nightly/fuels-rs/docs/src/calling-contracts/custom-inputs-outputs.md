# Custom inputs and outputs

If you need to add specific inputs and outputs to contract calls, you can use the `with_inputs` and `with_outputs` methods.

```rust,ignore
        let _ = contract_instance
            .methods()
            .initialize_counter(42)
            .with_inputs(custom_inputs)
            .with_outputs(custom_outputs)
            .add_signer(wallet_2.signer().clone())
            .call()
            .await?;
```

> **Note:** if custom inputs include coins that need to be signed, use the `add_signer` method to add the appropriate signer.
