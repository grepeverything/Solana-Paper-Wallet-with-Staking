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
On the **networked computer**, download the latest stable release (v2.2.15 at the time of writing) from [GitHub](https://github.com/anza-xyz/agave/releases/latest). Look for the Linux prebuilt binary: `solana-release-x86_64-unknown-linux-gnu.tar.bz2`.

Execute commands from the home directory (`~/`).

```bash
wget https://github.com/anza-xyz/agave/releases/download/v2.2.15/solana-release-x86_64-unknown-linux-gnu.tar.bz2
```

Unpack the Solana CLI and add it to the PATH:

```bash
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
cd solana-release/
export PATH=$PWD/bin:$PATH
```

To persist across terminal sessions, add to `.bashrc`:

```bash
# set PATH environment variable to include the Solana CLI tools
export PATH=$PATH:~/solana-release/bin
```

Verify installation:

```bash
solana --version
```

Expected output:

```
solana-cli 2.2.15 (src:7aff93a2; feat:798020478, client:Agave)
```

## Create the Hot Wallet
The hot wallet resides on the networked computer and should not store large amounts of SOL. Ensure a secure environment (no cameras, private screen) and have paper and a pen ready to record the 12-word seed phrase.

Generate the hot wallet keypair:

```bash
solana-keygen new --no-bip39-passphrase -o hot-wallet.json
```

**Write down the 12-word seed phrase** and store it securely. The `.json` file contains the private key—guard it carefully.

Clear the terminal:

```bash
clear
```

Set up the Solana CLI for **devnet** (test run) or **mainnet-beta** (live run):

*Test Run*:
```bash
solana config set --url devnet -k ~/hot-wallet.json
```

*Live Run*:
```bash
solana config set --url mainnet-beta -k ~/hot-wallet.json
```

Fund the hot wallet:

*Test Run* (airdrop 1.5 SOL):
```bash
solana airdrop 1.5 hot-wallet.json
```

*Live Run*:
Send SOL (amount to stake + 0.05 SOL for fees) from an existing wallet or exchange to the hot wallet's public key. 

Display the public key:

```bash
solana-keygen pubkey hot-wallet.json
# or
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
Install [Tails OS](https://tails.net/install/index.en.html) on the high-quality USB (8GB or larger) and enable [Persistent Storage](https://tails.net/doc/persistent_storage/index.en.html) for the Persistent Folder only. Disable all networking in additional settings and set a strong passphrase (e.g., `Margaret_Thatcher_is_100%_Sexy!`).

Transfer files from the Data USB to the Persistent folder:
- `hot-wallet.json`
- `solana-release-x86_64-unknown-linux-gnu.tar.bz2`
- This guide

Unpack the Solana CLI in the Persistent directory:

```bash
cd Persistent
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
cd solana-release/
export PATH=$PWD/bin:$PATH
cd ..
```

Persistent dotfiles are not enabled, so the PATH will need to be added each time Tails is rebooted

Verify:

```bash
solana --version
```

Expected output:

```
solana-cli 2.2.15 (src:7aff93a2; feat:798020478, client:Agave)
```

## Create the Cold Wallet
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
On the **networked computer**, ensure the correct network and keypair:

```bash
solana config get
```

Copy `cold-wallet-address.txt` from the Data USB to `~/`:

```bash
cp /media/<USER>/<Data_USB>/cold-wallet-address.txt ~
```

Transfer SOL from the hot wallet to the cold wallet:

*Test Run*:
```bash
solana transfer \
--allow-unfunded-recipient \
--from hot-wallet.json \
--fee-payer hot-wallet.json \
$(cat cold-wallet-address.txt) 1.1
```

*Live Run*:
Update the amount to include staking amount + 0.05 SOL for fees.

Check the cold wallet balance:

```bash
solana balance $(cat cold-wallet-address.txt)
```

## Create a Nonce Account
A nonce account is required for offline signing. Learn more [here](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces).

Generate the nonce account keypair:

```bash
solana-keygen new --no-bip39-passphrase -s -o nonce-account.json
```

Fund the nonce account (0.0015 SOL for rent):

```bash
solana create-nonce-account nonce-account.json 0.0015
```

Save the public key and nonce value:

```bash
solana address -k nonce-account.json > nonce-account-address.txt
cp nonce-account-address.txt /media/<USER>/<Data_USB>/
solana nonce nonce-account.json > nonce.txt
cp nonce.txt /media/<USER>/<Data_USB>/
```

Display nonce account details:

```bash
solana nonce-account nonce-account.json
```

## Create the Stake Account
On the **air-gapped computer**, set up the Solana CLI:

```bash
cd Persistent/solana-release/
export PATH=$PWD/bin:$PATH
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

Copy files from the Data USB:

```bash
cp /media/amnesia/<Data_USB>/nonce-account-address.txt ~/Persistent/
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/
```

Generate a stake account keypair:

```bash
solana-keygen new --no-passphrase --derivation-path -s -o stake-account.json
solana address -k stake-account.json > stake-account-address.txt
cp stake-account-address.txt /media/amnesia/<Data_USB>/
```

Create and fund the stake account (offline transaction):

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

*Live Run*:
Update the `<AMOUNT>` as needed.

Save signing pairs to the Data USB (replace `<signing_pair_X>` with actual values):

```bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
echo <signing_pair_3> > /media/amnesia/<Data_USB>/signer3.txt
```

On the **networked computer**, copy `stake-account-address.txt`, `signer1.txt`, `signer2.txt`, and `signer3.txt` to `~/`.

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
--fee-payer hot-wallet.json \
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
solana nonce nonce-account.json > nonce.txt
cp nonce.txt /media/<USER>/<Data_USB>/
```

Select a validator vote account:

*Test Run*:
```bash
solana validators
```

Choose a validator near the bottom of the list and save to `validator.txt`:

```bash
echo <vote_account_address> > validator.txt
cp validator.txt /media/<USER>/<Data_USB>/
```

*Live Run*:
Research validators using [Solana Beach](https://solanabeach.io/), [Validators.app](https://www.validators.app/validators), or [Stakewiz](https://stakewiz.com/). Example validator: `oRAnGeU5h8h2UkvbfnE5cjXnnAa4rBoaxmS4kbFymSe`.

On the **air-gapped computer**, set up the Solana CLI and copy `nonce.txt` and `validator.txt` to the Persistent directory. Delete old `signer*.txt` files from the Data USB.

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

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB.

On the **networked computer**, copy `signer1.txt` and `signer2.txt` to `~/` and submit:

```bash
solana delegate-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--fee-payer hot-wallet.json \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt) \
$(cat validator.txt)
```

View the stake account (activation takes a few days):

```bash
solana stake-account $(cat stake-account-address.txt)
```

Check your wallet on [Solscan](https://solscan.io/?cluster=devnet) (devnet) or [Solscan](https://solscan.io/) (mainnet-beta) using the cold wallet public key.

## Deactivate Stake
On the **networked computer**, advance the nonce:

```bash
solana new-nonce nonce-account.json
solana nonce nonce-account.json > nonce.txt
cp nonce.txt /media/<USER>/<Data_USB>/
```

On the **air-gapped computer**, set up the Solana CLI and copy `nonce.txt`. Create the offline transaction:

```bash
solana deactivate-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json
```

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB.

On the **networked computer**, copy `signer1.txt` and `signer2.txt` and submit:

```bash
solana deactivate-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--fee-payer hot-wallet.json \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt)
```

View the stake account (deactivation takes a few days):

```bash
solana stake-account $(cat stake-account-address.txt)
```

## Withdraw Stake
On the **networked computer**, verify deactivation and note the balance:

```bash
solana stake-account $(cat stake-account-address.txt)
```

Save the balance (numerical value only) to `balance.txt`:

```bash
echo <balance> > balance.txt
cp balance.txt /media/<USER>/<Data_USB>/
```

Advance the nonce:

```bash
solana new-nonce nonce-account.json
solana nonce nonce-account.json > nonce.txt
cp nonce.txt /media/<USER>/<Data_USB>/
```

On the **air-gapped computer**, set up the Solana CLI and copy `nonce.txt` and `balance.txt`to the Persistent directory.
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

Save signing pairs to `signer1.txt` and `signer2.txt` on the Data USB.

On the **networked computer**, copy `signer1.txt` and `signer2.txt` and submit:

```bash
solana withdraw-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--withdraw-authority $(cat cold-wallet-address.txt) \
--fee-payer hot-wallet.json \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt) \
$(cat cold-wallet-address.txt) $(cat balance.txt)
```

Check balances:

```bash
solana balance $(cat cold-wallet-address.txt)
solana stake-account $(cat stake-account-address.txt)
```

## Recover the Cold Wallet
**Warning**: Recovering in a software wallet makes it a hot wallet. Use [Solflare](https://www.solflare.com/):
1. Select "I already have a wallet" and choose the 24-word recovery phrase option.
2. Disable networking while entering the seed phrase and wallet password.
3. For devnet, switch to Devnet in settings if no balance is shown.
4. Use derivation path `m/44'/501'/0'/0'` to match `cold-wallet-address.txt`.

## Conclusion
This guide covers creating a secure Solana paper wallet with staking using offline signing. Try transferring the cold wallet balance back to the hot wallet as an exercise.

Consider a small SOL donation to the author:

An3xM6xCLmBEn3NXjYoggvWQToL9Lmx5mH2USchUpyNz
![QR Code](https://github.com/grepeverything/Solana-Paper-Wallet-with-Staking/blob/main/SOL_Donation_QR_Code.png)

