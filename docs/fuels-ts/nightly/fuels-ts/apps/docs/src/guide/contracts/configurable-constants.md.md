# Configurable Constants

Sway introduces a powerful feature: configurable constants. When creating a contract, you can define constants, each assigned with a default value.

Before deploying the contract, you can then redefine the value for these constants, it can be all of them or as many as you need.

This feature provides flexibility for dynamic contract environments. It allows a high level of customization, leading to more efficient and adaptable smart contracts.

## Defining Configurable Constants

Below is an example of a contract in which we declare four configurable constants:

```
contract;

enum MyEnum {
    Checked: (),
    Pending: (),
}

struct MyStruct {
    x: u8,
    y: u8,
    state: MyEnum,
}

configurable {
    age: u8 = 25,
    tag: str[4] = __to_str_array("fuel"),
    grades: [u8; 4] = [3, 4, 3, 2],
    my_struct: MyStruct = MyStruct {
        x: 1,
        y: 2,
        state: MyEnum::Pending,
    },
}

abi EchoConfigurables {
    fn echo_configurables() -> (u8, str[4], [u8; 4], MyStruct);
}

impl EchoConfigurables for Contract {
    fn echo_configurables() -> (u8, str[4], [u8; 4], MyStruct) {
        (age, tag, grades, my_struct)
    }
}
```

In this contract, the function `echo_configurables` returns the values of the configurable constants, which we'll use for demonstrating the setting of configurables via the SDK.

## Setting New Values For Configurable Constants

During contract deployment, you can define new values for any/all of the configurable constants. The example below shows setting of one configurable constant, while the others will have default values.

```
const configurableConstants = {
  age: 10,
};

const deploy = await EchoConfigurablesFactory.deploy(wallet, {
  configurableConstants,
});
const { contract } = await deploy.waitForResult();

const {
  value: [age, tag, grades, myStruct],
} = await contract.functions.echo_configurables().get();

// age got updated
console.log('age', age); // 10
// while the rest are default values
console.log('tag', tag); // 'fuel'
console.log('grades', grades); // [3, 4, 3, 2]
console.log('myStruct', myStruct); // { x: 1, y: 2, state: 'Pending' }
```

Please note that when assigning new values for a `Struct`, all properties of the `Struct` must be defined. Failing to do so will result in an error:

```
const invalidConfigurables = {
  my_struct: {
    x: 10,
  },
};
try {
  await EchoConfigurablesFactory.deploy(wallet, {
    configurableConstants: invalidConfigurables,
  });
} catch (e) {
  console.log('error', e);
  // error: Error setting configurable constants on contract:
  // Invalid struct MyStruct. Field "y" not present.
}
```
