Elegant Raspberry Sidewinder

medium

# Second preimage attack in FootiumPrizeDistributor::claimETHPrize

## Summary
A second preimage attack in Merkle trees can happen when an intermediate node in a merkle tree is presented as a leaf. 

See reference [here](https://www.rareskills.io/post/merkle-tree-second-preimage-attack#:~:text=The%20second%20preimage%20attack%20in,have%20multiple%20(computable)%20preimages.).

## Vulnerability Detail
OpenZeppelin states:
```solidity
/*
* WARNING: You should avoid using leaf values that are 64 bytes long prior to
* hashing, or use a hash function other than keccak256 for hashing leaves.
* This is because the concatenation of a sorted pair of internal nodes in
* the merkle tree could be reinterpreted as a leaf value.
*/
```
In this case `ethMerkleRoot` leaves are 64 bytes long that is the same length as the parents that are hashed. As a result an attacker can provide an intermediate node as a leaf with a shortened proof that will be accepted as valid.

**Proof of concept**

Consider the following tree:

h(c) <- c <- h(a) + h(b)

A leaf is calculated as:
```solidity
keccak256(abi.encode(_to, _amount))
```
`abi.encode(address,uint)` will output 64 bytes. Since encoding of the parent hashes `abi.encode(bytes32,bytes32)` will also be 64 bytes, it is possible to provide h(a) as _to and h(b) as _amount and [] as proof.

However there are two drawbacks to the attack that will make it challenging to pull off in practice. The first is that `_to` variable is a 20 bytes address and thus will require the node in the tree to have 12 leading zero bytes which may not occur. Second is the value is transferred to the user and so it is likely that the balance of the contract will not be sufficient for this transfer to succeed. 

For this reason this issue is rated as medium.

## Impact
This attack will disrupt the claim function and eth may be sent to invalid addresses.

## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/d14705eb59153988e5a7dc0643ad8c0495a9f2ad/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L147-L175

## Tool used
Manual Review

## Recommendation
Add address zero when hashing ETH merkle tree leaves values:
```solidity
keccak256(abi.encode(address(0), _to, _amount))
```