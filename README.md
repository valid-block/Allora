# Allora

### Prerequisites
1. **Install Allorad**:  
   You can install the Allorad binary using the following command (Linux/Mac):
   ```
   curl -sSL https://raw.githubusercontent.com/allora-network/allora-chain/main/install.sh | bash -s -- v0.0.8
   ```
   Alternatively, clone the repository and build it manually:
   ```
   git clone https://github.com/allora-network/allora-chain.git
   cd allora-chain
   make install
   ```

2. **Install Docker**: Ensure Docker is installed, as it is required for running the node.

3. **System Requirements**: 
   - 4+ CPUs
   - 16GB+ RAM
   - 1TB SSD storage
   - Stable internet connection

### Steps to Launch the Validator Node

#### 1. **Run a Full Node**
   First, run and sync a full node:
   ```
   docker compose up -d
   ```
   Check the node’s syncing status:
   ```
   curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
   ```
   Wait until the node is fully synced (`"false"`).

#### 2. **Fund Your Account**
   The script `l1_node.sh` generates the account info. You can fund your account using a faucet if you are on a testnet environment:
   ```
   cat data/validator0.account_info
   ```
   Use the address to request tokens from the Allora testnet faucet.

#### 3. **Stake as a Validator**
   Access the validator shell:
   ```
   docker compose exec validator0 bash
   ```
   Prepare your staking information:
   ```bash
   cat > stake-validator.json << EOF
   {
       "pubkey": $(allorad comet show-validator),
       "amount": "1000000uallo",
       "moniker": "$(echo $MONIKER)",
       "commission-rate": "0.1",
       "commission-max-rate": "0.2",
       "commission-max-change-rate": "0.01",
       "min-self-delegation": "1"
   }
   EOF
   ```
   Execute the staking command:
   ```bash
   allorad tx staking create-validator ./stake-validator.json \
       --chain-id=testnet \
       --home="$APP_HOME" \
       --keyring-backend=test \
       --from="$MONIKER"
   ```

#### 4. **Verify Your Validator Setup**
   Confirm that your validator is properly registered:
   ```bash
   VAL_PUBKEY=$(allorad comet show-validator | jq -r .key)
   allorad q staking validators -o=json | jq '.validators[] | select(.consensus_pubkey == $VAL_PUBKEY)'
   ```
