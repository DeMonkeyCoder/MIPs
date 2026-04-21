# MIP Proposal: Mina Provider API

This proposal introduces a standardized Provider API for Mina Protocol wallets, account providers, and public providers. The goal is to establish a consistent interface for zkApp developers to interact with user accounts, inspired by the widely adopted EIP-1193 and EIP-1474 standards in Ethereum.

The specification in this proposal is intentionally brief, including only the necessary methods and events to facilitate easy adoption. Once current Mina wallets adopt this specification, more elaborate extensions can be designed in the future.

Auro Wallet currently uses an object format for `params` instead of the array format. While the object format is valid according to the JSON-RPC 2.0 Specification, this proposal uses the array format because it aligns with the conventions of other major blockchains, making it easier for multichain wallets to support.

# Motivation

Currently, major Mina wallets such as Auro Wallet and Pallad expose differing JavaScript Provider APIs, requiring zkApp developers to maintain separate integration code for each wallet. This fragmentation increases development complexity, testing overhead, and the risk of bugs.

Furthermore, without a defined standard, new wallet or account provider developers may introduce additional incompatible APIs, worsening the issue and hindering ecosystem growth.

A unified standard would:

- Enable seamless multi-wallet support for zkApps.
- Provide clear guidance for new wallet implementations.
- Reduce developer friction and accelerate zkApp adoption.
- Facilitate better user experiences, such as easy wallet switching.

[RFC-0008](https://github.com/MinaFoundation/Core-Grants/blob/main/RFCs/rfc-0008-wallet-provider-api.md) already exists but has some flaws that can be improved, such as:

- Lack of methods for connecting and disconnecting the wallet.
- Use of `chainId` instead of `networkId` (the term used in both Auro Wallet and Pallad), and referring to it as a hexadecimal value while Mina's network ID is a string.

# Proposal

We propose the formal adoption of the **Mina Provider API**, an EIP-1193-style provider with methods as defined below.

The provider implements:

- `request(args: { method: string; params?: array }): Promise<unknown>`
- Event subscription via `on(eventName: string, listener: Function): void` and `removeListener(eventName: string, listener: Function): void`

All methods use positional parameters (array format) as per JSON-RPC conventions adopted by major providers.

## Specification

### Public Provider Methods

#### mina_blockHash

##### Description

Returns the hash of the latest block.

##### Parameters

*(none)*

##### Returns

`string` - the current block hash

##### Example

```json
// Request
{
    "method": "mina_blockHash",
    "params": []
}

// Response
{
    "result": "3NLeFzJBrAKh4BhpHFcs1DaPFkTKemgBdTY2W1EFYtrHJHfiC96Q"
}
```

#### mina_networkId

##### Description

Returns the current network ID.

##### Parameters

*(none)*

##### Returns

`string` - the network ID (e.g., "mina:mainnet", "mina:devnet")

##### Example

```json
// Request
{
    "method": "mina_networkId",
    "params": []
}

// Response
{
    "result": "mina:mainnet"
}
```

#### mina_getBalance

##### Description

Returns the balance of a given account.

##### Parameters

1. `string` - account public key (address)
2. `string` (optional) - token ID

##### Returns

`string` - balance as a string quantity

##### Example

```json
// Request
{
    "method": "mina_getBalance",
    "params": ["B62qoYJCCwNSGRw73ww5eNCZnKCvjoEtrqr9UxuLnVHamDBEwSXjaw3", "1"]
}

// Response
{
    "result": "5000000000"
}
```

#### mina_getTransactionCount

##### Description

Returns the transaction nonce for a given account.

##### Parameters

1. `string` - account public key (address)

##### Returns

`string` - nonce as a string quantity

##### Example

```json
// Request
{
    "method": "mina_getTransactionCount",
    "params": ["B62qoYJCCwNSGRw73ww5eNCZnKCvjoEtrqr9UxuLnVHamDBEwSXjaw3"]
}

// Response
{
    "result": "42"
}
```

#### mina_sendSignedTransaction

##### Description

Submits a signed transaction to the network.

##### Parameters

1. `object` - signed transaction object

##### Returns

`string` - transaction hash

##### Example

```json
// Request
{
    "method": "mina_sendSignedTransaction",
    "params": [{
        "signature": {
            "field":"2270917456437054151866310845889777237190541188364956508055930611671093285487",
            "scalar":"21449516654198770916732742168324673178939547645509705487897779421915836159965"
        },
        "input": {
            "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
            "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
            "fee": "10000000",
            "amount": "1000000000",
            "nonce": "33",
            "memo": "Offline Payment",
            "validUntil": "4294967295"
        }
    }]
}

// Response
{
    "result": "5Ju...txhash..."
}
```

### Wallet Provider Methods

#### mina_accounts

##### Description

Returns the accounts currently connected to the dApp.

##### Parameters

*(none)*

##### Returns

`array<string>` - list of account public keys

##### Example

```json
// Request
{
    "method": "mina_accounts",
    "params": []
}

// Response
{
    "result": ["B62q1...", "B62q2..."]
}
```

#### mina_requestAccounts

##### Description

Prompts the user to connect accounts.

##### Parameters

*(none)*

##### Returns

`array<string>` - list of connected account public keys

##### Example

```json
// Request
{
    "method": "mina_requestAccounts",
    "params": []
}

// Response
{
    "result": ["B62q..."]
}
```

#### mina_addChain

##### Description

Requests to add a new network configuration.

##### Parameters

1. `object` - chain configuration (including `url`)

##### Returns

`null`

##### Example

```json
// Request
{
    "method": "mina_addChain",
    "params": [{
        "url": "https://api.minascan.io/node/devnet/v1/graphql"
    }]
}

// Response
{
    "result": null
}
```

#### mina_switchChain

##### Description

Requests to switch to a different network.

##### Parameters

1. `string` - target network ID

##### Returns

`null`

##### Example

```json
// Request
{
    "method": "mina_switchChain",
    "params": ["mina:devnet"]
}

// Response
{
    "result": null
}
```

#### mina_signTransaction

##### Description

Requests the wallet to sign a transaction without sending it.

##### Parameters

1. `object` - transaction request (of type `TransactionRequest` as used in the reference implementation)

##### Returns

`object` - signed `ZkappCommand` or `Signature`

##### Example

```json
// Request
{
    "method": "mina_signTransaction",
    "params": [{
        "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "fee": "10000000",
        "amount": "1000000000",
        "nonce": "33",
        "memo": "Offline Payment",
        "validUntil": "4294967295"
    }]
}

// Response
{
    "result": {
        "field":"2270917456437054151866310845889777237190541188364956508055930611671093285487",
        "scalar":"21449516654198770916732742168324673178939547645509705487897779421915836159965"
    }
}
```

#### mina_sendTransaction

##### Description

Requests the wallet to sign and send a transaction.

##### Parameters

1. `object` - transaction request, same as for `mina_signTransaction`

##### Returns

`string` - transaction hash

##### Example

```json
// Request
{
    "method": "mina_sendTransaction",
    "params": [{
        "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "fee": "100000000",
        "amount": "1000000000",
        "nonce": "33",
        "memo": "Offline Payment",
        "validUntil": "4294967295"
    }]
}

// Response
{
    "result": "5Ju...txhash..."
}
```

#### wallet_revokePermissions

##### Description

Revokes the current connection permissions.

##### Parameters

*(none)*

##### Returns

`null`

##### Example

```json
// Request
{
    "method": "wallet_revokePermissions",
    "params": []
}

// Response
{
    "result": null
}
```

## Events

The provider emits the following events:

### chainChanged

If the network the Provider is connected to changes, it emits the `chainChanged` event.

##### Parameter

`networkId: string`

The new network ID as a string.

### accountsChanged

If the accounts available to the Provider change (i.e., the result of `mina_accounts` would differ), it emits the `accountsChanged` event.

##### Parameter

`accounts: array<string>`

An array of account public keys (addresses).

### message

The `message` event is for arbitrary notifications not covered by other events.

##### Parameter

`message: { type: string; data: unknown }`

This can be used for future extensions, such as subscription notifications if supported.

## Reference Implementation

The proposed methods and types are directly derived from the TypeScript definitions in the Vimina project:

https://raw.githubusercontent.com/wagmina/vimina/5bdf1a500343a6fdd1bc621109041334d3fd00a8/src/types/jsApiStandard.ts

This file defines `PublicRpcSchema` and `WalletRpcSchema` as the authoritative source.

# Compatibility

The current APIs of Auro Wallet, Pallad, and [RFC-0008](https://github.com/MinaFoundation/Core-Grants/blob/main/RFCs/rfc-0008-wallet-provider-api.md) have been taken into consideration to minimize required changes and ensure a smooth transition and implementation.

Current wallets can continue to support both this standard and their existing formats until all zkApps update to the new standard.

Additionally, [wagmina](https://github.com/wagmina/wagmina), developed by the author of this proposal, can be used to provide zkApp developers with a consistent interface regardless of the underlying wallet implementation, as demonstrated in [Scaffold Mina](https://github.com/wagmina/scaffold-mina).

# Conclusion

Adopting this Mina Provider API standard represents a straightforward yet impactful step toward a more unified and developer-friendly ecosystem. It encourages wallet providers (Auro Wallet, Pallad, and emerging ones) to converge on a common interface.

**Author:** TheMonkeyCoder  
**Status:** Draft

Community discussion and collaboration with wallet teams are welcomed to refine and implement this standard as an official MIP.