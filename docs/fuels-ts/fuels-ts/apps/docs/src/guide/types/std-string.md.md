# `StdString`

A dynamic string of variable length can be represented using the `StdString` type, also known as a Standard Lib String or `std-lib-string`. It behaves much like a dynamic string in most languages, and is essentially an array of characters.

## Using a `StdString`

The `StdString` type can be integrated with your contract calls. Consider the following contract that can compare and return a String:

```
contract;

use std::string::String;

abi StdStringTest {
    fn echo_string(value: String) -> String;
    fn string_comparison(value: String) -> bool;
}

impl StdStringTest for Contract {
    fn echo_string(value: String) -> String {
        value
    }

    fn string_comparison(value: String) -> bool {
        let expected = String::from_ascii_str("Hello World");

        value.as_bytes() == expected.as_bytes()
    }
}
```

A string can be created using a native JavaScript string, and sent to a Sway contract:

```
const stdString: StdString = 'Hello Fuel';

const { value } = await contract.functions.echo_string(stdString).get();

console.log('value', value);
// 'Hello Fuel'
```
