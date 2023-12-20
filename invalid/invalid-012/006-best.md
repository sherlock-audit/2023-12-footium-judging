Jolly Blonde Kookaburra

medium

# EIP2981 is not available in FootiumClub, user can bypass player royalties by selling clubs as a whole

## Summary

Same issue as previous Footium report. 

Players contract has royalty but clubs do not have. Club owner can bulk sell players via clubs to bypass the fee when selling players.

## Vulnerability Detail

FootiumPlayer.sol implements ERC2981

```solidity
contract FootiumPlayer is
    ERC721Upgradeable,
    AccessControlUpgradeable,
    ERC2981Upgradeable,
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    OwnableUpgradeable
{
```

Clubs do not implement ERC2981

```solidity
contract FootiumClub is
    ERC721Upgradeable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    OwnableUpgradeable
{
```

Players can sell their club to avoid fees on player sales.

## Impact

Players can bypass player royalties by selling club instead

## Code Snippet

https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L14-L20
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumPlayer.sol#L16-L23

## Tool used

Manual Review

## Recommendation

Add EIP2981 on Footium Club as well.