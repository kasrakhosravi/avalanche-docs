---
tags: [Build, Virtual Machines]
description: In this guide, we'll define the precompile by implementing the `HelloWorld` interface.
sidebar_label: Defining Your Precompile
pagination_label: Defining Your Precompile
sidebar_position: 3
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Defining Your Precompile

Now that we have autogenerated the template code required for our precompile, let's actually write the logic for the precompile itself.

## Setting Config Key

Let's jump to `helloworld/module.go` file first. This file contains the module definition for our
precompile. You can see the `ConfigKey` is set to some default value of `helloWorldConfig`.
This key should be unique to the precompile.
This config key determines which JSON key to use when reading the precompile's config from the
JSON upgrade/genesis file. In this case, the config key is `helloWorldConfig` and the JSON config
should look like this:

```json
{
  "helloWorldConfig": {
    "blockTimestamp": 0
		...
  }
}
```

## Setting Contract Address

In the `helloworld/module.go` you can see the `ContractAddress` is set to some default value.
This should be changed to a suitable address for your precompile.
The address should be unique to the precompile. There is a registry of precompile addresses
under [`precompile/registry/registry.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/registry/registry.go).
A list of addresses is specified in the comments under this file.
Modify the default value to be the next user available stateful precompile address. For forks of
Subnet-EVM or Precompile-EVM, users should start at `0x0300000000000000000000000000000000000000` to ensure
that their own modifications do not conflict with stateful precompiles that may be added to
Subnet-EVM in the future. You should pick an address that is not already taken.

```go
// This list is kept just for reference. The actual addresses defined in respective packages of precompiles.
// Note: it is important that none of these addresses conflict with each other or any other precompiles
// in core/vm/contracts.go.
// The first stateful precompiles were added in coreth to support nativeAssetCall and nativeAssetBalance. New stateful precompiles
// originating in coreth will continue at this prefix, so we reserve this range in subnet-evm so that they can be migrated into
// subnet-evm without issue.
// These start at the address: 0x0100000000000000000000000000000000000000 and will increment by 1.
// Optional precompiles implemented in subnet-evm start at 0x0200000000000000000000000000000000000000 and will increment by 1
// from here to reduce the risk of conflicts.
// For forks of subnet-evm, users should start at 0x0300000000000000000000000000000000000000 to ensure
// that their own modifications do not conflict with stateful precompiles that may be added to subnet-evm
// in the future.
// ContractDeployerAllowListAddress = common.HexToAddress("0x0200000000000000000000000000000000000000")
// ContractNativeMinterAddress      = common.HexToAddress("0x0200000000000000000000000000000000000001")
// TxAllowListAddress               = common.HexToAddress("0x0200000000000000000000000000000000000002")
// FeeManagerAddress                = common.HexToAddress("0x0200000000000000000000000000000000000003")
// RewardManagerAddress             = common.HexToAddress("0x0200000000000000000000000000000000000004")
// HelloWorldAddress                = common.HexToAddress("0x0300000000000000000000000000000000000000")
// ADD YOUR PRECOMPILE HERE
// {YourPrecompile}Address          = common.HexToAddress("0x03000000000000000000000000000000000000??")
```

Don't forget to update the actual variable `ContractAddress` in `module.go` to the address you chose.
It should look like this:

```go
// ContractAddress is the defined address of the precompile contract.
// This should be unique across all precompile contracts.
// See params/precompile_modules.go for registered precompile contracts and more information.
var ContractAddress = common.HexToAddress("0x0300000000000000000000000000000000000000")
```

Now when Subnet-EVM sees the `helloworld.ContractAddress` as input when executing
[`CALL`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/core/vm/evm.go#L284),
[`CALLCODE`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/core/vm/evm.go#L355),
[`DELEGATECALL`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/core/vm/evm.go#L396),
[`STATICCALL`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/core/vm/evm.go#L445),
it can run the precompile if the precompile is enabled.

## Adding Custom Code

Search (`CTRL F`) throughout the file with `CUSTOM CODE STARTS HERE` to find the areas in the
precompile package that you need to modify. You should start with the reference imports code block.

#### Module File

The module file contains fundamental information about the precompile. This includes the key for the
precompile, the address of the precompile, and a configurator. This file is located at
[`./precompile/helloworld/module.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/helloworld/module.go)
for Subnet-EVM and
[./helloworld/module.go](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/helloworld/module.go)
for Precompile-EVM.

This file defines the module for the precompile. The module is used to register the precompile to the
precompile registry. The precompile registry is used to read configs and enable the precompile.
Registration is done in the `init()` function of the module file. `MakeConfig()` is used to create a
new instance for the precompile config. This will be used in custom Unmarshal/Marshal logic.
You don't need to override these functions.

##### Configure()

Module file contains a `configurator` which implements the `contract.Configurator` interface. This interface
includes a `Configure()` function used to configure the precompile and set the initial
state of the precompile. This function is called when the precompile is enabled. This is typically used
to read from a given config in upgrade/genesis JSON and sets the initial state of the
precompile accordingly. This function also calls `AllowListConfig.Configure()` to invoke AllowList
configuration as the last step. You should keep it as it is if you want to use AllowList.
You can modify this function for your custom logic. You can circle back to this function later
after you have finalized the implementation of the precompile config.

#### Config File

The config file contains the config for the precompile. This file is located at
[`./precompile/helloworld/config.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/helloworld/config.go)
for Subnet-EVM and
[./helloworld/config.go](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/helloworld/config.go)
for Precompile-EVM.
This file contains the `Config` struct, which implements `precompileconfig.Config` interface.
It has some embedded structs like `precompileconfig.Upgrade`. `Upgrade` is used to enable
upgrades for the precompile. It contains the `BlockTimestamp` and `Disable` to enable/disable
upgrades. `BlockTimestamp` is the timestamp of the block when the upgrade will be activated.
`Disable` is used to disable the upgrade. If you use `AllowList` for the precompile, there is also
`allowlist.AllowListConfig` embedded in the `Config` struct. `AllowListConfig` is used to specify initial
roles for specified addresses. If you have any custom fields in your precompile config, you can add them
here. These custom fields will be read from upgrade/genesis JSON and set in the precompile config.

```go
// Config implements the precompileconfig.Config interface and
// adds specific configuration for HelloWorld.
type Config struct {
	allowlist.AllowListConfig
	precompileconfig.Upgrade
}
```

##### Verify()

`Verify()` is called on startup and an error is treated as fatal. Generated code contains a call
to `AllowListConfig.Verify()` to verify the `AllowListConfig`. You can leave that as is and start
adding your own custom verify code after that.

We can leave this function as is right now because there is no invalid custom configuration for the `Config`.

```go
// Verify tries to verify Config and returns an error accordingly.
func (c *Config) Verify() error {
	// Verify AllowList first
	if err := c.AllowListConfig.Verify(); err != nil {
		return err
	}

	// CUSTOM CODE STARTS HERE
	// Add your own custom verify code for Config here
	// and return an error accordingly
	return nil
}
```

##### Equal()

Next, we see is `Equal()`. This function determines if two precompile configs are equal. This is used
to determine if the precompile needs to be upgraded. There is some default code that is generated for
checking `Upgrade` and `AllowListConfig` equality.

<!-- markdownlint-disable MD013 -->

```go
// Equal returns true if [s] is a [*Config] and it has been configured identical to [c].
func (c *Config) Equal(s precompileconfig.Config) bool {
	// typecast before comparison
	other, ok := (s).(*Config)
	if !ok {
		return false
	}
	// CUSTOM CODE STARTS HERE
	// modify this boolean accordingly with your custom Config, to check if [other] and the current [c] are equal
	// if Config contains only Upgrade  and AllowListConfig  you can skip modifying it.
	equals := c.Upgrade.Equal(&other.Upgrade) && c.AllowListConfig.Equal(&other.AllowListConfig)
	return equals
}
```

<!-- markdownlint-enable MD013 -->

We can leave this function as is since we check `Upgrade` and `AllowListConfig` for equality which are
the only fields that `Config` struct has.

#### Modify Configure()

We can now circle back to `Configure()` in `module.go` as we finished implementing `Config` struct.
This function configures the `state` with the
initial configuration at`blockTimestamp` when the precompile is enabled.
In the HelloWorld example, we want to set up a default
key-value mapping in the state where the key is `storageKey` and the value is `Hello World!`. The
`StateDB` allows us to store a key-value mapping of 32-byte hashes. The below code snippet can be
copied and pasted to overwrite the default `Configure()` code.

```go
const defaultGreeting = "Hello World!"

// Configure configures [state] with the given [cfg] precompileconfig.
// This function is called by the EVM once per precompile contract activation.
// You can use this function to set up your precompile contract's initial state,
// by using the [cfg] config and [state] stateDB.
func (*configurator) Configure(chainConfig contract.ChainConfig, cfg precompileconfig.Config, state contract.StateDB, _ contract.BlockContext) error {
	config, ok := cfg.(*Config)
	if !ok {
		return fmt.Errorf("incorrect config %T: %v", config, config)
	}
	// CUSTOM CODE STARTS HERE

	// This will be called in the first block where HelloWorld stateful precompile is enabled.
	// 1) If BlockTimestamp is nil, this will not be called
	// 2) If BlockTimestamp is 0, this will be called while setting up the genesis block
	// 3) If BlockTimestamp is 1000, this will be called while processing the first block
	// whose timestamp is >= 1000
	//
	// Set the initial value under [common.BytesToHash([]byte("storageKey")] to "Hello World!"
	StoreGreeting(state, defaultGreeting)
	// AllowList is activated for this precompile. Configuring allowlist addresses here.
	return config.AllowListConfig.Configure(state, ContractAddress)
}
```

#### Contract File

The contract file contains the functions of the precompile contract that will be called by the EVM. The
file is located at [`./precompile/helloworld/contract.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/helloworld/contract.go)
for Subnet-EVM and
[./helloworld/contract.go](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/helloworld/contract.go)
for Precompile-EVM.
Since we use `IAllowList` interface there will be auto-generated code for `AllowList`
functions like below:

```go
// GetHelloWorldAllowListStatus returns the role of [address] for the HelloWorld list.
func GetHelloWorldAllowListStatus(stateDB contract.StateDB, address common.Address) allowlist.Role {
	return allowlist.GetAllowListStatus(stateDB, ContractAddress, address)
}

// SetHelloWorldAllowListStatus sets the permissions of [address] to [role] for the
// HelloWorld list. Assumes [role] has already been verified as valid.
// This stores the [role] in the contract storage with address [ContractAddress]
// and [address] hash. It means that any reusage of the [address] key for different value
// conflicts with the same slot [role] is stored.
// Precompile implementations must use a different key than [address] for their storage.
func SetHelloWorldAllowListStatus(stateDB contract.StateDB, address common.Address, role allowlist.Role) {
	allowlist.SetAllowListRole(stateDB, ContractAddress, address, role)
}
```

These will be helpful to use AllowList precompile helper in our functions.

##### Packers and Unpackers

There are also auto-generated Packers and Unpackers for the ABI. These will be used in `sayHello` and
`setGreeting` functions to comfort the ABI.
These functions are auto-generated
and will be used in necessary places accordingly.
You don't need to worry about how to deal with them, but it's good to know what they are.

Each input to a precompile contract function has it's own `Unpacker` function as follows:

```go
// UnpackSetGreetingInput attempts to unpack [input] into the string type argument
// assumes that [input] does not include selector (omits first 4 func signature bytes)
func UnpackSetGreetingInput(input []byte) (string, error) {
	res, err := HelloWorldABI.UnpackInput("setGreeting", input)
	if err != nil {
		return "", err
	}
	unpacked := *abi.ConvertType(res[0], new(string)).(*string)
	return unpacked, nil
}
```

The ABI is a binary format and the input to the precompile contract function is a
byte array. The `Unpacker` function converts this input to a more easy-to-use format so that we can
use it in our function.

Similarly, there is a `Packer` function for each output of a precompile contract function as follows:

```go
// PackSayHelloOutput attempts to pack given result of type string
// to conform the ABI outputs.
func PackSayHelloOutput(result string) ([]byte, error) {
	return HelloWorldABI.PackOutput("sayHello", result)
}
```

This function converts the output of the function to a byte array that conforms to the ABI and can be
returned to the EVM as a result.

##### Modify sayHello()

The next place to modify is in our `sayHello()` function. In a previous step, we created the `IHelloWorld.sol`
interface with two functions `sayHello()` and `setGreeting()`. We finally get to implement them here.
If any contract calls these functions from the interface, the below function gets executed. This function
is a simple getter function. In `Configure()` we set up a mapping with the key as `storageKey` and
the value as `Hello World!` In this function, we will be returning whatever value is at `storageKey`.
The below code snippet can be copied and pasted to overwrite the default `setGreeting` code.

First, we add a helper function to get the greeting value from the stateDB, this will be helpful
when we test our contract.

```go
// GetGreeting returns the value of the storage key "storageKey" in the contract storage,
// with leading zeroes trimmed.
// This function is mostly used for tests.
func GetGreeting(stateDB contract.StateDB) string {
	// Get the value set at recipient
	value := stateDB.GetState(ContractAddress, storageKeyHash)
	return string(common.TrimLeftZeroes(value.Bytes()))
}
```

Now we can modify the `sayHello` function to return the stored value.

<!-- markdownlint-disable MD013 -->

```go
func sayHello(accessibleState contract.AccessibleState, caller common.Address, addr common.Address, input []byte, suppliedGas uint64, readOnly bool) (ret []byte, remainingGas uint64, err error) {
	if remainingGas, err = contract.deductGas(suppliedGas, SayHelloGasCost); err != nil {
		return nil, 0, err
	}
	// CUSTOM CODE STARTS HERE

	// Get the current state
	currentState := accessibleState.GetStateDB()
	// Get the value set at recipient
	value := GetGreeting(currentState)
	packedOutput, err := PackSayHelloOutput(value)
	if err != nil {
		return nil, remainingGas, err
	}

	// Return the packed output and the remaining gas
	return packedOutput, remainingGas, nil
}
```

<!-- markdownlint-enable MD013 -->

##### Modify setGreeting()

We can also modify our `setGreeting()` function. This is a simple setter function. It takes in `input`
and we will set that as the value in the state mapping with the key as `storageKey`. It also checks
if the VM running the precompile is in read-only mode. If it is, it returns an error.

There is also a generated `AllowList` code in that function. This generated code checks if the caller
address is eligible to perform this state-changing operation. If not, it returns an error.

Let's add the helper function to set the greeting value in the stateDB, this will be helpful
when we test our contract.

```go
// StoreGreeting sets the value of the storage key "storageKey" in the contract storage.
func StoreGreeting(stateDB contract.StateDB, input string) {
	inputPadded := common.LeftPadBytes([]byte(input), common.HashLength)
	inputHash := common.BytesToHash(inputPadded)

	stateDB.SetState(ContractAddress, storageKeyHash, inputHash)
}
```

The below code snippet can be copied and pasted to overwrite the default `setGreeting()` code.

<!-- markdownlint-disable MD013 -->

```go
func setGreeting(accessibleState contract.AccessibleState, caller common.Address, addr common.Address, input []byte, suppliedGas uint64, readOnly bool) (ret []byte, remainingGas uint64, err error) {
	if remainingGas, err = contract.DeductGas(suppliedGas, SetGreetingGasCost); err != nil {
		return nil, 0, err
	}
	if readOnly {
		return nil, remainingGas, vmerrs.ErrWriteProtection
	}
	// attempts to unpack [input] into the arguments to the SetGreetingInput.
	// Assumes that [input] does not include selector
	// You can use unpacked [inputStruct] variable in your code
	inputStruct, err := UnpackSetGreetingInput(input)
	if err != nil {
		return nil, remainingGas, err
	}

	// Allow list is enabled and SetGreeting is a state-changer function.
	// This part of the code restricts the function to be called only by enabled/admin addresses in the allow list.
	// You can modify/delete this code if you don't want this function to be restricted by the allow list.
	stateDB := accessibleState.GetStateDB()
	// Verify that the caller is in the allow list and therefore has the right to call this function.
	callerStatus := allowlist.GetAllowListStatus(stateDB, ContractAddress, caller)
	if !callerStatus.IsEnabled() {
		return nil, remainingGas, fmt.Errorf("%w: %s", ErrCannotSetGreeting, caller)
	}
	// allow list code ends here.

	// CUSTOM CODE STARTS HERE
	// Check if the input string is longer than HashLength
	if len(inputStruct) > common.HashLength {
		return nil, 0, ErrInputExceedsLimit
	}

	// setGreeting is the execution function
	// "SetGreeting(name string)" and sets the storageKey
	// in the string returned by hello world
	StoreGreeting(stateDB, inputStruct)

	// This function does not return an output, leave this one as is
	packedOutput := []byte{}

	// Return the packed output and the remaining gas
	return packedOutput, remainingGas, nil
}
```

<!-- markdownlint-enable MD013 -->

### Setting Gas Costs

Setting gas costs for functions is very important and should be done carefully.
If the gas costs are set too low,
then functions can be abused and can cause DoS attacks.
If the gas costs are set too high, then the contract will be too expensive
to run.
Subnet-EVM has some predefined gas costs for write and read operations
in [`precompile/contract/utils.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contract/utils.go#L19-L20).
In order to provide a baseline for gas costs, we have set the following gas costs.

```go
// Gas costs for stateful precompiles
const (
	WriteGasCostPerSlot = 20_000
	ReadGasCostPerSlot  = 5_000
)
```

`WriteGasCostPerSlot` is the cost of one write such as modifying a state storage slot.

`ReadGasCostPerSlot` is the cost of reading a state storage slot.

This should be in your gas cost estimations based on how many times the precompile function does a
read or a write. For example, if the precompile modifies the state slot of its precompile address
twice then the gas cost for that function would be `40_000`. However, if the precompile does additional
operations and requires more computational power, then you should increase the gas costs accordingly.

On top of these gas costs, we also have to account for the gas costs of AllowList gas costs. These
are the gas costs of reading and writing permissions for addresses in AllowList. These are defined
under Subnet-EVM's [`precompile/allowlist/allowlist.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/allowlist/allowlist.go#L28-L29).
By default, these are added to the default gas costs of the state-change functions (SetGreeting)
of the precompile. Meaning that these functions will cost an additional `ReadAllowListGasCost` in order
to read permissions from the storage. If you don't plan to read permissions from the storage then
you can omit these.

Now going back to our `/helloworld/contract.go`, we can modify our precompile function gas costs.
Please search (`CTRL F`) `SET A GAS COST HERE` to locate the default gas cost code.

```go
SayHelloGasCost    uint64 = 0                                  // SET A GAS COST HERE
SetGreetingGasCost uint64 = 0 + allowlist.ReadAllowListGasCost // SET A GAS COST HERE
```

We get and set our greeting with `sayHello()` and `setGreeting()` in one slot
respectively so we can define the gas costs as follows. We also read permissions from the
AllowList in `setGreeting()` so we keep `allowlist.ReadAllowListGasCost`.

```go
SayHelloGasCost    uint64 = contract.ReadGasCostPerSlot
SetGreetingGasCost uint64 = contract.WriteGasCostPerSlot + allowlist.ReadAllowListGasCost
```

### Registering Your Precompile

We should register our precompile package to the Subnet-EVM to be discovered by other packages.
Our `Module` file contains an `init()` function that registers our precompile.
`init()` is called when the package is imported.
We should register our precompile in a common package so
that it can be imported by other packages.

<!-- vale off -->

<Tabs groupId="evm-tabs">

<TabItem value="subnet-evm-tab" label="Subnet-EVM" default>

For Subnet-EVM we have a precompile registry under [`/precompile/registry/registry.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/registry/registry.go).
This registry force-imports precompiles from other packages, for example:

```go
// Force imports of each precompile to ensure each precompile's init function runs and registers itself
// with the registry.
import (
	_ "github.com/ava-labs/subnet-evm/precompile/contracts/deployerallowlist"

	_ "github.com/ava-labs/subnet-evm/precompile/contracts/nativeminter"

	_ "github.com/ava-labs/subnet-evm/precompile/contracts/txallowlist"

	_ "github.com/ava-labs/subnet-evm/precompile/contracts/feemanager"

	_ "github.com/ava-labs/subnet-evm/precompile/contracts/rewardmanager"

	_ "github.com/ava-labs/subnet-evm/precompile/contracts/helloworld"
	// ADD YOUR PRECOMPILE HERE
	// _ "github.com/ava-labs/subnet-evm/precompile/contracts/yourprecompile"
)
```

<!-- vale off -->

The registry itself also force-imported by the [`/plugin/evm/vm.go](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/plugin/evm/vm.go#L50).
This ensures that the registry is imported and the precompiles are registered.

<!-- vale on -->

</TabItem>
<TabItem value="precompile-evm-tab" label="Precompile-EVM"  >

For Precompile-EVM there is a [`plugin/main.go`](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/plugin/main.go)
file in Precompile-EVM that orchestrates this precompile registration.

```go
// (c) 2019-2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package main

import (
	"fmt"

	"github.com/ava-labs/avalanchego/version"
	"github.com/ava-labs/subnet-evm/plugin/evm"
	"github.com/ava-labs/subnet-evm/plugin/runner"

	// Each precompile generated by the precompilegen tool has a self-registering init function
	// that registers the precompile with the subnet-evm. Importing the precompile package here
	// will cause the precompile to be registered with the subnet-evm.
	_ "github.com/ava-labs/precompile-evm/helloworld"
	// ADD YOUR PRECOMPILE HERE
	//_ "github.com/ava-labs/precompile-evm/{yourprecompilepkg}"
)
```

</TabItem>
</Tabs>

<!-- vale on -->
  375 changes: 375 additions & 0 deletions375  
docs/build/vm/evm/defining-tests.md
Viewed
@@ -0,0 +1,375 @@
---
tags: [Build, Virtual Machines]
description: Testing Your Stateful Precompile
sidebar_label: Writing Test Cases
pagination_label: Writing Test Cases
sidebar_position: 4
---

# Writing Test Cases

In this section, we will go over the different ways we can write test cases for our stateful precompile.

## Adding Config Tests

Precompile generation tool generates skeletons for unit tests as well. Generated config tests will
be under [`./precompile/contracts/helloworld/config_test.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/helloworld/config_test.go)
for Subnet-EVM and [`./helloworld/config_test.go`](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/helloworld/config_test.go)
for Precompile-EVM.
There are mainly two functions we need
to test: `Verify` and `Equal`. `Verify` checks if the precompile is configured correctly. `Equal`
checks if the precompile is equal to another precompile. Generated `Verify` tests contain a valid case.
You can add more invalid cases depending on your implementation. `Equal` tests generate some
invalid cases to test different timestamps, types, and AllowList cases.
You can check each `config_test.go` files for other precompiles
under the Subnet-EVM's [`./precompile/contracts`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/)
directory for more examples.

## Adding Contract Tests

The tool also generates contract tests to make sure our precompile is working correctly. Generated
tests include cases to test allow list capabilities, gas costs, and calling functions in read-only mode.
You can check other `contract_test.go` files in the `/precompile/contracts`. Hello World contract
tests will be under [`./precompile/contracts/helloworld/contract_test.go`](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/precompile/contracts/helloworld/contract_test.go)
for Subnet-EVM and
[`./helloworld/contract_test.go`](https://github.com/ava-labs/precompile-evm/blob/hello-world-example/helloworld/contract_test.go)
for Precompile-EVM.
We will also add more test to cover functionalities of `sayHello()` and `setGreeting()`.
Contract tests are defined in a standard structure that each test
can customize to their needs. The test structure is as follows:

```go
// PrecompileTest is a test case for a precompile
type PrecompileTest struct {
	// Caller is the address of the precompile caller
	Caller common.Address
	// Input the raw input bytes to the precompile
	Input []byte
	// InputFn is a function that returns the raw input bytes to the precompile
	// If specified, Input will be ignored.
	InputFn func(t *testing.T) []byte
	// SuppliedGas is the amount of gas supplied to the precompile
	SuppliedGas uint64
	// ReadOnly is whether the precompile should be called in read only
	// mode. If true, the precompile should not modify the state.
	ReadOnly bool
	// Config is the config to use for the precompile
	// It should be the same precompile config that is used in the
	// precompile's configurator.
	// If nil, Configure will not be called.
	Config precompileconfig.Config
	// BeforeHook is called before the precompile is called.
	BeforeHook func(t *testing.T, state contract.StateDB)
	// AfterHook is called after the precompile is called.
	AfterHook func(t *testing.T, state contract.StateDB)
	// ExpectedRes is the expected raw byte result returned by the precompile
	ExpectedRes []byte
	// ExpectedErr is the expected error returned by the precompile
	ExpectedErr string
	// BlockNumber is the block number to use for the precompile's block context
	BlockNumber int64
}
```

Each test can populate the fields of the `PrecompileTest` struct to customize the test.
This test uses an AllowList helper function
`allowlist.RunPrecompileWithAllowListTests(t, Module, state.NewTestStateDB, tests)`
which can run all specified tests plus AllowList test suites. If you don't plan to use AllowList,
you can directly run them as follows:

```go
	for name, test := range tests {
		t.Run(name, func(t *testing.T) {
			test.Run(t, module, newStateDB(t))
		})
	}
```

## Adding VM Tests (Optional)

This is only applicable for direct Subnet-EVM forks as test files are not directly exported in
Golang. If you use Precompile-EVM you can skip this step.

VM tests are tests that run the precompile by calling it through the Subnet-EVM. These are the most
comprehensive tests that we can run. If your precompile modifies how the Subnet-EVM works, for example
changing blockchain rules, you should add a VM test. For example, you can take a look at the
TestRewardManagerPrecompileSetRewardAddress function in [here](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/plugin/evm/vm_test.go#L2675).
For this Hello World example, we don't modify any Subnet-EVM rules, so we don't need to add any VM tests.

## Adding Solidity Test Contracts

Let's add our test contract to `./contracts/contracts`. This smart contract lets us interact
with our precompile! We cast the `HelloWorld` precompile address to the `IHelloWorld`interface. In
doing so, `helloWorld` is now a contract of type `IHelloWorld` and when we call any functions on
that contract, we will be redirected to the HelloWorld precompile address. The below code snippet
can be copied and pasted into a new file called `ExampleHelloWorld.sol`:

```sol
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "./IHelloWorld.sol";
// ExampleHelloWorld shows how the HelloWorld precompile can be used in a smart contract.
contract ExampleHelloWorld {
  address constant HELLO_WORLD_ADDRESS =
    0x0300000000000000000000000000000000000000;
  IHelloWorld helloWorld = IHelloWorld(HELLO_WORLD_ADDRESS);
  function sayHello() public view returns (string memory) {
    return helloWorld.sayHello();
  }
  function setGreeting(string calldata greeting) public {
    helloWorld.setGreeting(greeting);
  }
}
```

:::warning

Hello World Precompile is a different contract than ExampleHelloWorld and has a different address.
Since the precompile uses AllowList for a permissioned access,
any call to the precompile including from ExampleHelloWorld will be denied unless
the caller is added to the AllowList.

:::

Please note that this contract is simply a wrapper and is calling the precompile functions.
The reason why we add another example smart contract is to have a simpler stateless tests.

For the test contract we write our test in `./contracts/test/ExampleHelloWorldTest.sol`.

<!-- vale off -->
<!-- vale off -->

<Tabs groupId="evm-tabs">

<TabItem value="subnet-evm-tab" label="Subnet-EVM" default>

<!-- vale on -->

```sol
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "../ExampleHelloWorld.sol";
import "../interfaces/IHelloWorld.sol";
import "./AllowListTest.sol";
contract ExampleHelloWorldTest is AllowListTest {
  IHelloWorld helloWorld = IHelloWorld(HELLO_WORLD_ADDRESS);
  function step_getDefaultHelloWorld() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    assertEq(example.sayHello(), "Hello World!");
  }
  function step_doesNotSetGreetingBeforeEnabled() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    try example.setGreeting("testing") {
      assertTrue(false, "setGreeting should fail");
    } catch {}
  }
  function step_setAndGetGreeting() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    helloWorld.setEnabled(exampleAddress);
    assertRole(
      helloWorld.readAllowList(exampleAddress),
      AllowList.Role.Enabled
    );
    string memory greeting = "testgreeting";
    example.setGreeting(greeting);
    assertEq(example.sayHello(), greeting);
  }
}
```

</TabItem>
<TabItem value="precompile-evm-tab" label="Precompile-EVM"  >

For Precompile-EVM, you should import `AllowListTest` with `@avalabs/subnet-evm-contracts` NPM package:

```sol
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "../ExampleHelloWorld.sol";
import "../interfaces/IHelloWorld.sol";
import "@avalabs/subnet-evm-contracts/contracts/test/AllowListTest.sol";
contract ExampleHelloWorldTest is AllowListTest {
  IHelloWorld helloWorld = IHelloWorld(HELLO_WORLD_ADDRESS);
  function step_getDefaultHelloWorld() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    assertEq(example.sayHello(), "Hello World!");
  }
  function step_doesNotSetGreetingBeforeEnabled() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    try example.setGreeting("testing") {
      assertTrue(false, "setGreeting should fail");
    } catch {}
  }
  function step_setAndGetGreeting() public {
    ExampleHelloWorld example = new ExampleHelloWorld();
    address exampleAddress = address(example);
    assertRole(helloWorld.readAllowList(exampleAddress), AllowList.Role.None);
    helloWorld.setEnabled(exampleAddress);
    assertRole(
      helloWorld.readAllowList(exampleAddress),
      AllowList.Role.Enabled
    );
    string memory greeting = "testgreeting";
    example.setGreeting(greeting);
    assertEq(example.sayHello(), greeting);
  }
}
```

</TabItem>
</Tabs>

<!-- vale on -->

## Adding DS-Test Case

We can now trigger this test contract via `hardhat` tests. The test script uses Subnet-EVM's `test`
framework test in `./contracts/test`.
You can find more information about the test framework [here](https://github.com/ava-labs/subnet-evm/blob/helloworld-official-tutorial-v2/contracts/test/utils.ts).

<!-- vale off -->

<Tabs groupId="evm-tabs">

<TabItem value="subnet-evm-tab" label="Subnet-EVM" default>

The test script looks like this:

```ts
// (c) 2019-2022, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

import { ethers } from "hardhat";
import { test } from "./utils";

// make sure this is always an admin for hello world precompile
const ADMIN_ADDRESS = "0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC";
const HELLO_WORLD_ADDRESS = "0x0300000000000000000000000000000000000000";

describe("ExampleHelloWorldTest", function () {
  this.timeout("30s");

  beforeEach("Setup DS-Test contract", async function () {
    const signer = await ethers.getSigner(ADMIN_ADDRESS);
    const helloWorldPromise = ethers.getContractAt(
      "IHelloWorld",
      HELLO_WORLD_ADDRESS,
      signer
    );

    return ethers
      .getContractFactory("ExampleHelloWorldTest", { signer })
      .then((factory) => factory.deploy())
      .then((contract) => {
        this.testContract = contract;
        return contract.deployed().then(() => contract);
      })
      .then(() => Promise.all([helloWorldPromise]))
      .then(([helloWorld]) => helloWorld.setAdmin(this.testContract.address))
      .then((tx) => tx.wait());
  });

  test("should gets default hello world", ["step_getDefaultHelloWorld"]);

  test(
    "should not set greeting before enabled",
    "step_doesNotSetGreetingBeforeEnabled"
  );

  test(
    "should set and get greeting with enabled account",
    "step_setAndGetGreeting"
  );
});
```

</TabItem>
<TabItem value="precompile-evm-tab" label="Precompile-EVM"  >
The test script looks like this:

```ts
// (c) 2019-2022, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

import { ethers } from "hardhat";
import { test } from "@avalabs/subnet-evm-contracts";

// make sure this is always an admin for hello world precompile
const ADMIN_ADDRESS = "0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC";
const HELLO_WORLD_ADDRESS = "0x0300000000000000000000000000000000000000";

describe("ExampleHelloWorldTest", function () {
  this.timeout("30s");

  beforeEach("Setup DS-Test contract", async function () {
    const signer = await ethers.getSigner(ADMIN_ADDRESS);
    const helloWorldPromise = ethers.getContractAt(
      "IHelloWorld",
      HELLO_WORLD_ADDRESS,
      signer
    );

    return ethers
      .getContractFactory("ExampleHelloWorldTest", { signer })
      .then((factory) => factory.deploy())
      .then((contract) => {
        this.testContract = contract;
        return contract.deployed().then(() => contract);
      })
      .then(() => Promise.all([helloWorldPromise]))
      .then(([helloWorld]) => helloWorld.setAdmin(this.testContract.address))
      .then((tx) => tx.wait());
  });

  test("should gets default hello world", ["step_getDefaultHelloWorld"]);

  test(
    "should not set greeting before enabled",
    "step_doesNotSetGreetingBeforeEnabled"
  );

  test(
    "should set and get greeting with enabled account",
    "step_setAndGetGreeting"
  );
});
```

</TabItem>
</Tabs>

<!-- vale on -->