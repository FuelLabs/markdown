# Encrypting and Storing Keys

The code below shows how to:

- Encrypt and store your key using a master password.
- Ensure that the key can be retrieved later with the proper credentials.

```rust,ignore
        let dir = std::env::temp_dir();
        let keystore = Keystore::new(&dir);

        let phrase =
            "oblige salon price punch saddle immune slogan rare snap desert retire surprise";

        // Create a key from the mnemonic phrase using the default derivation path.
        let key = SecretKey::new_from_mnemonic_phrase_with_path(phrase, DEFAULT_DERIVATION_PATH)?;
        let password = "my_master_password";

        // Encrypt and store the key on disk. It can be recovered using `Keystore::load_key`.
        let uuid = keystore.save_key(key, password, thread_rng())?;

        // Recover key from disk
        let recovered_key = keystore.load_key(&uuid, password)?;
```
