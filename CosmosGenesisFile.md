# Cosmos Genesis File
Structur and parameters explanation of the genesis file, can be found [here](https://hub.cosmos.network/main/resources/genesis.html).

## Not changeable parameters
* genesis_time
* chain_id

Consensus Validator:
* pub_key_types

Staking:
* historical_entries
* max_validators

Genesis Transactions (gen_txs):
* memo
* gas_limit

## Changeable parameters
Consensus Block:
* max_bytes
* max_gas
* time_iota_ms

Consensus Evidence:
* max_age_num_blocks
* max_age_duration
* max_bytes

App State:
* max_memo_characters
* tx_sig_limit
* tx_size_cost_per_byte
* sig_verify_cost_ed25519
* sig_verify_cost_secp256k1

Crisis Fee:
* amount
* denom

Distribution:
* base_proposer_reward
* bonus_proposer_reward
* community_tax
* withdraw_addr_enabled

Genesis Transactions (gen_txs):
* moniker
* rate
* max_rate
* max_change_rate
* delegator_address
* validator_address

Connected Genesis:
* max_expected_time_per_block

Mint:
* annual_provisions
* inflation
* blocks_per_year
* goal_bonded
* inflation_max
* inflation_min
* inflation_rate_change
* mint_denom

UGDMint:
* subsidy_halving_interval
* blocks_per_year
* goal_bonded
* mint_denom

Slashing:
* downtime_jail_duration
* min_signed_per_window
* signed_blocks_window
* slash_fraction_double_sign
* slash_fraction_downtime

Staking:
* bond_denom
* max_entries
* min_commission_rate
* unbonding_time
