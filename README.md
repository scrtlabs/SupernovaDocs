# Supernova Upgrade

Welcome to the Supernova Upgrade! On the 10th of November, *block height 813800* we will be undergoing a network upgrade.

The estimated halt time is `Wed Nov 10 15:00:00 UTC 2021`

We expect the upgrade process to take up to 2 hours.

Please note that the `ibc` module will be disabled on launch, and will require a governance proposal to enable

## App Developers

We recommend reading the [CHANGELOG.md](CHANGELOG.md) as well as specific information in [app developers](./app%20developers) and make sure your apps are compatible with the upgrade

## Node Runners and Validators

### Preparation

Gracefully halt the secret-3 by configuring halt height - 

```
perl -i -pe 's/^halt-height =.*/halt-height = 813800/' ~/.secretd/config/app.toml
```

And restarting your node

### Recovery

Prior to exporting secret-3 state, validators are encouraged to take a full data snapshot at the export height before proceeding. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the .secretcli and .secretd directories.

In the event that the upgrade does not succeed, validators and operators must downgrade back to v1.0.6, and restore to their latest snapshot before restarting their nodes.

### State Migration

Validators may perform these steps manually, however links to the genesis file will be published once export and migration are complete.

1. Export the state:
```bash
secretd export --height=813800 --for-zero-height --jail-whitelist secretvaloper1qx5pppsfrqwlnmxj7prpx8rysxm2u5vzhaux25 > secret-3-genesis-export.log
tail -1 secret-3-genesis-export.log > secret-3-genesis-export.json
```

2. Download Secretcli v1.2.0
```
wget -O secretcli https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/secretcli-Linux
chmod +x ./secretcli
```

3. Migrate the state, using the downloaded Secretcli v1.2.0
```
./secretcli migrate ./secret-3-genesis-export.json --chain-id=secret-4 --initial-height=813800 --genesis-time=2021-11-10T15:00:00Z --log_level info > genesis.json
```

4. Verify the hash of the final genesis file:
```
$ echo "759e1b6761c14fb448bf4b515ca297ab382855b20bae2af88a7bdd82eb1f44b9 genesis.json" | sha256sum --check
```

### Node Migration

Follow further instructions on migration [here](./validators/migrate-a-validator.md)
