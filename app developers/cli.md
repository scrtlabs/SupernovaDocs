# CLI Changes

Due to the Cosmos SDK upgrade, there are multiple changes to CLI functionality. We recommend to anyone relying on SecretCLI commands to test their application or code to make sure they are compatible with the new application. 

Highlights include:

* Home directory for the CLI is now `~/.secretd`
 
* Key format has been changed, and keys must be manually migrated to be used in the new CLI

* Querying of balances is now done via `secretcli query bank balances <account>`

* Sending of tokens is now done via the `secretd tx bank send` command
  
* More detailed SDK changes can be seen [here](https://github.com/cosmos/cosmos-sdk/blob/v0.44.3/CHANGELOG.md)

## Migrating keys

Always make sure you're using the right hd-path.

### Using the mnemonic
If you have the mnemonic for your key, you can import it to the new CLI with this command:
```shell
secretcli keys add mykey --recover
```

### Without the mnemonic
If you _do not_ have access to your mnemonic phrase, you will want to export your key from the old CLI and import it with the new CLI:
```shell
# Old CLI
secretcli keys export mykey
```
This prints the private key to `stderr`, you can then paste in into the file `mykey.backup`.
```shell
# New CLI
secretcli keys import mykey mykey.backup
```
