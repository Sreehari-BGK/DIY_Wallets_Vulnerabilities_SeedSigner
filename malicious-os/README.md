# Malicious SeedSigner OS

This malicious OS employs a custom 'embit' Python library, whose signing function has been modified to allow easy extraction of the private key after a transaction is signed using the function.


## Requirements

* Modify the Python embit library functions that the SeedSigner uses for signing a PSBT.
* Build the SeedSigner OS locally with this custom or modified embit library.

## Modified Functions
The modification revolves around deriving the deterministic 'k' value from the hash of the public key. This approach is fundamentally unique since the public key itself originates directly from the private key. By combining the hashed public key with the message digest for nonce derivation, our modified OS consistently produces signatures that are both deterministic and distinct.

The following code snippets show the changes we made in the sign_ecdsa function.

Original function to Sign Bitcoin Transactions:
![snippet_1](add_link)

Modified function to Sign Bitcoin Transactions: 
![snippet_2](add_link)

We also made the following changes to the sign function in the ec.py file to call our modified sign_ecdsa function specifically. 

Original sign function:
![snippet_3](add_link)

Modified sign function;
![snippet_3](add_link)


## Working & Outcome 
The variables required to calculate the ‘k’ value, namely the message/transaction digest and the public key, are readily accessible to anyone. All we need are the transaction details and the address, which are easy to get from any block explorer. 

In a scenario where a user conducts a transaction using SeedSigner with this malicious OS, an attacker can effortlessly trace the transaction by using the user's address. They can then calculate the nonce or 'k' value, obtain the transaction hash 'z', and directly access the signature from the witness section in native SegWit transactions. Both the 'r' and 's' values can be calculated or extracted.  This enables the attacker to derive and extract the private key used for signing from just a single transaction. The same is applicable and possible for all other types of transactions, too. 


