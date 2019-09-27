# CryptoPandas Birthing Process

## Permissioned version (non-custodial)
1. The user selects two pandas of opposite gender from their wallet and sends their transaction hashes to the backend, as well as a non-slp UTXO for fees and a name for the child panda.
2. The backend prepares an unsigned (non-SLP) transaction that contains the following outputs and sends the preimage to the frontend:
    1. An OP_RETURN with the following fields:
        1. lokad id: PND1
        2. the father transaction hash (note: not token id)
        3. the mother transaction hash
        4. public key for father and mother
        5. a signature with the public key for the message "PANDA S3X:{father tokenid}+{mother tokenid}"
    2. A P2PKH with the address of the operators of CryptoPandas of 0.001 BCH.
    3. Any change back to the user's wallet.
3. The frontend signs the preimage and sends the signature to the backend.
4. The backend verifies the signature, completes the PND1 transaction with the signature and broadcasts it on the network.
5. The backend waits until the PND1 transaction confirmed.
6. After the PND1 transaction has been mined, the genome of the new panda is determined. It’s the result of gene swapping and random mutation based on the blockhash of the block containing the fertilizing transaction and the transaction hash of the PND1 transaction.
7. The backend prepares a SLP NFT1 transaction with the following inputs and outputs:
    * Inputs:
        1. An SLP NFT1 parent token (0x81) owned exlusively by CryptoPandas.
        2. The fee output of the PND1 transaction (for easier external verification).
        3. Any additional input required for fees/combining inputs (likely none).
    * Outputs:
        1. The SLP NFT1 OP_RETURN output with the following fields:
            1. lokad id: SLP\0
            2. token type: 0x41
            3. transaction type: GENESIS
            4. token ticker: PANDA
            5. token name: {name of the child panda}
            6. token document url: https://pand.as.cash/panda/{panda_genome_hex} !TBD!
            7. token document hash: !TBD!
            8. decimals: 0
            9. mint_baton_vout: {empty string}
            10. initial_token_mint_quantity: 1
        2. P2PKH output with the address of the user
        3. Any change back to CryptoPanda's wallet.
8. The backend broadcasts the SLP NFT1 transaction and updates its UTXO set.

## Permissionless version (not required for the hackathon)

The technical process looks as follows (review SLP NFT1 specification for SLP details):
1. The user selects two pandas of opposite gender from their wallet
2. The app creates two DNA transactions for each panda, where the panda is both kept alive as new output (usually, procreation doesn’t result in death) and where a DNA baton output is created that will be consumed in a fertilizing transaction
3. The app then creates a fertilizing transaction, which consumes both DNA outputs, puts some metadata in OP_RETURN, and creates a new P2SH smart contract output with 0.001 BCH as birthing fee.
    1. The first challenge (or function) of the P2SH smart contract can only be spent by the creator of the fertilizing transaction and must send 0.001 BCH to the operators of CryptoPandas at vout=2. This will be chosen if the user wants to birth the panda after reviewing its stats
    2. The second challenge can only be spent by the operators of CryptoPandas pandas with a timelock of 1 day. This will be chosen if the user didn’t birth the panda after a day, and the fee can be claimed by the operators of CryptoPandas.
4. The app waits until the fertilizing transaction is mined and then notifies the user that the panda is ready to be born.
5. For the birthing transaction, an SLP NFT1 CryptoPandas parent token is required (see SLP NFT1 spec). These will be provided as anyone-can-spend smart contracts by the operators of CryptoPandas. They will maintain a pool of ~100 outputs that contain this parent token in the following P2SH output with two challenges:
    1. The output can be spent by anyone as long as 0.001 BCH are send to the operators of CryptoPandas at vout=2 and the input appears at the first position (vin=0).
    2. The output can be spent by the operators of CryptoPandas.
6. After the fertilizing transaction has been mined, the genome of the new panda is determined. It’s the result of gene swapping and random mutation based on the blockhash of the block containing the fertilizing transaction.
7. To get a panda token, a new SLP NFT1 GENESIS transaction will be created with the genome as part of the <token_document_url>.
8. For this, the P2SH output of the fertilizing transaction will be combined with a CryptoPandas parent token as above and both spent for the GENESIS transaction. The fee for the birthing is already part of the P2SH output and just has to be forwarded to the operators of CryptoPandas.
9. Immediately, the user can send the new panda token to a different address just like any other SLP NFT1 token.
