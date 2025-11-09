# HOW TO UPGRADE AZTEC NODE 

install aztec tools
```
bash -i <(curl -s https://install.aztec.network)
```

```
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
```
```
source ~/.bashrc
```

```
aztec-up
```



install foundry
```
curl -L https://foundry.paradigm.xyz | bash
```
```
source ~/.bashrc
```
```
foundryup
```

create a script to join the new testnet
follow the onscreen instructions 

```
nano setup.sh
```
paste the content below inside 
```
#!/bin/bash

clear
echo "──────────────────────────────────────────────"
echo "     AZTEC 2.1.2 TESTNET — VALIDATOR SETUP"
echo "──────────────────────────────────────────────"
echo ""

# --- Collect old validator details ---
echo "Please provide your previous validator information."
read -sp "   Enter your OLD Sequencer Private Key (hidden): " OLD_PRIVATE_KEY && echo
read -p  "   Enter your Sepolia RPC URL (e.g., https://sepolia.infura.io/v3/...): " ETH_RPC
echo ""
echo "Starting setup..."
echo ""

# --- Remove existing keystore (if any) ---
rm -f ~/.aztec/keystore/key1.json 2>/dev/null

echo "Be prepared to write down your new ETH private key, BLS private key, and ETH address."
read -p "   Press [Enter] to generate new keys..."

# --- Generate new validator keys ---
aztec validator-keys new \
  --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 && echo ""

# --- Extract generated key details ---
KEYSTORE_FILE="$HOME/.aztec/keystore/key1.json"
NEW_ETH_PRIVATE_KEY=$(jq -r '.validators[0].attester.eth' "$KEYSTORE_FILE")
NEW_BLS_PRIVATE_KEY=$(jq -r '.validators[0].attester.bls' "$KEYSTORE_FILE")
NEW_PUBLIC_ADDRESS=$(cast wallet address --private-key "$NEW_ETH_PRIVATE_KEY")

echo ""
echo "New keys generated successfully. Please save the following details securely:"
echo "   - ETH Private Key: $NEW_ETH_PRIVATE_KEY"
echo "   - BLS Private Key: $NEW_BLS_PRIVATE_KEY"
echo "   - Public ETH Address: $NEW_PUBLIC_ADDRESS"
echo ""

echo "You must now fund this address with approximately 0.2–0.5 Sepolia ETH for gas fees."
echo "   $NEW_PUBLIC_ADDRESS"
read -p "   After confirming the transaction, press [Enter] to continue..." && echo ""

# --- Approve STAKE token spending ---
echo "Approving 200,000 STAKE for the Aztec rollup contract..."
cast send 0x139d2a7a0881e16332d7D1F8DB383A4507E1Ea7A \
  "approve(address,uint256)" \
  0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 200000ether \
  --private-key "$OLD_PRIVATE_KEY" \
  --rpc-url "$ETH_RPC" && echo ""

# --- Join the Aztec testnet ---
echo "Registering your validator on the Aztec testnet..."
aztec add-l1-validator \
  --l1-rpc-urls "$ETH_RPC" \
  --network testnet \
  --private-key "$OLD_PRIVATE_KEY" \
  --attester "$NEW_PUBLIC_ADDRESS" \
  --withdrawer "$NEW_PUBLIC_ADDRESS" \
  --bls-secret-key "$NEW_BLS_PRIVATE_KEY" \
  --rollup 0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 && echo ""

# --- Completion message ---
echo ""
echo "──────────────────────────────────────────────"
echo "            SETUP COMPLETED SUCCESSFULLY ✅"
echo "──────────────────────────────────────────────"
echo ""
echo "You have successfully joined the new Aztec 2.1.2 testnet."
echo "Next steps:"
echo "   1. Restart your node using your new private key and address."
echo "   2. Monitor logs to confirm your validator is active."
echo ""
```

ctrl x+y then enter to save

make the script executable and run it
```
chmod +x setup.sh
./setup.sh
```

THE SCRIPT EXPLAINED 
1: Generate New Keys
Removes old keystore and creates new ETH + BLS keys.
Saves them to ~/.aztec/keystore/key1.json.
Extracts and displays your ETH private key, BLS private key, and public ETH address.

2: Fund the New Address
Send 0.2–0.5 Sepolia ETH to your new public address.
Confirm with ENTER

3: Approve STAKE Token
Uses your old sequencer private key to approve 200,000 STAKE for the rollup contract.
Required before you can register your validator.


4: Register the Validator
Executes aztec add-l1-validator to link your new keys on-chain.
Confirms your validator on the new Aztec 2.1.2 testnet.
Afterwards, restart your node using your new private key and address.




EDIT YOUR .ENV FILE 

```
cd aztec
nano .env
```

-Replace validator-Private-Keys value with the new ETH private key that you saved from running the script
-Replace coinbase address with the new public address u saved

CTRL+X, press Y, then Enter to save the edit

RESTART NODE
```
cd aztec && \
docker compose down -v && \
sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.1.2|' docker-compose.yml && \
docker compose pull && \
docker compose up -d
```

check the attester pub address(i.e the new public address on https://dashtec.xyz/queue to confirm your validator registration
