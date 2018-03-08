chip012 -- Taproot and Graftroot via BLS Aggregates

## Abstract

Chia will support Taproot and Graftroot for enhanced script privacy.

## Motivation


## Specification


### Taproot

https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html

Taproot hides a script commitment underneath a public key in a witness script.

Alice and Bob wish to enter into an on-chain contract, enforced by some script, `S`.

Alice and Bob will create a new BLS threshold or msig pubkey `C`, and store their shares of the private key `c_a` and `c_b`. `C = g_2^(c_a * c_b)`.

Alice and Bob will modify `C` to create `P = C * g_2^H(C||S)`.

Alice and Bob pay to `P`.

Funds may be retrieved via mutual cooperation. Either Alice or Bob must offset their signature by `g_1^H(C||S)` after signing the message. Once both parties have signed, the resulting aggregate is multiplied by `g_1^H(C||S)` to make a valid signature for `P`.

# TODO: does this break aggregates? Weird factoring attacks? Ask advisors.

Alternatively, any party that can satisfy the script may publish the script and its arguments.

This means that unless the the script is revealed, its existence is hidden. Outside observers see a single BLS signature. It is indistinguishable from a single-signer account.

Taproot is not limited to two participants. `C` may be the public key for an msig or threshold group of any size.

### Graftroot

https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html

Graftroot is a system for delegating the ability to spend an output.

First a taproot pubkey `P` is created. Should the parties wish to delegate the UTXO, they create a new script `S_1` and sign the script such that the signature is valid under `P` (as described above). The holder of the delegate script may at any time publish it and its arguments, along with the signature under `P` to spend the UTXO.

Note: This delegation is not exclusive. The original participants may still spend by signing a transaction, or by revealing the Taproot script (if any).


1