# Converting Types

Below are some common type conversions in Sway:

- [Identity Conversions](#identity-conversions)
- [String Conversions](#string-conversions)
- [Number Conversions](#number-conversions)
- [Byte Array Conversions](#byte-array-conversions)

## Identity Conversions

### Convert to `Identity`

```sway
    let identity_from_b256: Identity = Identity::Address(Address::from(b256_address));
    let identity_from_address: Identity = Identity::Address(address);
    let identity_from_contract_id: Identity = Identity::ContractId(contract_id);
```

### Convert `Identity` to `ContractId` or `Address`

```sway
    match my_identity {
        Identity::Address(address) => log(address),
        Identity::ContractId(contract_id) => log(contract_id),
    };
```

### Convert `ContractId` or `Address` to `b256`

```sway
    let b256_from_address: b256 = address.into();
    let b256_from_contract_id: b256 = contract_id.into();
```

### Convert `b256` to `ContractId` or `Address`

```sway
    let address_from_b256: Address = Address::from(b256_address);
    let contract_id_from_b256: ContractId = ContractId::from(b256_address);
```

## String Conversions

### Convert `str` to `str[]`

```sway
    let fuel_str: str = "fuel";
    let fuel_str_array: str[4] = fuel_str.try_as_str_array().unwrap();
```

### Convert `str[]` to `str`

```sway
    let fuel_str_array: str[4] = __to_str_array("fuel");
    let fuel_str: str = from_str_array(fuel_str_array);
```

## Number Conversions

### Convert to `u256`

```sway
    let u8_1: u8 = 2u8;
    let u16_1: u16 = 2u16;
    let u32_1: u32 = 2u32;
    let u64_1: u64 = 2u64;
    let b256_1: b256 = 0x0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20;

    let u256_from_u8: u256 = u8_1.as_u256();
    let u256_from_u16: u256 = u16_1.as_u256();
    let u256_from_u32: u256 = u32_1.as_u256();
    let u256_from_u64: u256 = u64_1.as_u256();
    let u256_from_b256: u256 = b256_1.as_u256();
```

### Convert to `u64`

```sway
    let u8_1: u8 = 2u8;
    let u16_1: u16 = 2u16;
    let u32_1: u32 = 2u32;
    let u256_1: u256 = 0x0000000000000000000000000000000000000000000000000000000000000002u256;

    let u64_from_u8: u64 = u8_1.as_u64();

    let u64_from_u16: u64 = u16_1.as_u64();

    let u64_from_u32: u64 = u32_1.as_u64();

    let u64_from_u256: Option<u64> = <u64 as TryFrom<u256>>::try_from(u256_1);
```

### Convert to `u32`

```sway
    let u8_1: u8 = 2u8;
    let u16_1: u16 = 2u16;
    let u64_1: u64 = 2;
    let u256_1: u256 = 0x0000000000000000000000000000000000000000000000000000000000000002u256;

    let u32_from_u8: u32 = u8_1.as_u32();

    let u32_from_u16: u32 = u16_1.as_u32();

    let u32_from_u64_1: Option<u32> = u64_1.try_as_u32();
    let u32_from_u64_2: Option<u32> = <u32 as TryFrom<u64>>::try_from(u64_1);

    let u32_from_u256: Option<u32> = <u32 as TryFrom<u256>>::try_from(u256_1);
```

### Convert to `u16`

```sway
    let u8_1: u8 = 2u8;
    let u32_1: u32 = 2u32;
    let u64_1: u64 = 2;
    let u256_1: u256 = 0x0000000000000000000000000000000000000000000000000000000000000002u256;

    let u16_from_u8: u16 = u8_1.as_u16();

    let u16_from_u32_1: Option<u16> = u32_1.try_as_u16();
    let u16_from_u32_2: Option<u16> = <u16 as TryFrom<u32>>::try_from(u32_1);

    let u16_from_u64_1: Option<u16> = u64_1.try_as_u16();
    let u16_from_u64_2: Option<u16> = <u16 as TryFrom<u64>>::try_from(u64_1);

    let u16_from_u256: Option<u16> = <u16 as TryFrom<u256>>::try_from(u256_1);
```

### Convert to `u8`

```sway
    let u16_1: u16 = 2u16;
    let u32_1: u32 = 2u32;
    let u64_1: u64 = 2;
    let u256_1: u256 = 0x0000000000000000000000000000000000000000000000000000000000000002u256;

    let u8_from_u16_1: Option<u8> = u16_1.try_as_u8();
    let u8_from_u16_2: Option<u8> = <u8 as TryFrom<u16>>::try_from(u16_1);

    let u8_from_u32_1: Option<u8> = u32_1.try_as_u8();
    let u8_from_u32_2: Option<u8> = <u8 as TryFrom<u32>>::try_from(u32_1);

    let u8_from_u64_1: Option<u8> = u64_1.try_as_u8();
    let u8_from_u64_2: Option<u8> = <u8 as TryFrom<u64>>::try_from(u64_1);

    let u8_from_u256: Option<u8> = <u8 as TryFrom<u256>>::try_from(u256_1);
```

### Convert to `Bytes`

```sway
use std::{bytes::Bytes, bytes_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*}};
```

```sway
use std::{bytes::Bytes, bytes_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*}};
    let num = 5;
    let little_endian_bytes: Bytes = num.to_le_bytes();
    let big_endian_bytes: Bytes = num.to_be_bytes();
```

### Convert from `Bytes`

```sway
use std::{bytes::Bytes, bytes_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*}};
```

```sway
    let u16_from_le_bytes: u16 = u16::from_le_bytes(little_endian_bytes);
    let u16_from_be_bytes: u16 = u16::from_be_bytes(big_endian_bytes);

    let u32_from_le_bytes: u32 = u32::from_le_bytes(little_endian_bytes);
    let u32_from_be_bytes: u32 = u32::from_be_bytes(big_endian_bytes);

    let u64_from_le_bytes: u64 = u64::from_le_bytes(little_endian_bytes);
    let u64_from_be_bytes: u64 = u64::from_be_bytes(big_endian_bytes);

    let u256_from_le_bytes = u256::from_le_bytes(little_endian_bytes);
    let u256_from_be_bytes = u256::from_be_bytes(big_endian_bytes);

    let b256_from_le_bytes = b256::from_le_bytes(little_endian_bytes);
    let b256_from_be_bytes = b256::from_be_bytes(big_endian_bytes);
```

## Byte Array Conversions

### Convert to a Byte Array

```sway
use std::array_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*};
```

```sway
use std::array_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*};
    let u16_1: u16 = 2u16;
    let u32_1: u32 = 2u32;
    let u64_1: u64 = 2u64;
    let u256_1: u256 = 0x0000000000000000000000000000000000000000000000000000000000000002u256;
    let b256_1: b256 = 0x000000000000000000000000000000000000000000000000000000000000002A;
    // little endian
    let le_byte_array_from_u16: [u8; 2] = u16_1.to_le_bytes();
    let le_byte_array_from_u32: [u8; 4] = u32_1.to_le_bytes();
    let le_byte_array_from_u64: [u8; 8] = u64_1.to_le_bytes();
    let le_byte_array_from_u256: [u8; 32] = u256_1.to_le_bytes();
    let le_byte_array_from_b256: [u8; 32] = b256_1.to_le_bytes();
    // big endian
    let be_byte_array_from_u16: [u8; 2] = u16_1.to_be_bytes();
    let be_byte_array_from_u32: [u8; 4] = u32_1.to_be_bytes();
    let be_byte_array_from_u64: [u8; 8] = u64_1.to_be_bytes();
    let be_byte_array_from_u256: [u8; 32] = u256_1.to_be_bytes();
    let be_byte_array_from_b256: [u8; 32] = b256_1.to_be_bytes();
```

### Convert from a Byte Array

```sway
use std::array_conversions::{b256::*, u16::*, u256::*, u32::*, u64::*};
```

```sway
    let u16_byte_array: [u8; 2] = [2_u8, 1_u8];
    let u32_byte_array: [u8; 4] = [4_u8, 3_u8, 2_u8, 1_u8];
    let u64_byte_array: [u8; 8] = [8_u8, 7_u8, 6_u8, 5_u8, 4_u8, 3_u8, 2_u8, 1_u8];
    let u256_byte_array: [u8; 32] = [
        32_u8, 31_u8, 30_u8, 29_u8, 28_u8, 27_u8, 26_u8, 25_u8, 24_u8, 23_u8, 22_u8,
        21_u8, 20_u8, 19_u8, 18_u8, 17_u8, 16_u8, 15_u8, 14_u8, 13_u8, 12_u8, 11_u8,
        10_u8, 9_u8, 8_u8, 7_u8, 6_u8, 5_u8, 4_u8, 3_u8, 2_u8, 1_u8,
    ];
    // little endian
    let le_u16_from_byte_array: u16 = u16::from_le_bytes(u16_byte_array);
    let le_u32_from_byte_array: u32 = u32::from_le_bytes(u32_byte_array);
    let le_u64_from_byte_array: u64 = u64::from_le_bytes(u64_byte_array);
    let le_u256_from_byte_array: u256 = u256::from_le_bytes(u256_byte_array);
    let le_b256_from_byte_array: b256 = b256::from_le_bytes(u256_byte_array);
    // big endian
    let be_u16_from_byte_array: u16 = u16::from_be_bytes(u16_byte_array);
    let be_u32_from_byte_array: u32 = u32::from_be_bytes(u32_byte_array);
    let be_u64_from_byte_array: u64 = u64::from_be_bytes(u64_byte_array);
    let be_u256_from_byte_array: u256 = u256::from_be_bytes(u256_byte_array);
    let be_b256_from_byte_array: b256 = b256::from_be_bytes(u256_byte_array);
```
