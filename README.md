# Breaking DIY Bitcoin Signing Device

SeedSigner is a completely offline, stateless & air-gapped DIY Bitcoin Signing Device. It can be built using inexpensive, publicly available hardware components (usually < $50). SeedSigner helps users save with Bitcoin by assisting with trustless private key generation and multi-signature wallet setup and helps users transact with Bitcoin via a secure, air-gapped QR-exchange signing model.

How the SeedSigner works: https://www.hedgewithcrypto.com/seedsigner-review/


Link to original SeedSigner OS repo: https://github.com/SeedSigner/seedsigner-os


Link to original embit Python library: https://github.com/diybitcoinhardware/embit

This repo contains:

* Modified embit Python library.
* Modified OS build.
* Python notebook of the script to calculate the private key from a single input single output transaction.

# Malicious SeedSigner OS

This malicious OS employs a custom 'embit' Python library, whose signing function has been modified to allow easy extraction of the private key after a transaction is signed using the function.

## Requirements

* Modify the Python embit library functions that the SeedSigner uses for signing a PSBT.
* Build the SeedSigner OS locally with this custom or modified embit library.

## Modified Functions

For building the malicious software, we wanted to explore if there were ways in which someone could make changes in the original OS code without arousing user suspicion or detectable operational differences. Our findings and methodology are outlined below.

For some background, during a Bitcoin transaction, a crucial element, the random nonce 'k,' is used in the signing function (ECDSA). This nonce is deterministically derived, following the RFC6979 algorithm, from the transaction digest and the private key involved in the signing process.

The modification revolves around deriving the deterministic 'k' value from the hash of the public key. This approach is fundamentally unique since the public key itself originates directly from the private key. By combining the hashed public key with the message digest for nonce derivation, our modified OS consistently produces signatures that are both deterministic and distinct.

The following code snippets show the changes we made in the sign_ecdsa function.

Original function to sign Bitcoin transactions:
![snippet_1](https://github.com/Sreehari-BGK/SeedSigner_Scripts/blob/main/malicious-os/snippet-images/snippet_1.png)

Modified function to sign Bitcoin transactions: 
![snippet_2](https://github.com/Sreehari-BGK/SeedSigner_Scripts/blob/main/malicious-os/snippet-images/snippet_2.png)

Here the get_pubkey() will give the uncompressed public key derived from the private key, which will be then hashed and sent as input to the 'k' derivation function.  

We also made the following changes to the sign function in the ec.py file to call our modified sign_ecdsa function specifically. Both of the files in which these functions exist are also added in the "changed-embit-files" folder.

Original sign function:

![snippet_3](https://github.com/Sreehari-BGK/SeedSigner_Scripts/blob/main/malicious-os/snippet-images/snippet_3.png)


Modified sign function:
![snippet_4](https://github.com/Sreehari-BGK/SeedSigner_Scripts/blob/main/malicious-os/snippet-images/snippet_4.png)

## Building the OS
* First build the OS normally by following the SeedSigner build guide: https://github.com/SeedSigner/seedsigner-os/blob/main/docs/building.md
* Install or download the embit library, make the necessary modification to the necessary files, and copy and paste the whole embit library folder to the '\src' path. It may look something like this depending on where your SeedSigner OS files are:

  C:\dev\projects\seedsigner-os\opt\rootfs-overlay\opt\src.
  
* Remove any calls to the embit library from the OS files, that is remove the embit-specific files from the external-packages folder, and also remove embit references from the pre-build, and post-build files of the board that you're using. This includes the following files:
  * seedsigner-os\opt\pi0-dev\board\post-build.sh
  * seedsigner-os\opt\pi0-dev\Config.in
    
  And the same inside the pi0 folder (that is if the board you're using is the pi0 make changes in both pi0-dev and pi0).
  
* Once these changes are made, then just build the OS locally. This OS should now be using the modified embit library. You can use the following command to build the OS:
  * SS_ARGS="--pi0 --no-clean --skip-repo" docker-compose up --force-recreate --build

## Working & Outcome 
The variables required to calculate the ‘k’ value, namely the message/transaction digest and the public key, are readily accessible to anyone. All we need are the transaction details and the address, which are easy to get from any block explorer. 

In a scenario where a user conducts a transaction using SeedSigner with this malicious OS, an attacker can effortlessly trace the transaction by using the user's address. They can then calculate the nonce or 'k' value, obtain the transaction hash 'z', and directly access the signature from the witness section in native SegWit transactions. Both the 'r' and 's' values can be calculated or extracted.  This enables the attacker to derive and extract the private key used for signing from just a single transaction. The same is applicable and possible for all other types of transactions, too. 

# Private Key Calculation From Signature

## References 
* Native P2WPKH transactions: https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#native-p2wpkh
* Constructing SegWit transactions: https://medium.com/coinmonks/creating-and-signing-a-segwit-transaction-from-scratch-ec98577b526a

## Fetching the transaction using BlockCypher API 

The full transaction details can be fetched using BlockCypher API using the transaction hash. 
![script_snippet_1](https://github.com/Sreehari-BGK/DIY_Wallets_Vulnerabilities_SeedSigner/blob/main/scripts/snippet-images/script_snippet_1.png)

The following code snippet shows how the message hash and signature are taken.
![script_snippet_3](https://github.com/Sreehari-BGK/DIY_Wallets_Vulnerabilities_SeedSigner/blob/main/scripts/snippet-images/script_snippet_3.png)


![script_snippet_2](https://github.com/Sreehari-BGK/DIY_Wallets_Vulnerabilities_SeedSigner/blob/main/scripts/snippet-images/script_snippet_2.png)

## Calculating the private key

The 'r' and 's' values are taken from the signature. The 'k' value is derived using the public key which is also taken from the transaction details fetched using the BlockCypher API. The following code snippet shows the algorithm to calculate the private key. 

![script_snippet_4](https://github.com/Sreehari-BGK/DIY_Wallets_Vulnerabilities_SeedSigner/blob/main/scripts/snippet-images/script_snippet_4.png)

Also, check out the full notebook -> scripts/private_key_calc_single_in_out_txn.ipynb for the full code for better understanding and direct use. 







