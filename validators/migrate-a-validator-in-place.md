# In-Place Upgrade Instructions from `secret-3` to `secret-4`

:warning: Please read carefully before you begin the upgrade.

## Validators

:warning: Don't delete your `secret-3` machine and its files, as we might have to relaunch it.

You're probably familiar with SGX by now:

- [Setup SGX](../node%20runners/setup-sgx-mainnet.md)
- [Verify SGX](../node%20runners/verify-sgx.md)

### 1. Preparation
Prepare your `secret-3` validator to halt at 2021-11-10 15:00:00 UTC.

On the old chain (`secret-3`):

```bash
perl -i -pe 's/^halt-height =.*/halt-height = 813800/' ~/.secretd/config/app.toml

sudo systemctl restart secret-node
```

Since the halt time has passed, just stop your node
```bash
sudo systemctl stop secret-node
```

### 2. Export up your keys
If you need to export keys from `secretd` or `secretcli`, do that now:
```shell
secretcli keys export <your key> 2> foobar.key
```
Depending on your setup the command above might prompt for a password in which case remove the `2> foobar.key` part, follow the prmpt, and manually copy the key to a file.

### 3. Make a backup
After the chain halts Back up the state of the `secret-3` chain:
```shell
mv ~/.secretd ~/.secretd.backup
```

### 4. Install the new binaries
Then install the new release:
```bash
cd ~

wget "https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/secretnetwork_1.2.0_amd64.deb"

echo "b8cf9be5c81e510584a9e829b9db4bfee02fedb2584e76bb6d6901a29d73ad06 secretnetwork_1.2.0_amd64.deb" | sha256sum --check

sudo apt remove secretnetwork

sudo apt install -y ./secretnetwork_1.2.0_amd64.deb

sudo chmod +x /usr/local/bin/secretd
```

### 5. Import your keys
import the key files that you previously exported. `secretcli` and `secretd` now share the configuration so you only have to import to one of them.

Also, make sure you're using the right `--hd-path`. e.g. `--legacy-hd-path` or `--hd-path "44'/118'/0'/0/0"`.
```shell
secretd keys import <your key> foobar.key
```

### 6. Initialize the new node

```shell
secretd init <MONIKER> --chain-id secret-4

mkdir -p ~/.secretd/.node
cp ~/.secretd.backup/.node/seed.json ~/.secretd/.node/seed.json

cp ~/.secretd.backup/config/priv_validator_key.json ~/.secretd/config/priv_validator_key.json

mkdir -p /opt/secret/.sgx_secrets
sudo cp -rf ~/.sgx_secrets /opt/secret/
```
- **Note**: if you get `Error: failed to parse log level (main:info,state:info,*:error): Unknown Level String: 'main:info,state:info,*:error', defaulting to NoLevel` when running `secretd`, change the following value in `.secretd/config/config.toml`:
```
# old value
log_level = "main:info,state:info,*:error"

# new value
log_level = "info"
```

### 7. Import the quicksync data (optional)
```bash
cd ~

wget "TBD"

echo "TBD quicksync.tar.xz" | sha256sum --check

pv quicksync.tar.xz | tar -xJf -
```

### 8. Set up your SGX machine and become a `secret-4` validator

```bash
cd ~

wget -O ~/.secretd/config/genesis.json "https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/genesis.json"

echo "759e1b6761c14fb448bf4b515ca297ab382855b20bae2af88a7bdd82eb1f44b9 .secretd/config/genesis.json" | sha256sum --check

perl -i -pe 's/pruning =.*/pruning = "everything"/' ~/.secretd/config/app.toml

perl -i -pe 's/persistent_peers =.*/persistent_peers = "7127b730503eb8fd9e3c5fb7a92a210472da8fd3\@bootstrap.node.scrtlabs.com:26656"/' ~/.secretd/config/config.toml

sudo systemctl enable secret-node

sudo systemctl start secret-node # (Now your new node is live and catching up)

secretd config node tcp://localhost:26657
secretd config output json
```

Now wait until you're done catching up. This is fast.  
Once the following command outputs `true` you can continue:

```bash
watch 'secretcli status | jq ".sync_info.catching_up == false"'
```

Once your node is done catching up, you can unjail your validator:

```bash
secretcli config chain-id secret-4
secretcli tx slashing unjail --from "$YOUR_KEY_NAME" --gas-prices 0.25uscrt
```

You’re now a validator in `secret-4`! :tada:

To make sure your validator is unjailed, look for it in here:

```bash
secretcli q staking validators | jq -r '.[] | select(.status == 2) | .description.moniker'
```

## In case of an upgrade failure

If after a few hours the Enigma team announces on the chat that the upgrade failed, we will relaunch `secret-3`.

1. restore the old state:

```bash
mv ~/.secretd ~/.secretd.failed
mv ~/.secretd.backup ~/.secretd
perl -i -pe 's/^halt-height =.*/halt-height = 0/' ~/.secretd/config/app.toml

sudo systemctl restart secret-node
   ```

2. Wait for 67% of voting power to come back online.

## Removing an installation

You can remove previous `secretnetwork` installations and start fresh using:

```bash
cd ~
sudo systemctl stop secret-node
secretd unsafe-reset-all
sudo apt purge -y secretnetwork
rm -rf ~/.secretd/*
```
