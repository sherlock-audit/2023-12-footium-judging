Tangy Scarlet Duck

high

# `FootiumGeneralPaymentContract` and `FootiumAcademy` cannot update the `footiumClub` address. This may cause functions to revert for new club owners.

## Summary
`FootiumGeneralPaymentContract` and `FootiumAcademy` do not have setter functions to update the `footiumClub` address. So, club owners minted by `FootiumClubMinter` using a new address for `footiumClub` will not be able to call functions like `makePayment` and `mintPlayer`.

## Vulnerability Detail

The contract `FootiumClubMinter` mints a new club to a receiver and 20 new players for the club. `FootiumClubMinter` also has a setter function that updates the `footiumClub` address:

```solidity
    /**
     * @dev Updates Footium Clubs contract address.
     * @param _footiumClub Footium Clubs contract address.
     * @dev Only owner address allowed.
     */
    function setClubAddress(IFootiumClub _footiumClub) external onlyOwner {
        footiumClub = _footiumClub;
    }
```
But, this setter function is absent from `FootiumGeneralPaymentContract` and `FootiumAcademy`. This could lead to critical problems for new club owners.

Let's say that the owner updates for whatever reason the `footiumClub` address in the `FootiumClubMinter`. The owner calls the `mint` function to mint a club and players for a receiver. Note that this new club was minted for the receiver using an updated address for the `footiumClub`. The `FootiumGeneralPaymentContract` is not aware of this updated `footiumClub` address and has no way to update it because of a lack of setter functions. For `FootiumGeneralPaymentContract`, the `footiumClub` is still the older address that was first initialised. When new users(club owners that were minted using the updated address in `FootiumGeneralPaymentContract`) use the function `makePayment`, the following condition will revert:

```solidity
        // Require that the sender is the owner of the club
        if (msg.sender != footiumClub.ownerOf(_clubId)) {
            revert NotClubOwner(_clubId, msg.sender);
        }
```

This is because the above conditional checks for the owner of the `_clubId` using an older address for `footiumClub`. The `_clubId` was minted using the updated address. Hence the condition fails and the new club owner is never able to make a payment.

Similarly, in the `FootiumAcademy` which also lacks these setter functions, the new club owner(minted using the updated address) is never able to call `mintPlayer` because the same condition reverts. `FootiumAcademy` too would be using the older address for the `footiumClub`. 

This also leads to another vulnerability in `FootiumAcademy`. A `clubId` could already exist for the older address of the `footiumClub`. So, an older club owner with the same `clubId` associated with the older`footiumClub` address, could simply call `mintPlayer` and mint themselves a player with the `playerId` that is supposed to be meant for a new club owner.

## Impact
Functions like `makePayment` and `mintPlayer` will revert.

## Code Snippet

[makePayment](https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumGeneralPaymentContract.sol#L84):

```solidity
    /**
     * @notice Makes a payment for the given feature.
     * @param _clubId Club ID.
     * @param _message A message describing a feature to pay for.
     * @dev This function is payable.
     * Emits a {PaymentMade} event.
     */
    function makePayment(uint256 _clubId, string calldata _message)
        external
        payable
        whenNotPaused
        nonReentrant
    {
```

[mintPlayer](https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L96): 

```solidity
    /**
     * @notice Mint a player.
     * @param clubId The ID of the club to receive the players.
     * @param playerId The ID of the player to be minted.
     * @param mintProof The proof that a certain player can be minted.
     * @dev Only the owner can mint players to their club.
     */
    function mintPlayer(
        uint256 clubId,
        string calldata playerId,
        bytes32[] calldata mintProof
    ) external payable whenNotPaused nonReentrant {
```

## Tool used

Manual Review

## Recommendation
Add the appropriate setter functions in `FootiumGeneralPaymentContract` and `FootiumAcademy`