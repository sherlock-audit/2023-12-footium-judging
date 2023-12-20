Round Bubblegum Mammoth

medium

# Merkle leaf is 64 bytes

## Summary
Like the previous iteration of Footium, the merkle leaf used to verify player minting uses an id of 256 bits (32 bytes) and a string, which too is of 32 bytes size. 

## Vulnerability Detail
For a more extensive overview check out: https://github.com/sherlock-audit/2023-04-footium-judging/issues/300
Hashing together 64 bytes long arguments can lead to internal collisions, messing with the merkle proof

## Impact
Player Abuse

## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L113-L115

## Tool used

Manual Review

## Recommendation
Use a combination of variables that doesn't sum to 64 bytes




















