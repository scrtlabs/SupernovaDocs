# CHANGELOG

# 1.2.0

Version 1.2.0 has been released, called the Supernova upgrade!

## Highlights

* Upgraded to Cosmos SDK 0.44.3. Full changelog can be found [here](https://github.com/cosmos/cosmos-sdk/blob/v0.44.3/CHANGELOG.md)

* Gas prices are lower - as a result of performance upgrades and optimizations, gas amounts required will be much lower.
* GRPC for cosmos-sdk modules in addition to legacy REST API. See API [here](http://api.scrt.network/swagger/)

* New modules:

    * [Fee Grant](https://docs.cosmos.network/master/modules/feegrant/) - allows an address to give an allowance to another address
    * [Upgrade](https://docs.cosmos.network/master/modules/upgrade/) - Allows triggering of network-wide software upgrades, which significantly reduces the amount of coordination effort hard-forks require

* Auto Registration - The new node registering process is now automated via a new command `secretd auto-register`

* (experimental) Wasm Module Cache in the Enclave - Added a config value `wasm.contract-memory-enclave-cache-size` (default 0) 

## API and endpoints

### Registration module

* The endpoint `/reg/consensus-io-exch-pubkey` has been changed to `/reg/tx-key` and now returns `{"TxKey": bytes }`
* The endpoint `/reg/consensus-seed-exch-pubkey ` has been changed to `/reg/registration-key` and now returns `{"RegistrationKey": bytes }`

## GRPC and REST endpoints

GRPC endpoints have been added for cosmos-sdk modules in addition to legacy REST APIs, which remain mostly unchanged.

GRPC endpoints for the registration and compute modules will be added in a future testnet release

## SecretCLI and Secretd

Unlike other cosmos chains, we chose to maintain the differentiating CLI and Node runner executables.
SecretCLI still contains the interface for all user-facing commands and trying to run node-running commands using SecretCLI will fail.
Secretd now contains both node-running and user-facing commands.

As a result of cosmos-sdk upgrade, some CLI commands will have different syntax

Secretd nodes now run the REST API (previously named LCD REST server) by default on port 1317. You can change this behavior by
modifying /home/\<account\>/.secretd/config/app.toml and looking for the `api` configuration options

The directory at `~/.sgx_secrets` has been moved to `/opt/secret/.sgx_secrets`.

which lets you hold an amount of compiled wasm modules in the enclave's memory in an LRU cache. Increasing this number will increase the minimum RAM requirement but will increase performance of repeated calls to contracts with the same code, such as multiple queries handled in an API node, or a wave of usage coming in to a small set of contracts.

## SecretJS

Version 0.17.0 has been released!
SecretJS has been upgraded to support the Supernova upgrade.
All APIs remain unchanged, although the versions are NOT backwards compatible.

For compatibility with 1.2.0+, use SecretJS 0.17.x.
For compatibility with 1.0.x (legacy), use SecretJS 0.16.x

## CosmWasm

Secret-CosmWasm remains in a version that is compatible with the v0.10 of vanilla CosmWasm, and previous versions compatible with secret-2 will still work with this upgrade. 

A new feature has been added - plaintext logs. To send an unencrypted log (contract output), use `plaintext_log` instead of `log`.
This allows contracts to emit public events, and attach websockets to listen to specific events. To take advantage of this feature, compile contracts with
`cosmwasm-std = { git = "https://github.com/enigmampc/SecretNetwork", tag = "v1.2.0" }`

## Known Issues

* SecretCLI still uses /home/.secretd to store configuration and keys
* Signatures other than secp256k1 are unsupported for CosmWasm transactions.
* snip20 CLI commands not working
* Fee grant messages not supported by CosmWasm
* SecretCLI incompatible on M1 Mac
