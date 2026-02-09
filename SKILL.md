---
name: zkool
description: Send and receive Zcash native token $ZEC through the Zkool GraphQL API.
compatibility: Requires git, cargo, rust, npm, node, jq, and an internet connection.
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

A CLI tool to manage a Zcash wallet, using the Zkool GraphQL interface.

## Requirements

This skill needs Rust, git, and Node.js installed on the host machine. Since the CLI output is JSON, it's also recommended to have `jq` installed.

## Setup

Here is a step-by-step guide to clone, build, and run the skill.

### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install -y pkg-config libudev-dev
```

### 2. Set Environment Variables
```bash
export DB_PATH="zkool.db"
export LWD_URL="https://infra.zcashbr.com:9067"
```

### 3. Install npm packages

This skill requires the `graphql-request` npm package.
```bash
npm install
```

### 4. Install Rust

If you don't have Rust installed, follow the official instructions at [rust-lang.org](https://www.rust-lang.org/tools/install).

### 5. Download and Compile Zkool GraphQL API

Create a file at `{baseDir}/wallet.json` to track the wallet's state:
```json
{
    "accountId": -1
}
```
A `-1` value indicates the wallet is not yet configured.

Inside the skill's `{baseDir}`, clone and compile the Zkool GraphQL API:
```bash
git clone https://github.com/hhanh00/zkool2.git
cd zkool2/rust
cargo build --bin zkool_graphql --features=graphql --release
```

After compilation, a `zkool_graphql` binary will be in the `target/release` directory.

Start the GraphQL daemon (spawn as a separate job if needed):
```bash
./target/release/zkool_graphql --db-path $DB_PATH --lwd-url $LWD_URL
```

## Usage

On first use, check `{baseDir}/wallet.json` for an `accountId`:
```bash
jq '.accountId' wallet.json
```

If `accountId` is `-1`, configure the wallet:
```bash
node {baseDir}/scripts/zkool_cli.js createNewAccount '' 0 3232580 "OpenClaw" ''
```
Update `{baseDir}/wallet.json` with the `createAccount` value from the output. This will be your `accountId`.

**Do not** create new accounts after this initial setup.

### Get your Zcash shielded address
Usage: `getAddress <accountId>`
```bash
node {baseDir}/scripts/zkool_cli.js getAddress 1
```
*Prefer the `orchard` address, as it is fully shielded.*

### Synchronize Wallet
*It's important that the wallet is fully synced before checking the balance or sending transactions.*
```bash
node {baseDir}/scripts/zkool_cli.js getServerHeight
node {baseDir}/scripts/zkool_cli.js getWalletHeight 1 # replace with your accountId
```

If the server height is greater than the wallet height, run sync:
```bash
node {baseDir}/scripts/zkool_cli.js synchronize 1 # replace with your accountId
```

### Get Wallet Balance
*Always synchronize your wallet first.*
```bash
node {baseDir}/scripts/zkool_cli.js getTotalBalance 1 # replace with your accountId
```
Your balance will be split by pool, with a combined total.

### Send a Transaction
*Always synchronize your wallet and verify your balance first.*

The recipient must be in this JSON format:
```json
[{
    "address": "u1...",
    "amount": 0.0001,
    "memo": "512 characters max"
}]
```
*Values are expressed in ZEC, i.e. `0.0001`.*

Usage: `sendTransaction <accountId> <recipient_json>`
```bash
node {baseDir}/scripts/zkool_cli.js sendTransaction 1 '[{"address": "u1...", "amount": 0.0001, "memo": "A memo"}]'
```

## Boundaries
This is a live Zcash wallet. Treat it with care.

- Never share your seed phrase or private keys. Anyone with them can control your funds.
- All transactions require network fees. Avoid very small transactions.
- Monitor your wallet balance and transaction history regularly.
- Keep secure, offline backups of your seed phrase.
- You are solely responsible for the security of this wallet and its funds.