# Supernova Upgrade

Welcome to the Supernova Upgrade! On the 10th of November, *block height 813800* we will be undergoing a network upgrade.

The estimated halt time is `Wed Nov 10 15:00:00 UTC 2021`

## Recovery
Prior to exporting secret-3 state, validators are encouraged to take a full data snapshot at the export height before proceeding. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the .secretcli and .secretd directories.

In the event that the upgrade does not succeed, validators and operators must downgrade back to v1.0.6, and restore to their latest snapshot before restarting their nodes.

## State Migration

Validators may perform these steps manually, however links to the genesis file will be published once export and migration are complete.

1. Export the state:
```bash
secretd export --height=813800 --for-zero-height --jail-whitelist secretvaloper1qx5pppsfrqwlnmxj7prpx8rysxm2u5vzhaux25 > secret-3-genesis-export.json
```

2. Verify the hash:
```
$ sha256sum secret-3-genesis-export.json
```

3. Download Secretcli v1.2.0
```
wget -o secretcli https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/secretcli-Linux
```

4. Migrate the state
```
secretd migrate ./secret-3-genesis-export.json --chain-id=secret-4 --initial-height=813800 --genesis-time=TBD > genesis.json
```

5. Verify the hash of the final genesis file:
```
$ sha256sum genesis.json
```

## Validator Migration

You can find instructions on migration [here](https://github.com/scrtlabs/SupernovaDocs/blob/master/validators/migrate-a-validator.md)
