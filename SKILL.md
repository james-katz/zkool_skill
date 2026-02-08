---
name: zkool
description: Send and receive Zcash native token $ZEC through Zkool GraphQL API.
compatibility: Requires git, cargo, rust, npm, node, jq, and access to the internet.
license: MIT
triggers:
  - zcash
  - transaction
  - wallet
  - crypto
  - address
metadata:
  openclaw:
    emoji: "🛡️"
---

# Zcash / Zkool Wallet

A CLI tool to manage the state of a Zcash wallet, using Zkool GraphQL interface.

## Requirements

This skill  needs Rust installed on the host machine. It also need git to clone and compile the Zkool GraphQL API. Since the CLI output is JSON, it's also recomended to have jq installed.

## Setup

Here the step-by-step guide to clone, build and run the skill.

### Install build dependencies
```bash
sudo apt-get update
sudo apt-get install -y pkg-config libudev-dev
```

### Set env variables
```bash
export DB_PATH="zkool.db"
export LWD_URL="https://infra.zcashbr.com:9067"
```

### Install npm packages

This skill needs npm package `graphql-request`

```
npm install
```

### Setup Rust

```bash
# Check if rustup is already installed
if command -v rustup &> /dev/null
then
    echo "Rust is already installed. Updating instead."
    rustup update
else
    # Download and run the official rust installation script
    curl --proto '=https' --tlsv1.2 -sF https://sh.rustup.rs | sh -s -- -y
    
    # Source the cargo environment for the current session.
    # This might need to be done manually in the user's shell profile for persistence.
    source "$HOME/.cargo/env"
    echo "Rust installation/update complete."
fi
```

### Download and compile Zkool GraphQL API

To keep track of wallet installation, create a file `{baseDir}/wallet.json` with the content:
```json
{
    "accountId": -1,
}
```

A `-1` value indicates the wallet is not configured yet. This will be updated later.

Inside the skill's `{baseDir}`, clone and compile Zkool GraphQL API:

```bash
git clone https://github.com/hhanh00/zkool2.git
cd zkool2/rust
cargo b --bin zkool_graphql --features=graphql --release
```

Check if the binary exists (while still inside {baseDir}/zkool2/rust`):
```bash
ls target/release
```

There should be a binary file called `zkool_graphql`.

Copy this file onto somewhere public, in your `$PATH`, e.g.:

```bash
cp target/release/zkool_graphql /usr/local/bin
```

Start the GraphQL daemon (spawn as a separete job if needed):
```bash
/usr/local/bin/zkool_graphql --db-path $DB_PATH --lwd-url $LWD_URL
```

## Usage

On first use: Check `{baseDir}/wallet.json` for an accountId:

```bash
cat wallet.json | jq '.accountId'
```

If accountId is `-1`, then you'll need to configure the wallet first.

```bash
node {baseDir}/scripts/zkool_cli.js createNewAccount '' 0 3232580 "OpenClaw" ''
```

Get the resulting json `createAccount` value and store it in `{baseDir}/wallet.json`. This will be your accountId. Keep track of this id.

**Do not** create new accounts after wallet initialization.

### Get you Zcash shielded address
Usage:
```
getAddress <accountId>
```

Example:
```bash
node {baseDir}/scripts/zkool_cli.js getAddress 1
```

*Prefer to use the `orchard` address, since it's the fully shielded one, but the other can be used as needed.*

### Synchronize wallet
*When getting the balance, it's important that the wallet is fully synched*
```bash
node {baseDir}/scripts/zkool_cli.js getServerHeight
node {baseDir}/scripts/zkool_cli.js getWalletHeight 1 # replace with accountId
```

If value received from `getServerHeight` is greater than `getWalletHeight`, run wallet sync:
```bash
node {baseDir}/scripts/zkool_cli.js synchronize 1 # replace with accountId
```

### Get wallet balance
*Always synchronize wallet first*

```bash
node {baseDir}/scripts/zkool_cli.js getTotalBalance 1 # replace with accountId
```

You'll see your balance splitted by pool, and the total balance combined.

### Send a transaction
*Always synchronize wallet first*
*Verify your balance first*

recipient format:
```json
[{
    "address": "u1...",
    "amount": 0.0001,
    "memo": "512 characters max"
}]
```
*Values are expressed with fractions of ZEC, i.e. `0.0001`*

Usage:
```
sendTransaction <accountId> <recipient>
```
Example:
```bash
node {baseDir}/scripts/zkool_cli.js sendTransaction 1 '[{"address": "u1...", "amount": 0.0001, "memo": "512 characters max"}]'

```

## Boundaries
This is a live Zcash wallet. Treat it with care.

Never share your seed phrase or private keys with anyone. Anyone who has access to them can control your funds.

All transactions require network fees. Avoid making unnecessary or very small transactions that may waste funds on fees.

Always monitor your wallet balance and transaction history regularly.

Keep secure backups of your seed phrase in a safe, offline location.

You are solely responsible for the security and management of this wallet and its funds.