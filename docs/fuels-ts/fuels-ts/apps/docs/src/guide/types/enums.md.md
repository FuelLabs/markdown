# Enums

Sway Enums are a little distinct from TypeScript Enums. In this document, we will explore how you can represent Sway Enums in the SDK and how to use them with Sway contract functions.

## Basic Sway Enum Example

Consider the following basic Sway Enum called `StateError`:

```
// #region enums-4
pub enum StateError {
    Void: (),
    Pending: (),
    Completed: (),
}
```

The type `()` indicates that there is no additional data associated with each Enum variant. Sway allows you to create Enums of Enums or associate types with Enum variants.

### Using Sway Enums As Function Parameters

Let's define a Sway contract function that takes a `StateError` Enum variant as an argument and returns it:

```
    fn echo_state_error_enum(state_error: StateError) -> StateError {
        state_error
    }
```

To execute the contract function and validate the response, we can use the following code:

```
import { Provider, Wallet } from 'fuels';

import { LOCAL_NETWORK_URL, WALLET_PVT_KEY } from '../../../../env';
import { EchoEnumFactory } from '../../../../typegend';
import { StateErrorInput } from '../../../../typegend/contracts/EchoEnum';

const provider = new Provider(LOCAL_NETWORK_URL);
const wallet = Wallet.fromPrivateKey(WALLET_PVT_KEY, provider);
const deploy = await EchoEnumFactory.deploy(wallet);
const { contract } = await deploy.waitForResult();

const enumParam = StateErrorInput.Completed;

const { value } = await contract.functions
  .echo_state_error_enum(enumParam)
  .get();

console.log('value', value);
// StateErrorInput.Completed
```

In this example, we simply pass the Enum variant as a value to execute the contract function call.

## Enum of Enums Example

In this example, the `Error` Enum is an Enum of two other Enums: `StateError` and `UserError`.

```
pub enum StateError {
    Void: (),
    Pending: (),
    Completed: (),
}
// #endregion enums-1

pub enum UserError {
    Unauthorized: (),
    InsufficientPermissions: (),
}

pub enum Error {
    StateError: StateError,
    UserError: UserError,
}
```

### Using Enums of Enums with Contract Functions

Now, let's create a Sway contract function that accepts any variant of the `Error` Enum as a parameter and returns it immediately. This variant could be from either the `StateError` or `UserError` Enums.

```
    fn echo_error_enum(error: Error) -> Error {
        error
    }
```

Since the `Error` Enum is an Enum of Enums, we need to pass the function parameter differently. The parameter will be a TypeScript object:

```
const enumParam = { UserError: UserErrorInput.InsufficientPermissions };

const { value } = await contract.functions.echo_error_enum(enumParam).get();

console.log('value', value);
// { UserError: UserErrorInput.InsufficientPermissions }
```

In this case, since the variant `InsufficientPermissions` belongs to the `UserError` Enum, we create a TypeScript object using the Enum name as the object key and the variant as the object value.

We would follow the same approach if we intended to use a variant from the `StateError` Enum:

```
const enumParam = { StateError: StateErrorInput.Completed };

const { value } = await contract.functions.echo_error_enum(enumParam).get();

console.log('value', value);
// { StateError: StateErrorInput.Completed }
```

## Errors

While working with enums, you may run into the following issues:

### Using an invalid enum type

Thrown when the type being passed to the enum does not match that expected by it.

```
// Valid types: string
const emumParam = 1;

try {
  // @ts-expect-error number is not a valid type
  await contract.functions.echo_state_error_enum(emumParam).get();
} catch (error) {
  console.log('error', error);
}
```

### Using an invalid enum value

Thrown when the parameter passed is not an expected enum value.

```
// Valid values: 'Void', 'Pending', 'Completed'
const invalidEnumValue = 'NotStateEnumValue';

try {
  // @ts-expect-error NotStateEnumValue is not a valid value
  await contract.functions.echo_state_error_enum(invalidEnumValue).get();
} catch (error) {
  console.log('error', error);
}
```

### Using an invalid enum case key

Thrown when the passed enum case is not an expected enum case value.

```
// Valid case keys: 'StateError', 'UserError'
const enumParam = { UnknownKey: 'Completed' };

try {
  // @ts-expect-error UnknownKey is not a valid key
  await contract.functions.echo_error_enum(enumParam).get();
} catch (error) {
  console.log('error', error);
}
```
