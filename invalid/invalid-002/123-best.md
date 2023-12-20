Spare Flint Halibut

medium

# Academy player NFT minting beyond expected functionality.

## Summary

According to the documentation, the expected number of players entering the academy each season is 10, and NFTs are supposed to be minted for them. However, in the `FootiumAcademy::mintPlayer` function, there is no limit specified based on the documentation. Instead, it allows minting NFTs for all non-initial players using their `playerId`. This has the potential to exceed the expected functionality of the academy by allowing more than 10 players. This is not in line with the expected functionality!

## Vulnerability Detail

1. The club owner has 20 initial players (with `playerId`s of `85-0-0`, `85-0-1`, `85-0-2`, etc.) and 20 non-initial players (including 15 academy players with `playerId`s of `85-1-0`, `85-1-1`, `85-1-2`, etc.).
2. According to the documentation, the expected number of academy players is 10.
3. However, due to the lack of conditional restrictions in `FootiumAcademy::mintPlayer`, the minting of academy player NFTs exceeds the expected functionality. This will result in more than 15 academy player NFTs being minted.

## Impact

Minting of NFTs exceeding the expected number of academy players.

## Code Snippet

[FootiumAcademy.sol#L96-L138](https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L96-L138)

## Tool used

Manual Review

## Recommendation

I suggest adding a conditional check to limit the maximum number of Academy Player NFTs minted to 10 per season.
