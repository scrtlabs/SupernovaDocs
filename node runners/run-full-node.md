# How To Join Secret Network as a Full Node

This document details how to join the Secret Network `secret-4` mainnet as a full node. Once your full node is running, you can turn it into a validator in the optional last step.

## Requirements

- Ubuntu/Debian host (with ZFS or LVM to be able to add more storage easily)
- A public IP address
- Open ports `TCP 26656 & 26657` _Note: If you're behind a router or firewall then you'll need to port forward on the network device._
- Reading [Tendermint: Running in production](https://docs.tendermint.com/v0.34/tendermint-core/running-in-production.html)
- RPC address of an already active node. You can use any node that exposes RPC services.

### Minimum requirements

- 16GB RAM
- 100GB HDD
- 1 dedicated core of any Intel Skylake processor (Intel® 6th generation) or better

### Recommended requirements

- 32GB RAM
- 256GB SSD
- 2 dedicated cores of any Intel Skylake processor (Intel® 6th generation) or better
- Motherboard with support for SGX in the BIOS

Refer to [Intel Processor Specifications](https://ark.intel.com/content/www/us/en/ark.html#@Processors) if you're unsure if your processor supports SGX

## Installation

### 0. Step up SGX on your local machine
See instructions for [setup](setup-sgx-mainnet.md) and [verification](verify-sgx.md).

### 1. Download the Secret Network package installer for Debian/Ubuntu:

```bash
wget https://github.com/enigmampc/SecretNetwork/releases/download/v1.2.0/secretnetwork_1.2.0_amd64.deb
# check the hash of the downloaded binary
echo "b8cf9be5c81e510584a9e829b9db4bfee02fedb2584e76bb6d6901a29d73ad06 secretnetwork_1.2.0_amd64.deb" | sha256sum --check
```

([How to verify releases](https://github.com/scrtlabs/SecretNetwork/blob/master/docs/verify-releases.md))

### 2. Install the package:

```bash
sudo dpkg -i secretnetwork_1.2.0_amd64.deb
```

### 3. Initialize your installation of the Secret Network.

Choose a **moniker** for yourself, and replace `<MONIKER>` with your moniker below.
This moniker will serve as your public nickname in the network.

```bash
secretd init <MONIKER> --chain-id secret-4
```

### 4. Download a copy of the Genesis Block file: `genesis.json`

```bash
wget -O ~/.secretd/config/genesis.json "https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/genesis.json"
```

### 5. Validate the checksum for the `genesis.json` file you have just downloaded in the previous step:

```bash
echo "759e1b6761c14fb448bf4b515ca297ab382855b20bae2af88a7bdd82eb1f44b9 $HOME/.secretd/config/genesis.json" | sha256sum --check
```

### 6. Validate that the `genesis.json` is a valid genesis file:

```bash
secretd validate-genesis
```

### 7. The rest of the commands should be run from the home folder (`/home/<your_username>`)

```bash
cd ~
```

### 8. Initialize secret enclave

You can choose between two ways, **8a (automatic)** or **8b (manual)**:

**Note:** if this machine has been registered before, and have the following files:
```bash
/home/user/.sgx_secrets/
├── consensus_seed.sealed
└── new_node_seed_exchange_keypair.sealed
```
you can move them to `/opt/secret/.sgx_secrets` and skip to **[step 16](#16-add-persistent-peers-to-your-configuration-file)** (if not working, try registering anyway).

### 8a. Initialize secret enclave - Automatic Registration (EXPERIMENTAL)

- **Note:** Make sure SGX is running or this step might fail.

Make sure the directory `/opt/secret/.sgx_secrets` exists:

```bash
mkdir -p /opt/secret/.sgx_secrets
```

Create env variables:

```bash
export SCRT_ENCLAVE_DIR=/usr/lib
export SCRT_SGX_STORAGE=/opt/secret/.sgx_secrets
```

Register:

```bash
secretd auto-register --node https://api.scrt.network:443 --registration-node http://register.mainnet.enigma.co:26667
```

**If this step was successful, you can skip straight to [step 16](#16-add-persistent-peers-to-your-configuration-file)**

### 8b. Initialize secret enclave - Manual Registration (legacy)

Make sure the directory `/opt/secret/.sgx_secrets/` exists:

```bash
mkdir -p /opt/secret/.sgx_secrets/
```

Make sure SGX is running or this step might fail.

```bash
secretd init-enclave 
```

### 9. Check that initialization was successful

Attestation certificate should have been created by the previous step

```bash
ls -lh /opt/secret/.sgx_secrets/attestation_cert.der
```

### 10. Check your certificate is valid

Should print your 64 character registration key if it was successful.

```bash
PUBLIC_KEY=$(secretd parse /opt/secret/.sgx_secrets/attestation_cert.der  2> /dev/null | cut -c 3-)
echo $PUBLIC_KEY
```

### 11. Config `secretcli`, to point to a working node and import a key with some SCRT

The steps using `secretcli` can be run on any machine, they don't need to be on the full node itself. We'll refer to the machine where you are using `secretcli` as the "CLI machine" below.

To run the steps with `secretcli` on another machine, [set up the CLI](../contract%20developers/cli.md) there.

Configure `secretcli`. Initially you'll be using the bootstrap node, as you'll need to connect to a running node and your own node is not running yet.

```bash
secretcli config chain-id secret-4
secretcli config node tcp://TBD:26657
secretcli config output json
```

Set up a key. Make sure you back up the mnemonic and the keyring password.

```bash
secretcli keys add $INSERT_YOUR_KEY_NAME
```

This will output your address, a 45 character-string starting with `secret1...`. Then you can fund it with some SCRT.

### 12. Register your node on-chain

Run this step on the CLI machine. If you're using a different CLI machine than the full node, copy `/opt/secret/.sgx_secrets/attestation_cert.der` from the full node to the CLI machine.

```bash
secretcli tx register auth </opt/secret/.sgx_secrets/attestation_cert.der> --from $INSERT_YOUR_KEY_NAME
```

### 13. Pull & check your node's encrypted seed from the network

Run this step on the CLI machine.

```bash
SEED=$(secretcli query register seed "$PUBLIC_KEY" | cut -c 3-)
echo $SEED
```

### 14. Get additional network parameters

Run this step on the CLI machine.

These are necessary to configure the node before it starts.

```bash
secretcli query register secret-network-params
ls -lh ./io-master-cert.der ./node-master-cert.der
```

If you're using a different CLI machine than the validator node, copy `node-master-cert.der` from the CLI machine to the validator node.

### 15. Configure your secret node

From here on, run commands on the full node again.

```bash
mkdir -p ~/.secretd/.node
secretd configure-secret node-master-cert.der "$SEED"
```

### 16. Add persistent peers to your configuration file.

```bash
perl -i -pe 's/persistent_peers = ""/persistent_peers = "971911193b09a17c347565d311a3cc4f6004156d\@peer.node.scrtlabs.com:26656"/' ~/.secretd/config/config.toml
```

### 17. Listen for incoming RPC requests so that light nodes can connect to you:

```bash
perl -i -pe 's/laddr = .+?26657"/laddr = "tcp:\/\/0.0.0.0:26657"/' ~/.secretd/config/config.toml
```

### 18. Enable `secret-node` as a system service:

```bash
sudo systemctl enable secret-node
```
If the service fails to run, make sure you have the full path of secertd as ExecStart (/usr/local/bin/secretd)

### 19. Start `secret-node` as a system service:

```bash
sudo systemctl start secret-node
```

### 20. If everything above worked correctly, the following command will show your node streaming blocks (this is for debugging purposes only, kill this command anytime with Ctrl-C):

```bash
journalctl -f -u secret-node
```

```
-- Logs begin at Mon 2020-02-10 16:41:59 UTC. --
Nov 09 11:16:31 scrt-node-01 secretd[619529]: 11:16AM INF indexed block height=12 module=txindex
Nov 09 11:16:35 scrt-node-01 secretd[619529]: 11:16AM INF Ensure peers module=pex numDialing=0 numInPeers=0 numOutPeers=0 numToDial=10
Nov 09 11:16:35 scrt-node-01 secretd[619529]: 11:16AM INF No addresses to dial. Falling back to seeds module=pex
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF Timed out dur=4983.86819 height=13 module=consensus round=0 step=1
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4","parts":{"hash":"58F6211ED5D6795E2AE4D3B9DBB1280AD92B2EE4EEBAA2910F707C104258D2A0","total":1}},"height":13,"pol_round":-1,"round":0,"signature":"eHY9dH8dG5hElNEGbw1U5rWqPp7nXC/VvOlAbF4DeUQu/+q7xv5nmc0ULljGEQR8G9fhHaMQuKjgrxP2KsGICg==","timestamp":"2021-11-09T11:16:36.7744083Z"}
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF received complete proposal block hash=0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4 height=13 module=consensus
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF finalizing commit of block hash=0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4 height=13 module=consensus num_txs=0 root=E4968C9B525DADA22A346D5E158C648BC561EEC351F402A611B9DA2706FD8267
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF minted coins from module account amount=6268801uscrt from=mint module=x/bank
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF executed block height=13 module=state num_invalid_txs=0 num_valid_txs=0
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF commit synced commit=436F6D6D697449447B5B373520353520323020352032342031312032333820353320383720313137203133372031323020313638203234302035302032323020353720343520363620313832203138392032333920393920323439203736203338203131322035342032332033203233362034375D3A447D
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF committed state app_hash=4B371405180BEE3557758978A8F032DC392D42B6BDEF63F94C2670361703EC2F height=13 module=state num_txs=0
^C
```

You are now a full node. :tada:

### 21. Get your node ID with:

```bash
secretd tendermint show-node-id
```

And publish yourself as a node with this ID:

```
<your-node-id>@<your-public-ip>:26656
```

Be sure to point your CLI to your running node instead of the bootstrap node

```
secretcli config node tcp://localhost:26657
```

If someone wants to add you as a peer, have them add the above address to their `persistent_peers` in their `~/.secretd/config/config.toml`.

And if someone wants to use your node from their `secretcli` then have them run:

```bash
secretcli config chain-id secret-4
secretcli config output json
secretcli config node tcp://<your-public-ip>:26657
```

### 22. Optional: make your full node is a validator

Your full node is now part of the network, storing and verifying chain data and Secret Contracts, and helping to distribute transactions and blocks. It's usable as a sentry node, for people to connect their CLI or light clients, or just to support the network.

It is however not producing blocks yet, and you can't delegate funds to it for staking. To do that you'll have to turn it into a validator by submitting a `create-validator` transaction.

On the full node, get the pubkey of the node:

```bash
secretd tendermint show-validator
```

The pubkey is an 83-character string starting with `secretvalconspub...`.

On the CLI machine, run the following command. The account you use becomes the operator account for your validator, which you'll use to collect rewards, participate in on-chain governance, etc., so make sure you keep good backups of the key. `<moniker>` is the name for your validator which is shown e.g. in block explorers.

```bash
secretcli tx staking create-validator \
  --amount=<amount-to-delegate-to-yourself>uscrt \
  --pubkey=<pubkey of the full node> \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --moniker="<moniker>" \
  --from=$INSERT_YOUR_KEY_NAME
```

The `create-validator` command allows using some more parameters. For more info on these and the additional parameters, run `secretcli tx staking create-validator --help`.

After you submitted the transaction, check you've been added as a validator:

```bash
secretcli q staking validators | grep moniker
```

Congratulations! You are now running a validator on the Secret Network testnet.
