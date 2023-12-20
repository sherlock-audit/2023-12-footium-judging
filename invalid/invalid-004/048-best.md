Flat Olive Goblin

medium

# Malicious sellers can frontrun clubs transfers and mint all available `playerId` for that club.

## Summary
A club can only mint a particular `playerId` once. This allows malicious club sellers to frontrun the club's transfer transaction and mint all the possible `playerId` for that club. Consequently, the club buyer would be unable to mint any additional players from `FootiumAcademy`.

## Vulnerability Detail
The `FootiumAcademy.mintPlayer()` function checks whether the `playerId` has already been minted before (`require(_mintedPlayers[playerId] == false, "Player already minted")`), preventing a single club from minting the same `playerId` multiple times. However, this approach allows malicious club sellers to frontrun attempts to purchase a club and mint all `playerId` available for that club. As a result, the club buyer is left unable to mint any new players from the `FootiumAcademy`.

Consider the following series of events:
1. Alice list her club at some NFT marketplace.
2. Bob sends the transaction to buy Alice's club.
3. Alice sees Bob's transaction and tries to frontrun it by minting all possible `playerId` for her club.

If Alice successfully executes the frontrun, Bob will purchase Alice's club but won't be able to mint any players using it.

## Impact
Buyers targeted by malicious sellers will buy a club, but will be unable to mint new players from `FootiumAcademy`.

## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L96-L138

## Tool used
Manual Review

## Recommendation
Consider implementing a timelock mechanism to restrict the transfer of clubs after a certain period following the minting of a new player.
