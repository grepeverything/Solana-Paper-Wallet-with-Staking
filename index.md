---
layout: default
title: Solana Paper Wallet with Staking
description: A guide for creating an air-gapped Solana paper wallet with staking support
google-site-verification: DMBFlxWnLAuyxAWcP0A3aSyRHCAj3AB2u7wghYOIHJ4
keywords: Solana, paper wallet, cryptocurrency, staking, blockchain, tutorial, guide, air-gapped, nonce, nonce-account, stake-account, cold wallet, offline signing, OpSec, linux, solana-cli, keypair, private keys, public keys, hot wallet, backpack.app
---
# Solana Paper Wallet with Staking

This guide provides step-by-step instructions to create a Solana paper wallet and a stake account controlled by the paper wallet. All sensitive operations are performed on an air-gapped computer to ensure the seed phrases and private keys remain offline. Transactions are constructed using offline signing via the Solana CLI, and the paper wallet is designed for easy recovery with software wallets. This guide assumes a Linux environment.

## Introduction to the Solana CLI
For more details, see the [Solana CLI documentation](https://docs.anza.xyz/cli/intro).

## Requirements
- **Air-gapped computer** with Solana CLI installed for wallet creation and offline signing
- **Networked computer** with Solana CLI installed to broadcast transactions (one computer can serve both roles with extra effort)
- **High-quality USB stick** (8GB minimum) for Tails OS (Live USB)
- **Second USB stick** for data transfer (Data USB, optionally encrypted for extra security)
- Comfort with the Linux command line
- Understanding of operational security (OpSec)
- A strong desire to keep your Solana safe

**WARNING**: This process is technical and requires learning about Solana transactions. For a simpler alternative, consider a hardware wallet for cold storage with staking.

Practice on **devnet** (using air-dropped test SOL) before attempting on **mainnet-beta** with real SOL.

## Download and Install the Solana CLI
On the **networked computer**, download the latest stable release (v2.2.16 at the time of writing) from [GitHub](https://github.com/anza-xyz/agave/releases/latest). Look for the Linux prebuilt binary: `solana-release-x86_64-unknown-linux-gnu.tar.bz2`.

Execute commands from the home directory (`~/`) or preferred working directory:

```bash
wget https://github.com/anza-xyz/agave/releases/download/v2.2.16/solana-release-x86_64-unknown-linux-gnu.tar.bz2
```

Unpack the Solana CLI and add it to the PATH:

```bash
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
```
```bash
cd solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

To persist across terminal sessions, add PATH to `.bashrc`:

```bash
# set PATH environment variable to include the Solana CLI tools
export PATH=$PATH:~/<WORKING_DIRECTORY>/solana-release/bin
```

Verify installation:

```bash
solana --version
```

Expected output:

```bash
solana-cli 2.2.16 (src:851b7526; feat:3073396398, client:Agave)
```

## Create the Hot Wallet
The hot wallet resides on the networked computer and should not store large amounts of SOL. Ensure a secure environment (no cameras, private screen) and have paper and a pen ready to record the 12-word seed phrase.

Generate the hot wallet keypair:

```bash
solana-keygen new \
--derivation-path \
--no-bip39-passphrase \
-o hot-wallet.json
```

**Write down the 12-word seed phrase** and store it securely. The `.json` file contains the private key—guard it carefully.

Clear the terminal:

```bash
clear
```

Set up the Solana CLI for **devnet** (test run) or **mainnet-beta** (live run), and assign hot-wallet.json as the client keypair.:

*Test Run*:
```bash
solana config set --url devnet -k hot-wallet.json
```

*Live Run*:
```bash
solana config set --url mainnet-beta -k hot-wallet.json
```

In this guide the hot wallet is going to serve as the nonce authority and the fee payer for the transactions. It only requires a small amount of SOL.


Fund the hot wallet:

*Test Run* (airdrop 0.005 SOL):
```bash
solana airdrop 0.005 hot-wallet.json
```

*Live Run*:
Send SOL from an existing wallet to the hot wallet's public key. 

Display the public key:

```bash
solana-keygen pubkey hot-wallet.json
```
or
```bash
solana address -k hot-wallet.json
```

Check the balance:

```bash
solana balance hot-wallet.json
```

Copy to the Data USB:
- `hot-wallet.json`
- `solana-release-x86_64-unknown-linux-gnu.tar.bz2`
- A copy of this guide

## Create the Live USB
Install [Tails OS](https://tails.net/install/index.en.html) on the high-quality USB (8GB or larger) and boot the **air-gapped computer** from the Live USB.
Enable [Persistent Storage](https://tails.net/doc/persistent_storage/index.en.html) for the Persistent Folder only. Disable all networking in additional settings and set a strong passphrase (e.g., `Margaret_Thatcher_is_100%_Sexy!`).

Transfer files from the Data USB to the Persistent folder:
- `hot-wallet.json`
- `solana-release-x86_64-unknown-linux-gnu.tar.bz2`
- This guide

Unpack the Solana CLI in the Persistent directory:

```bash
cd Persistent
```
```bash
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
```
```bash
cd solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

Persistent dotfiles are not enabled, so the PATH will need to be added each time Tails is rebooted.

Verify:

```bash
solana --version
```

Expected output:

```
solana-cli 2.2.16 (src:851b7526; feat:3073396398, client:Agave)
```

## Create the Cold Wallet (Paper Wallet)
Generate the cold wallet on the air-gapped computer with a 24-word seed phrase for extra security:

```bash
solana-keygen new \
--derivation-path \
--no-bip39-passphrase \
-o cold-wallet.json \
--word-count 24
```

**Write down the 24-word seed phrase** on paper and store two copies in separate, secure locations. Do not store digitally.

Clear the terminal:

```bash
clear
```

Save the public key to `cold-wallet-address.txt` and copy to the Data USB:

```bash
solana address -k cold-wallet.json > cold-wallet-address.txt
```
```bash
cp cold-wallet-address.txt /media/amnesia/<Data_USB>/
```

Verify the seed phrase:

```bash
solana-keygen pubkey 'prompt://?key=0/0'
```

Carefully type the 24-word seed phrase (not visible while typing). Press `Enter` at the passphrase prompt. The displayed public key should match `cold-wallet-address.txt`:

```bash
cat cold-wallet-address.txt
```

Verify the seed phrase controls the private key:

```bash
solana-keygen verify $(cat cold-wallet-address.txt) 'prompt://?key=0/0'
```

## Fund the Cold Wallet
On the **networked computer**, ensure the correct URL and keypair are set:

```bash
solana config get
```

Copy `cold-wallet-address.txt` from the Data USB to `~/`or preferred working directory:

```bash
cp /media/<USER>/<Data_USB>/cold-wallet-address.txt ~
```

Fund the cold wallet (amount to be staked plus 0.005 SOL for any future fees):

*Test Run* (airdrop 1.005 SOL):

```bash
solana airdrop 1.005 $(cat cold-wallet-address.txt)
```

*Live Run*:
Send the SOL from an existing wallet or an exchange to the cold wallet address.

Check the cold wallet balance:

```bash
solana balance $(cat cold-wallet-address.txt)
```

## Create a Nonce Account
A nonce account is required for offline signing. Learn more [here](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces).

On the **networked computer**, generate the nonce account keypair:

```bash
solana-keygen new --no-bip39-passphrase -s -o nonce-account.json
```

The hot wallet will fund the nonce account and be delegated as the nonce authority.

Create and fund the nonce account (0.0015 SOL for rent):

```bash
solana create-nonce-account nonce-account.json 0.0015
```


Save the public key and copy to the Data USB:

```bash
solana address -k nonce-account.json > nonce-account-address.txt
```
```bash
cp nonce-account-address.txt /media/<USER>/<Data_USB>/
```
Save the nonce and copy to the Data USB:

```bash
solana nonce nonce-account.json > nonce.txt
```
```bash
cp nonce.txt /media/<USER>/<Data_USB>/
```

Display nonce account details:

```bash
solana nonce-account nonce-account.json
```

The nonce-account.json file or the nonce account address will be required to recover the rent. Store them somewhere safe.

## Create the Stake Account
On the **air-gapped computer**, set up the Solana CLI:

```bash
cd Persistent/solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

*Test Run*:
```bash
solana config set -u devnet -k cold-wallet.json
```

*Live Run*:
```bash
solana config set -u mainnet-beta -k cold-wallet.json
```

Copy `nonce-account-address.txt` and `nonce.txt` from the Data USB to the Persistent directory:

```bash
cp /media/amnesia/<Data_USB>/nonce-account-address.txt ~/Persistent/
```
```bash
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/
```

Generate the stake account keypair and copy the address to the Data USB:

```bash
solana-keygen new --no-passphrase --derivation-path -s -o stake-account.json
```
```bash
solana address -k stake-account.json > stake-account-address.txt
```
```bash
cp stake-account-address.txt /media/amnesia/<Data_USB>/
```

The stake-account.json file or the stake account address will be required to deactivate and withdraw the stake. Store them somewhere safe.

Create and fund the stake account (offline signing):

*Test Run*:
```bash
solana create-stake-account \
--sign-only \
--blockhash $(cat nonce.txt) \
--fee-payer hot-wallet.json \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
stake-account.json 1
```
**Note** The amount sent to the stake account comes from the cold wallet as it is set as the keypair in the config.

*Live Run*:
Update the `<AMOUNT>` as needed. Leave a small amount in the cold wallet for future fees (0.005).

Save signing pairs to the Data USB (replace `<signing_pair_X>` with actual values):

```bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
```
```bash
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
```
```bash
echo <signing_pair_3> > /media/amnesia/<Data_USB>/signer3.txt
```

On the **networked computer**, copy `stake-account-address.txt`, `signer1.txt`, `signer2.txt`, and `signer3.txt` to `~/` or preferred working directory:

```bash
cp /media/<USER>/<Data_USB>/stake-account-address.txt ~
```
```bash
cp /media/<USER>/<Data_USB>/signer* ~
```

Submit the transaction:

*Test Run*:
```bash
solana create-stake-account \
--blockhash $(cat nonce.txt) \
--from $(cat cold-wallet-address.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--withdraw-authority $(cat cold-wallet-address.txt) \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
--signer $(cat signer3.txt) \
$(cat stake-account-address.txt) 1
```

*Live Run*:
Update the `<AMOUNT>` as needed.

View the stake account:

```bash
solana stake-account $(cat stake-account-address.txt)
```

## Delegate the Stake
Advance the nonce on the **networked computer**:

```bash
solana new-nonce nonce-account.json
```
```bash
solana nonce nonce-account.json > nonce.txt
```
```bash
cp nonce.txt /media/<USER>/<Data_USB>/
```

Select a validator vote account:

*Test Run*:
```bash
solana validators
```

Choose a validator near the bottom of the list and save the vote account address to `validator.txt`:

```bash
echo <vote_account_address> > validator.txt
```
```bash
cp validator.txt /media/<USER>/<Data_USB>/
```

*Live Run*:
Research validators using [Solana Beach](https://solanabeach.io/), [Validators.app](https://www.validators.app/validators), or [Stakewiz](https://stakewiz.com/). Example validator: `oRAnGeU5h8h2UkvbfnE5cjXnnAa4rBoaxmS4kbFymSe`.

On the **air-gapped computer**, set up the Solana CLI:

```bash
cd Persistent/solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

*Test Run*:
```bash
solana config set -u devnet -k cold-wallet.json
```

*Live Run*:
```bash
solana config set -u mainnet-beta -k cold-wallet.json
```

Copy `nonce.txt` and `validator.txt` to the Persistent directory. Delete old `signer*.txt` files from the Data USB:

```bash
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/
```
```bash
cp /media/amnesia/<Data_USB>/validator.txt ~/Persistent/
```
```bash
rm /media/amnesia/<Data_USB>/signer*
```

Delegate the stake:

```bash
solana delegate-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json \
$(cat validator.txt)
```

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB:

```bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
```
```bash
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
```

On the **networked computer**, copy `signer1.txt` and `signer2.txt` to `~/` or preferred working directory:

```bash
cp /media/<USER>/<Data_USB>/signer* ~
```

Submit the transaction:

```bash
solana delegate-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt) \
$(cat validator.txt)
```

View the stake account (activation takes a few days):

```bash
solana stake-account $(cat stake-account-address.txt)
```

View the wallet on [Solscan](https://solscan.io/?cluster=devnet) (devnet) or [Solscan](https://solscan.io/) (mainnet-beta) using the cold wallet public key.

**Note** If performing the *Test Run*, there is no need to wait for the stake to activate before continuing to the next steps.

## Deactivate Stake
On the **networked computer**, advance the nonce:

```bash
solana new-nonce nonce-account.json
```
```bash
solana nonce nonce-account.json > nonce.txt
```
```bash
cp nonce.txt /media/<USER>/<Data_USB>/
```

On the **air-gapped computer**, set up the Solana CLI:

```bash
cd Persistent/solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

*Test Run*:
```bash
solana config set -u devnet -k cold-wallet.json
```

*Live Run*:
```bash
solana config set -u mainnet-beta -k cold-wallet.json
```

Copy `nonce.txt` to the Persistent directory:

```bash
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/
```

Create the offline transaction:

```bash
solana deactivate-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json
```

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB:

```bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
```
```bash
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
```

On the **networked computer**, copy `signer1.txt` and `signer2.txt` to `~/` or preferred working directory:

```bash
cp /media/<USER>/<Data_USB>/signer* ~
```

Submit the transaction:

```bash
solana deactivate-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt)
```

View the stake account (deactivation takes a few days):

```bash
solana stake-account $(cat stake-account-address.txt)
```

**Note** If performing the *Test Run*, there is no need to wait for the stake to deactivate before continuing to the next steps.

## Withdraw Stake
On the **networked computer**, verify deactivation and note the balance:

```bash
solana stake-account $(cat stake-account-address.txt)
```

Save the balance (numerical value only) to `balance.txt`:

```bash
echo <balance> > balance.txt
```
```bash
cp balance.txt /media/<USER>/<Data_USB>/
```

Advance the nonce:

```bash
solana new-nonce nonce-account.json
```
```bash
solana nonce nonce-account.json > nonce.txt
```
```bash
cp nonce.txt /media/<USER>/<Data_USB>/
```

On the **air-gapped computer**, set up the Solana CLI:

```bash
cd Persistent/solana-release/
```
```bash
export PATH=$PWD/bin:$PATH
```
```bash
cd ..
```

*Test Run*:
```bash
solana config set -u devnet -k cold-wallet.json
```

*Live Run*:
```bash
solana config set -u mainnet-beta -k cold-wallet.json
```

Copy `nonce.txt` and `balance.txt`to the Persistent directory:

```bash
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/
```
```bash
cp /media/amnesia/<Data_USB>/balance.txt ~/Persistent/
```

Create the offline transaction:

```bash
solana withdraw-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json \
cold-wallet.json $(cat balance.txt)
```

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB:

```bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
```
```bash
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
```

On the **networked computer**, copy `signer1.txt` and `signer2.txt` to `~/` or preferred working directory:

```bash
cp /media/<USER>/<Data_USB>/signer* ~
```

Submit the transaction:

```bash
solana withdraw-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--withdraw-authority $(cat cold-wallet-address.txt) \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt) \
$(cat cold-wallet-address.txt) $(cat balance.txt)
```

Check balances:

```bash
solana balance $(cat cold-wallet-address.txt)
```
```bash
solana balance $(cat stake-account-address.txt)
```

## Withdraw from Nonce Account
The nonce account can remain active for future transactions or the rent amount can be recovered.

On the **networked computer**, recover rent amount to the hot wallet (nonce-authority):

```bash
solana withdraw-from-nonce-account nonce-account.json hot-wallet.json 0.0015
```


Check balances:

```bash
solana balance nonce-account.json
```
```bash
solana balance hot-wallet.json
```


## Software Wallets
A view-only wallet can be created using the [Backpack](https://backpack.app/) app.

If viewing a devnet wallet, switch the RPC Connection to use `https://api.devnet.solana.com` in settings.

If viewing a mainnet wallet, leave the RPC Connection as the default.

A software wallet can also be used to recover the paper wallet from the seed phrase.

**Warning**: Recovering in a software wallet makes it a hot wallet.


## Conclusion
This guide covers creating a secure Solana paper wallet with staking using offline signing. There are many more uses for the Solana CLI outside the scope of this guide.
The knowledge learned here should be a great base for further exploration.

Test the knowledge learned by constructing a transaction to transfer SOL from the cold wallet to the hot wallet. (Solution included in the Bonus Section).

**Consider a small SOL donation to the author:**

An3xM6xCLmBEn3NXjYoggvWQToL9Lmx5mH2USchUpyNz

![QR Code](https://raw.githubusercontent.com/grepeverything/Solana-Paper-Wallet-with-Staking/main/SOL_Donation_QR_Code.png)


## Bonus Section
Some additional items that may be of interest.

## Recover Keypair from Seed Phrase

**Air-gapped Computer**
```bash
solana-keygen recover -o recovered-wallet.json 'prompt://?key=0/0'
```


## Transfer from Cold Wallet to Hot Wallet

**Air-gapped Compuer**

Assuming `cold-wallet.json` is set as the client keypair:

```bash
solana transfer \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
hot-wallet.json <AMOUNT>
```

**Networked Computer**

Assuming `hot-wallet.json` is set as the client keypair:

```bash
solana transfer \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--from $(cat cold-wallet-address.txt)
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
hot-wallet.json <AMOUNT>
```


## Merge Stake Accounts
Learn about merging stake accounts: 
[Merging Stake Accounts](https://solana.com/docs/references/staking/stake-accounts#merging-stake-accounts)

**Air-gapped Computer**

Assuming `stake-account.json` is set as the client keypair:

```bash
solana merge-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
<STAKE_ACCOUNT_ADDRESS> <SOURCE_STAKE_ACCOUNT_ADDRESS>
```

**Networked Computer**

Assuming `hot-wallet.json` is set as the client keypair:

```bash
solana merge-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat staking-account-address.txt) \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
<STAKE_ACCOUNT_ADDRESS> <SOURCE_STAKE_ACCOUNT_ADDRESS>
```

More to come?





