chip019 -- Flyclient Support

## Abstract

Flyclient is a protocol that allows logarithmic time/space verification of the weight of a chain. Blocks commit to a merkle tree of all other block hashes, and provide merkle proofs of a random logarithmic subset of them, for use with light clients.
See slides [1]

## Motivation
In most current cryptocurrencies, light clients must download all block headers, which grows linearly with time.
(Bitcoin, 80 byte header, 50MB total. Ethereum: 510 byte header, 2.7GB total).
Chia will have even larger block headers, and thus it will be impractical for light clients to download them all.

Other approaches include creating a zero knowledge proof of the proof work on the chain, which is not practical for proving time yet, or NIPoPoWs, which are suceptible to bribing attacks.

## Explanation
Flyclient allows for configurable security, by sampling a random number of blocks, on logarithmic ranges:

First sample from the whole chain, then sample from the last 1/2, then from the last 1/4, then from the last 1/8, etc, until you hit some constant like 50 blocks.

Each block in the samples must be included in the correct position of the merkle tree in the final block.

Each sample convinces you that an attacker with 1/2 of the total space could not produce a correct response for the entire sample, therefore the attacker must have forked in a more recent block.

For example, if you sample 128 blocks from the last 1/8 blocks, an attacker with 1/2 of the total space in that time, can succeed in the proof with probability 1/2^128. Therefore, the fork must have happened in a block with height > (7/8) * height.

The sample can be deterministic, based on VDF of previous block, which allows for a non-interactive protocol: a proof is created once for each block, and can be passed around.

## Specification

Every block header must include the blockTreeRoot field, which is a merkle root committing to all previous block hashes.
The merkle tree is an append only merkle mountain range, with the leaves in the order of block height.

A flyclient proof at height h contains:
* The last l block headers, including their merkle proofs
* `floor(log2(h)) - 4` subproofs, each subproof containing k blocks headers, and a merkle proof of each block header. If there are less than k blocks to sample from, include all blocks.

The size of the proof will be around `((l + (log_2(h) - 4) * k) * (S + 32*log_2(h)))`.


For a header size S=10KB, constant l=64, k=50, and block height h=1 million, that is around 9MB total.

The blocks are sampled based on the VDF output of the previous block.

When syncing, a light client can just ask for the lastest block, along with a flyclient proof. The light client check that all the blocks were sampled correctly, that their merkle proofs are valid and include each sampled block in the right position, and that each sampled block is valid.

A light client can fall behind, if it goes offline for a while. When it wakes up, it can ask for all of the headers it's missing, or for a flyclient proof if it's very far behind.

For an SPV transaction proof, if the transaction happened before the light client went offline, the block merkle proof can be provided for the block that the light client has access to, so it's only necessary to catch up to view new transactions.

## Questions / Notes
1. Is it right to sample from VDF of previous block?
2. Can we keep k=50? Do we need 128 bit security here? Benedikt says no
3. How does this work with difficulty resets? Is there any modification necessary for it to work?
4. Do full nodes need to keep track of merkle trees for each block? How do we optimize the storage?
5. Do rewards chains need block merkle trees?

## References
[1] https://scalingbitcoin.org/stanford2017/Day1/flyclientscalingbitcoin.pptx.pdf


[2] https://scalingbitcoin.org/stanford2017/Day1/flyclientscalingbitcoin.pptx.pdf