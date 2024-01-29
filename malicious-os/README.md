# Malicious SeedSigner OS

This malicious OS employs a custom 'embit' Python library, whose signing function has been modified to allow easy extraction of the private key after a transaction is signed using the function.


## Requirements

* Modify the Python embit library functions that the SeedSigner uses for signing a PSBT.
* Build the SeedSigner OS locally with this custom or modified embit library.

## Modified Functions
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
* Install or download the embit library, make the necessary modification to the necessary files and copy paste the whole embit library folder to the '\src' path. It may look something like this depending on where your SeedSigner OS files are:
  C:\dev\projects\seedsigner-os\opt\rootfs-overlay\opt\src.
  
* Remove any calls to the embit library from the OS files, that is remove the embit-specific files from the external-packages folder, and also remove embit references from the pre-build, and post-build files of the board that you're using. This includes the following files:
  * /opt/pi0-dev/board/post-build.sh
  * /opt/pi0-dev/Config.in
    
  And same inside the pi0 folder (that is if the board you're using is the pi0 make changes in both pi0-dev and pi0).
  
* Once these changes are made, then just build the OS locally. This OS should now be using the modified embit library. You can use the following command to build the OS:
  * SS_ARGS="--pi0 --no-clean --skip-repo" docker-compose up --force-recreate --build

## Working & Outcome 
The variables required to calculate the ‘k’ value, namely the message/transaction digest and the public key, are readily accessible to anyone. All we need are the transaction details and the address, which are easy to get from any block explorer. 

In a scenario where a user conducts a transaction using SeedSigner with this malicious OS, an attacker can effortlessly trace the transaction by using the user's address. They can then calculate the nonce or 'k' value, obtain the transaction hash 'z', and directly access the signature from the witness section in native SegWit transactions. Both the 'r' and 's' values can be calculated or extracted.  This enables the attacker to derive and extract the private key used for signing from just a single transaction. The same is applicable and possible for all other types of transactions, too. 


