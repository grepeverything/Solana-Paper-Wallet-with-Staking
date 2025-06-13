Solana Paper Wallet with Staking

This document will provide the necessary steps to create a Solana paper wallet and a stake account that will be controlled by the paper wallet.
All the sensitive parts of this process will be done on an air-gapped computer. The seed phrases and private keys of the paper wallet will never be exposed to the internet. All sensitive transactions will be constructed on the air-gapped computer by using offline signing. The whole process will be done using the Solana CLI environment but the paper wallet will be created in a manner that makes recovering it with a software wallet an easy task. 
This guide assumes a Linux environment.

Introduction to the Solana CLI
https://docs.anza.xyz/cli/intro

Requirements
Air-gapped computer with Solana CLI installed for wallet creation and offline signing
Networked computer with Solana CLI installed to broadcast transactions
(one computer can serve both roles, with a little more effort)
A high quality USB stick (8GB minimum) for Tails OS (Live USB)
A second USB stick for data transfer (Data USB)
(The Data USB can be encrypted for extra security)
Comfort with the Linux command line
An understanding of the importance of OpSec
A strong desire to keep your Solana safe

WARNING! This is a little geeky. You will learn a lot about how Solana transactions work. Even if you don’t use this method for real world Solana transactions, it’s worth going through the test run which is done on devnet using air-dropped test SOL.
If you just don’t have the patience, then I recommend a hardware wallet for your cold storage with staking needs.

Going through all the steps on devnet does not require any real SOL. Once you have completed all the steps in the test environment, then you can repeat the steps on mainnet-beta with real Solana.

Download and install the Solana CLI

On the networked computer navigate to https://github.com/anza-xyz/agave/releases/latest
Download the latest stable release (v2.2.15 at this time).
It will say “This is a stable release suitable for use on Mainnet Beta.”
We want the Linux prebuilt binary solana-release-x86_64-unknown-linux-gnu.tar.bz2 

To keep things simple, execute the commands from the home directory.

bash
wget https://github.com/anza-xyz/agave/releases/download/v2.2.15/solana-release-x86_64-unknown-linux-gnu.tar.bz2

Unpack the Solana CLI on the networked computer and add the tools to the PATH

bash
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
cd solana-release/
export PATH=$PWD/bin:$PATH

To make it persist across terminal sessions and reboots, add the following to .bashrc:

# Set PATH environment variable to include the Solana CLI tools
export PATH=$PATH:~/solana-release/bin

Verify it’s working

bash
solana –version
solana-cli 2.2.15 (src:7aff93a2; feat:798020478, client:Agave)


 

The “solana-keygen new” command is used to create the seed phrase and keypair for the hot wallet. “solana-keygen new --help” will list the options with a description. Use the --help option with any new command to review the options. The following commands assume the working directory is ~/

Make sure you are in a secure environment with no cameras and where no one can see your screen. The hot wallet keypair will reside on the networked computer and therefore should not be used to store a large amount of SOL. We will write down the seed phrase as a backup. Have paper and a pen handy to write down the 12 word seed phrase.

Generate the hot wallet keypair

bash
solana-keygen new --no-bip39-passphrase -o hot-wallet.json

The .json file contains the private key. This should be closely guarded as anyone with access to it can take control of your funds.

Write down the 12 word seed phrase and store in a safe place.

Clear the terminal


bash
clear

Set up the Solana CLI environment to use devnet and assign hot-wallet.json as the default keypair.
(Remember to switch back to mainnet-beta when it comes time to do this with real SOL.)

*Test Run
bash 
solana config set --url devnet -k ~/hot-wallet.json

*Live Run
bash
solana config set --url mainnet-beta -k ~/hot-wallet.json

Fund the account. For the test run, airdrop 1.5 SOL

*Test Run
bash
solana airdrop 1.5 hot-wallet.json

*Live Run
The first time you do the live run with real SOL, I would still only recommend using a small amount until you are comfortable with the entire process. You are fully responsible for your own funds!
For the live run the hot wallet will need to be funded from an account that already has some SOL. This could be from an existing wallet or an exchange.
Send the amount to be staked and a small amount for fees (0.05) to the hot wallet pubkey (address).

Display the pubkey

bash
solana-keygen pubkey hot-wallet.json
 
or

bash
solana address -k hot-wallet.json

We can display the balance of the hot wallet account

bash
solana balance hot-wallet.json

Copy the following files to the Data USB
hot-wallet.json
solana-release-x86_64-unknown-linux-gnu.tar.bz2
* a copy of this guide

Create the Live USB

Install Tails on the high quality USB stick (8GB or larger)
https://tails.net/install/index.en.html

Enable Persistent Storage
https://tails.net/doc/persistent_storage/index.en.html

Boot the air-gapped computer from the Live USB

Enable Persistent Storage for:
- Persistent Folder

Do not use persistence for anything else. This Live USB should only be used for this purpose. Do not use it to go online or any other tasks.
 
In additional settings disable all networking.

Make sure to choose a strong passphrase to protect the persistent storage.
It should be something easy to remember, but impossible for someone to guess.
eg. Margaret_Thatcher_is_100%_Sexy!

Once Tails is started, transfer all the files from the Data USB to the Persistent folder.
hot-wallet.json
solana-release-x86_64-unknown-linux-gnu.tar.bz2
* a copy of this guide

Open the terminal and change to the Persistent directory. Unpack and prepare the Solana CLI as before and then switch back to the Persistent directory

bash
tar jxf solana-release-x86_64-unknown-linux-gnu.tar.bz2
cd solana-release/
export PATH=$PWD/bin:$PATH
cd ..

The PATH variable will not persist across terminal sessions or reboots.

To make it persist across terminal sessions, add the following to .bashrc:

# set PATH environment variable to include the Solana CLI tools
export PATH=$PATH:~/Persistent/solana-release/bin

(Or keep the terminal open for the entire process)

Persistent dotfiles are not enabled, so add the PATH variable each time Tails is rebooted.

Test that it’s working:

bash
solana --version

bash output
solana-cli 2.2.15 (src:7aff93a2; feat:798020478, client:Agave



First display the “solana-keygen new” options

bash
solana-keygen new --help

Use the following options:

--derivation-path [<DERIVATION_PATH>...]
            Derivation path. All indexes will be promoted to hardened. If arg is not presented then
            derivation path will not be used. If arg is presented with empty DERIVATION_PATH          value
            then m/44'/501'/0'/0' will be used

--no-bip39-passphrase
            Do not prompt for a BIP39 passphrase

-o, --outfile <FILEPATH>
            Path to generated file

--word-count <NUMBER>
            Specify the number of words that will be present in the generated seed phrase [default: 12] [possible values: 12, 15, 18, 21, 24]

Using the --derivation-path option with an empty value will create a wallet that can be easily recovered using modern Solana software wallets (m/44'/501'/0'/0').
Using the --no-bip39-passphrase option will suppress the prompt for a passphrase.
(some software wallets do not support passphrases)
Using the --outfile option, the name and path to the generated keypair file can be specified.
Using the --word-count option, the length of the seed phrase can be specified. 24 will be used in this case, for extra security.

Have paper and a pen handy to write down the 24 word seed phrase for safe offline storage. Do not store it on any digital media. Make sure you are in a secure environment with no cameras and where no one can see your screen.

Create the wallet

bash
solana-keygen new \
--derivation-path \
--no-bip39-passphrase \
-o cold-wallet.json \
--word-count 24

Write down the 24 word seed phrase on paper. Store 2 copies of the seed phrase in separate,  secure locations.

Clear the terminal

bash
clear

Save the pubkey to a text file named cold-wallet-address.txt in the Persistent directory and copy it to the Data USB.

bash
solana address -k cold-wallet.json > cold-wallet-address.txt
cp cold-wallet-address.txt /media/amnesia/<Data_USB>/

Test the seed phrase was written down correctly by using it to generate the pubkey

bash
solana-keygen pubkey 'prompt://?key=0/0'

bash output
[pubkey recovery] seed phrase: 

Very carefully type the 24 word seed phrase in order and with spaces between the words.
You will not be able to see what you enter, so type slowly and carefully to avoid mistakes.

bash output
[pubkey recovery] If this seed phrase has an associated passphrase, enter it now. Otherwise, press ENTER to continue:

Press enter to continue.

The pubkey displayed should match the one saved to the text file.

bash
cat cold-wallet-address.txt

Also verify that the seed phrase controls the private key

bash
solana-keygen verify $(cat cold-wallet-address.txt) 'prompt://?key=0/0'

Carefully type the seed phrase again.

Looking good!

Switch to the networked computer

Check that the Solana CLI environment is set for devnet for the test run or mainnet-beta for the live run and the keypair path is set to the hot wallet.

bash
solana config get

Fund the Cold Wallet from the Hot Wallet

First copy the cold-wallet-address.txt file from the Data USB to the working directory.

bash
cp /media/<USER>/<Data_USB>/cold-wallet-address.txt ~

Have a look at the solana transfer command options

bash
solana transfer --help

The ones used for this transaction are:

--allow-unfunded-recipient       Complete the transfer even if the recipient address is not funded
--from <FROM_ADDRESS>
            Source account of funds [default: cli config keypair]. Address is one of:
              * a base58-encoded public key
              * a path to a keypair file
              * a hyphen; signals a JSON-encoded keypair on stdin
              * the 'ASK' keyword; to recover a keypair via its seed phrase
              * a hardware wallet keypair URL (i.e. usb://ledger)
--fee-payer <KEYPAIR>
            Specify the fee-payer account. This may be a keypair file, the ASK keyword 
            or the pubkey of an offline signer, provided an appropriate --signer argument 
            is also passed. Defaults to the client keypair.
<RECIPIENT_ADDRESS>    Account of recipient. Address is one of:
                             * a base58-encoded public key
                             * a path to a keypair file
                             * a hyphen; signals a JSON-encoded keypair on stdin
                             * the 'ASK' keyword; to recover a keypair via its seed phrase
                             * a hardware wallet keypair URL (i.e. usb://ledger)
 <AMOUNT>               The amount to send, in SOL; accepts keyword ALL

Do not send all the fund from the hot wallet as a small amount will be needed for transaction fees.

Send from the hot wallet to the cold wallet

*Test Run
bash
solana transfer \
--allow-unfunded-recipient \
--from hot-wallet.json \
--fee-payer hot-wallet.json \
$(cat cold-wallet-address.txt) 1.1

*Live Run
Change <AMOUNT> to the amount that will be staked plus a small amount for future fees (0.05 SOL).

In this case the --from and--key-payer options are not required because the default is to use the keypair that is set in the config (hot-wallet.json).
They are included here as a visual.

Check the cold wallet balance

bash
solana balance $(cat cold-wallet-address.txt)

Create a Nonce Account

A nonce account will be required when constructing offline signing transactions.
Learn more here
https://solana.com/developers/guides/advanced/introduction-to-durable-nonces

The hot wallet will act as the nonce authority.

Generate the keypair for the nonce account

bash
solana-keygen new --no-bip39-passphrase -s -o nonce-account.json


Now fund the nonce account. The hot wallet will automatically be assigned as the nonce authority and will fund the account. Only a small amount of SOL is required for rent

bash
solana create-nonce-account \
nonce-account.json 0.0015

The pubkey will be needed on the air-gapped computer. Create a text file named nonce-account-address.txt in the working directory and copy it to the Data USB

bash
solana address -k nonce-account.json > nonce-account-address.txt
cp nonce-account-address.txt /media/<USER>/<Data_USB>/

Display information about the nonce-account

bash
solana nonce-account nonce-account.json

Output the current nonce value

bash
solana nonce nonce-account.json

Save the nonce in a file called nonce.txt in the working directory and copy it to the Data USB

bash
solana nonce nonce-account.json > nonce.txt
cp nonce.txt /media/<USER><Data_USB>/

We will get back to the exciting world of nonces later.


Back to the air-gapped computer

If Tails was rebooted, add the Solana tools to the PATH and set up the Solana CLI environment to use devnet for the test run or mainnet-beta for the live run and set the cold-wallet as the keypair

bash
cd Persistent/solana-release/
export PATH=$PWD/bin:$PATH
cd ..

*Test Run
bash
solana config set -u devnet -k cold-wallet.json

*Live Run
bash
solana config set -u mainnet-beta -k cold-wallet.json

Copy the following from the Data USB to the Persistent directory.

bash
cp /media/amnesia/<Data_USB>/nonce-account-address.txt ~/Persistent/
cp /media/amnesia/<Data_USB>/nonce.txt ~/Persistent/

Create the Stake Account

Learn more about stake accounts here
https://docs.anza.xyz/cli/examples/delegate-stake
(Open the link on the networked computer, not the air-gapped computer!)

First generate a keypair and pubkey for the stake account

bash
solana-keygen new --no-passphrase --derivation-path -s -o stake-account.json
 
Copy the pubkey to a text file named stake-account-address.txt in the Persistent directory

bash
solana address -k stake-account.json > stake-account-address.txt
cat stake-account-address.txt

Copy this file to the Data USB.

bash
cp stake-account-address.txt /media/amnesia/<Data_USB>/

Check out the solana create-stake-account options

bash
solana create-stake-account --help

Create the stake account and generate an offline transaction to fund it.

*Test Run
bash 
solana create-stake-account \
--sign-only \
--blockhash $(cat nonce.txt) \
--fee-payer hot-wallet.json \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
stake-account.json 1

*Live Run
Update the <AMOUNT> ARG as required

This will generate 3 signing pairs. One for the hot wallet, one for the stake account and one for the cold wallet.

Save each signing pair to the Data USB as a text file

example:
bash
echo <signing_pair_1> > /media/amnesia/<Data_USB>/signer1.txt
echo <signing_pair_2> > /media/amnesia/<Data_USB>/signer2.txt
echo <signing_pair_3> > /media/amnesia/<Data_USB>/signer3.txt

Switch back to the networked computer

Copy the newly created files from the Data USB to the working directory
stake-account-address.txt
signer1.txt
signer2.txt
signer3.txt

Submit the transaction to the network

*Test Run
bash 
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

*Live Run
Update the <AMOUNT> ARG as required

View the stake account

bash
solana stake-account $(cat stake-account-address.txt)

Delegate the Stake

Next, delegate the stake to a validator. This will involve creating a new offline transaction so a new nonce will be required.

Advance the nonce

bash
solana new-nonce nonce-account.json

Update the nonce.txt file in the working directory and copy it to the Data USB

bash
solana nonce nonce-account.json > nonce.txt

bash
cp nonce.txt /media/<USER>/Data_USB>/

The vote account address of the chosen validator is required. For the devnet test run the selection is not that important. Run the following command and choose one near the bottom of the list.

*Test Run
bash
solana validators

Save the vote account address to a text file in the working directory named validator.txt and copy it to the Data USB.


*Live Run
When it comes time to to delegate real Solana, you will have to do some research about choosing a good validator. Here are some resources to help you choose
https://solanabeach.io/
https://www.validators.app/validators
https://stakewiz.com/

(I like to use Orangefin Ventures)
https://stakewiz.com/validator/oRAnGeU5h8h2UkvbfnE5cjXnnAa4rBoaxmS4kbFymSe
Vote Account: oRAnGeU5h8h2UkvbfnE5cjXnnAa4rBoaxmS4kbFymSe

Back to the air-gapped computer


If tails was rebooted, add the Solana CLI tools to the PATH and
set up Solana CLI environment to devnet for the test run or mainnet-beta for the live run and set the cold-wallet.json as the key

bash
cd Persistent/solana-release/
export PATH=$PWD/bin:$PATH
cd ..

*Test Run
bash
solana config set -u devnet -k cold-wallet.json

*Live Run
solana config set -u mainnet-beta -k cold-wallet.json

Copy the new nonce.txt and validator.txt files from the Data USB to the Persistent directory.
Also delete the signer*.txt files from the Data USB to avoid confusion as new ones will be created.

Delegate the stake

bash
solana delegate-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json \
$(cat validator.txt)

Save each signing pair to their own text file.
signer1.txt
signer2.txt

Copy them to the Data USB

And back to the networked computer

Copy the new signer1.txt and signer2.txt files to the working directory

Submit the transaction to the network

bash
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

Good job!

View the stake account information

bash
solana stake-account $(cat stake-account-address.txt)

It will take a couple of days until the stake is active.

You can also view your wallet using a block explorer like solscan.io
Search for your cold wallet pubkey (address)
https://solscan.io/?cluster=devnet (for devnet)
https://solscan.io/ (for mainnet-beta)

Deactivate Stake

The offline signing process can be used to deactivate the stake.

On the networked computer:

Advance the nonce

bash
solana new-nonce nonce-account.json

Update the nonce.txt file and save to the Data USB

bash
solana nonce nonce-account.json > nonce.txt

Copy the new nonce.txt file to the Data USB.

Switch to the air-gapped computer

If tails was rebooted, add the Solana CLI tools to the PATH and
set up Solana CLI environment to devnet for the test run or mainnet-beta for the live run and set the cold-wallet.json as the key

bash
cd Persistent/solana-release/
export PATH=$PWD/bin:$PATH
cd ..

*Test Run
bash
solana config set -u devnet -k cold-wallet.json

*Live Run
solana config set -u mainnet-beta -k cold-wallet.json

Copy the new nonce.txt to the Persistent directory.

Create the offline transaction

*Test run or *Live Run
bash
solana deactivate-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json

Save each signing pair to their own text file.
signer1.txt
signer2.txt

Copy them to the Data USB


And back to the networked computer to send the transaction

Copy signer1.txt and signer2.txt to the working directory

*Test run or *Live Run
bash
solana deactivate-stake \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--stake-authority $(cat cold-wallet-address.txt) \
--fee-payer hot-wallet.json \
--signer $(cat signer1.txt) \
--signer $(cat signer2.txt) \
$(cat stake-account-address.txt)

View the stake account

bash
solana stake-account $(cat stake-account-address.txt)

It will take a couple of days for the stake to deactivate.

Withdraw Stake

The offline signing process can be used to withdraw the stake.

Once the deactivation is complete, the stake can be withdrawn.

On the networked computer

Check that the stake has been deactivated and make note of the balance

bash
solana stake-account $(cat stake-account-address.txt)

Create a text file with the numerical balance named balance.txt and save it to the Data USB.
(Do not include the SOL part, just the number)

Advance the nonce

bash
solana new-nonce nonce-account.json

Update the nonce.txt file and save it to the Data USB

bash
solana nonce nonce-account.json > nonce.txt

Switch to the air-gapped computer

If tails was rebooted, add the Solana CLI tools to the PATH and
set up Solana CLI environment to devnet for the test run or mainnet-beta for the live run and set the cold-wallet.json as the key

bash
cd Persistent/solana-release/
export PATH=$PWD/bin:$PATH
cd ..

*Test Run
bash
solana config set -u devnet -k cold-wallet.json

*Live Run
solana config set -u mainnet-beta -k cold-wallet.json

Copy the new nonce.txt and balance.txt to the Persistent directory.

Create the offline transaction





*Test Run or *Live Run
bash
solana withdraw-stake \
--sign-only \
--blockhash $(cat nonce.txt) \
--nonce $(cat nonce-account-address.txt) \
--nonce-authority hot-wallet.json \
--fee-payer hot-wallet.json \
stake-account.json \
cold-wallet.json $(cat balance.txt)

Save each signing pair to their own text file.
signer1.txt
signer2.txt

Copy them to the Data USB

Switch back to the networked computer

Copy signer1.txt and signer2.txt to the working directory.

Submit the withdraw transaction (You will get an error if deactivation has not completed.)

*Test Run or *Live Run
bash
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

Check the cold wallet account balance

bash
solana balance $(cat cold-wallet-address.txt)

Check the stake account

bash
solana stake-account $(cat stake-account-address.txt)

The error indicates it is no longer active.

Good job!

There are a few more stake operations that work with offline signing but they won’t be covered here.

With the knowledge learned, see if you can create and submit a transaction to send the balance from the cold wallet back to the hot wallet. 
The solution is included in a separate document. Or you can see below how to recover the cold wallet using a popular Solana software wallet. 


Recover the Cold Wallet using a Software Wallet (Hot Wallet)

Warning! When recovering the wallet in software, it ceases to be a cold wallet and becomes a hot wallet.
We can use a software wallet like Solflare.
https://www.solflare.com/

Open the wallet software and choose the “I already have a wallet” option.
Select the 24 word recovery phrase option.
Networking can be disabled while typing the seed phrase and wallet password.
*Note: the wallet password is not the same as the wallet passphrase. The wallet password is just used to protect the software wallet and is not tied to the seed phrase.
If this is the wallet created on devnet during the test run, the software will say “No Active Wallets Found”.
Select Advanced and Derivation Path. Ours should be m/44’/501’/0’/0’ and the address should match the one in cold-wallet-address.txt.
“You’re All Set!” Select Explore. 
Re-enable networking on the computer.
You still won’t see a balance if this is the devnet wallet.
Open setting and switch to Devnet.
Alright! We can now see all the details of our wallet.

I hope this guide was useful and enlightening! 
Please consider a small Solana donation to the author.











An3xM6xCLmBEn3NXjYoggvWQToL9Lmx5mH2USchUpyNz
