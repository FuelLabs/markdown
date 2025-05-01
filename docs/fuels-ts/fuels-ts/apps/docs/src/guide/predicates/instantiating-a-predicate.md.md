# Instantiating predicates

A predicate in Sway can be as simple as the following:

```
predicate;

fn main() -> bool {
    true
}
```

In this minimal example, the `main` function does not accept any parameters and simply returns true.

Just like contracts in Sway, once you've created a predicate, you can compile it using `forc build`. For more information on working with Sway, refer to the <a :href="introUrl" target="_blank" rel="noreferrer">Sway documentation</a>.

After compiling, you will obtain the binary of the predicate and its JSON ABI (Application Binary Interface). Using these, you can instantiate a predicate in TypeScript as shown in the code snippet below:

```
import { Provider } from 'fuels';

import { LOCAL_NETWORK_URL } from '../../../../env';
import { ReturnTruePredicate } from '../../../../typegend';

const provider = new Provider(LOCAL_NETWORK_URL);

const predicate = new ReturnTruePredicate({
  provider,
});
```

The created [`Predicate`](DOCS_API_URL/classes/_fuel_ts_account.Predicate.html) instance, among other things, has three important properties: the predicate `bytes` (byte code), the `chainId`, and the predicate `address`.

This address, generated from the byte code, corresponds to the Pay-to-Script-Hash (P2SH) address used in Bitcoin.

## Predicate with multiple arguments

You can pass more than one argument to a predicate. For example, this is a predicate that evaluates to `true` if the two arguments are not equal:

```
predicate;

fn main(arg1: u64, arg2: u64) -> bool {
    return arg1 != arg2;
}
```

You can pass the two arguments to this predicate like this:

```
import { Provider } from 'fuels';

import { LOCAL_NETWORK_URL } from '../../../../env';
import { PredicateMultiArgs } from '../../../../typegend';

const provider = new Provider(LOCAL_NETWORK_URL);

const predicate = new PredicateMultiArgs({ provider, data: [20, 30] });
```

## Predicate with a Struct argument

You can also pass a struct as an argument to a predicate. This is one such predicate that expects a struct as an argument:

```
predicate;

struct Validation {
    has_account: bool,
    total_complete: u64,
}

fn main(received: Validation) -> bool {
    let expected_has_account: bool = true;
    let expected_total_complete: u64 = 100;

    received.has_account == expected_has_account && received.total_complete == expected_total_complete
}
```

You can pass a struct as an argument to this predicate like this:

```
import { Provider } from 'fuels';

import { LOCAL_NETWORK_URL } from '../../../../env';
import { PredicateMainArgsStruct } from '../../../../typegend';

const provider = new Provider(LOCAL_NETWORK_URL);

const predicate = new PredicateMainArgsStruct({
  provider,
  data: [{ has_account: true, total_complete: 100 }],
});
```
