# CLI Changes

Due to the Cosmos SDK upgrade, there are multiple changes to CLI functionality. We recommend anyone relying on SecretCLI commands to test their application or code to make sure they are compatible with the new application. 

Highlights include:

* Home directory for the CLI is now `~/.secretd`
 
* Key format has been changed, and keys must be manually migrated to be used in the new CLI

* Querying of balances is now done via secretcli q bank balances <account>

* Sending of tokens is now done via the `secretd tx bank send` command
  
* More detailed SDK changes can be seen [here](https://github.com/cosmos/cosmos-sdk/blob/v0.44.3/CHANGELOG.md)
