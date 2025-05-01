# `RawSlice`

A dynamic array of values can be represented using the `RawSlice` type. A raw slice can be a value reference or a raw pointer.

## Using a `RawSlice`

The `RawSlice` type can be integrated with your contract calls. Consider the following contract that can compare and return a `RawSlice`:

```
contract;

abi RawSliceTest {
    fn echo_raw_slice(value: raw_slice) -> raw_slice;
    fn raw_slice_comparison(value: raw_slice) -> bool;
}

impl RawSliceTest for Contract {
    fn echo_raw_slice(value: raw_slice) -> raw_slice {
        value
    }

    fn raw_slice_comparison(value: raw_slice) -> bool {
        let vec: Vec<u8> = Vec::from(value);

        vec.len() == 3 && vec.get(0).unwrap() == 40 && vec.get(1).unwrap() == 41 && vec.get(2).unwrap() == 42
    }
}
```

A `RawSlice` can be created using a native JavaScript array of numbers or Big Numbers, and sent to a Sway contract:

```
const rawSlice: RawSlice = [8, 42, 77];

const { value } = await contract.functions.echo_raw_slice(rawSlice).get();
```
