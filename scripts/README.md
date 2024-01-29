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

From the signature, the 'r' and 's' values are taken. The 'k' value is derived using the public key which is also taken from the transaction details fetched using the BlockCypher API. The following code snippet shows the algorithm to calculate the private key. 

![script_snippet_4](https://github.com/Sreehari-BGK/DIY_Wallets_Vulnerabilities_SeedSigner/blob/main/scripts/snippet-images/script_snippet_4.png)

Also check out the full notebook -> scripts/private_key_calc_single_in_out_txn.ipynb for the full code for better understanding and direct use. 





