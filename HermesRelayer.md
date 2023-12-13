## Setting up a Cosmos SDK Relayer with Hermes

**Prerequisites:**

* A basic understanding of the Cosmos SDK and Inter-Blockchain Communication (IBC) protocol
* Access to a terminal or command-line interface (CLI)
* A recent version of Rust installed (1.71.0 or higher)
* A local Cosmos SDK blockchain instance running

### Installing Hermes

1. Download the latest Hermes release archive from the official repository: [https://github.com/informalsystems/hermes](https://github.com/informalsystems/hermes)

2. Extract the archive into a dedicated directory:

```bash
mkdir -p $HOME/.hermes/bin
tar -C $HOME/.hermes/bin/ -vxzf $ARCHIVE_NAME
```

3. Update your system's PATH environment variable to include the Hermes binary directory to enable access from the command line:

```bash
export PATH="$HOME/.hermes/bin:$PATH"
```

**Verifying Installation**

To confirm that Hermes is correctly installed and accessible, you can try running the `hermes --version` command. A successful installation should display the Hermes version information.

### Configuring Hermes for Relaying

Once Hermes is installed, you'll need to configure it to relay IBC packets between your Cosmos SDK blockchain instances. This involves providing Hermes with the necessary chain configuration details, including chain IDs, RPC endpoints, and authentication credentials (if applicable).

**Chain Configuration**

1. Create a JSON file named `chains.json` in the `~/.hermes/config` directory. This file will hold the configuration details for each chain you want to relay between.

2. For each chain, define a configuration object with the following properties:
   - `chain_id`: The unique identifier of the chain
   - `endpoint`: The RPC endpoint of the chain's full node
   - `auth_type`: The authentication type (e.g., `GRPC`)
   - `auth_key`: The authentication key (if applicable)
   - `auth_value`: The authentication value (if applicable)

3. Save the configuration file.


### Key Generation and Import for Hermes Relayer

Setting up a Hermes relayer involves generating and importing keys to enable secure communication between the relayer and the Cosmos SDK blockchain instances that it will be relaying IBC packets between.

**Generating Keys:**

1. Utilize the `paxd keys add` command to generate a unique key pair for each chain you intend to relay packets across.

    ```bash
    paxd keys add testnet-unigrid --output json | jq '.' > key_file_unigrid.json
    ```

   This command generates a new key pair and saves the JSON representation of the key pair to a file named `key_file_unigrid.json`.

**Importing Keys:**

2. Import the generated keys into Hermes using the `hermes keys add` command.

   ```bash
   hermes keys add --chain unigrid-testnet-1 --key-file key_file_unigrid.json
   ```

   This command introduces the key pair into Hermes, allowing the relayer to utilize it for connecting to the designated chain.

***Repeat these steps for each chain you are going to relay to***

For example connecting to Cosmos Hub testnet `theta-testnet-001`:

   ```bash
   gaiad keys add testnet-cosmos --output json | jq '.' > key_file_hub.json
   ```

   ```bash
   hermes keys add --chain theta-testnet-001 --key-file key_file_hub.json
   ```


```toml
# Config for Hermes Relayer

# Hermes requires a configuration file to define the details of the Cosmos SDK blockchain instances
# that it will be relaying IBC packets between. This file is located in the
# $HOME/.hermes/config.toml directory.

# Example configuration file for a Hermes relayer that will relay packets between the
# Unigrid-testnet-1 chain and another chain:

[chains]

[chains.address_type]
derivation = "cosmos"

[[chains]]
id = "unigrid-testnet-1"
type = "CosmosSdk"
rpc_addr = "https://rpc-testnet.unigrid.org/"
grpc_addr = "http://38.242.156.2:9090"
rpc_timeout = "10s"
trusted_node = false
account_prefix = "unigrid"
key_name = "keyunigrid"
key_store_type = "Test"
store_prefix = "ibc"
default_gas = 100000
max_gas = 400000
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
max_grpc_decoding_size = 33554432
clock_drift = "5s"
max_block_time = "30s"
ccv_consumer_chain = false
memo_prefix = ""
sequential_batch_tx = false

[chains.event_source]
mode = "push"
url = "wss://rpc-testnet.unigrid.org/websocket"
batch_delay = "500ms"

[chains.trust_threshold]
numerator = "1"
denominator = "3"

[chains.gas_price]
price = 0.025
denom = "ugd"

[chains.packet_filter]
policy = 'allow'
list = [
  ['ica*', '*'],
  ['transfer', 'channel-0'],
]

[chains.packet_filter.min_fees]
```

Then if we want to connect to the Cosmos Hub Testnet:

```toml
[[chains]]
id = "theta-testnet-001"
type = "CosmosSdk"
rpc_addr = "https://rpc.sentry-01.theta-testnet.polypore.xyz"
grpc_addr = "https://grpc.sentry-01.theta-testnet.polypore.xyz"
rpc_timeout = "10s"
trusted_node = false
account_prefix = "cosmos"
key_name = "keyhub"
key_store_type = "Test"
store_prefix = "ibc"
default_gas = 100000
max_gas = 400000
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
max_grpc_decoding_size = 33554432
clock_drift = "5s"
max_block_time = "30s"
ccv_consumer_chain = false
memo_prefix = ""
sequential_batch_tx = false

[chains.event_source]
mode = "push"
url = "wss://rpc.sentry-01.theta-testnet.polypore.xyz/websocket"
batch_delay = "500ms"

[chains.trust_threshold]
numerator = "1"
denominator = "3"

[chains.gas_price]
price = 0.025
denom = "uatom"

[chains.packet_filter]
policy = "allow"
list = [[
    "transfer",
    "channel-141",
]]

[chains.packet_filter.min_fees]

[chains.address_type]
derivation = "cosmos"

```

For more information, refer to the [Hermes documentation](https://hermes.informal.systems/tutorials/production/setup-hermes.html).


**Relaying IBC Packets**

To start relaying packets between the configured chains, simply run the `hermes start &> hermes.log` command. Hermes will continuously monitor the chains for IBC events and relay packets accordingly.

## Open IBC channel

```bash
hermes create channel --a-chain unigrid-testnet-1 --b-chain theta-testnet-001 --a-port transfer --b-port transfer --new-client-connection
```
This command initiates the creation of a channel between the `unigrid-testnet-1` and `theta-testnet-001`

**Monitoring Relayer Status**

You can use the `hermes health-check` command to check the current health of the relayer.

**Further Configuration and Customization:**

Hermes offers various options for configuring and customizing its behavior. You can explore the [documentation](https://hermes.informal.systems/index.html) and command-line flags to tailor the relayer to your specific requirements.



