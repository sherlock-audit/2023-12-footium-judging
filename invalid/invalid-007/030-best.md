Great Amber Falcon

medium

# Audit:  Unrestricted Minting in FootiumAcademy's mintPlayer Function. USers can mint by sending any value of ether. There is no minimum fee

## Summary
`FootiumAcademy::mintPlayer` has no minimum required amount of ether expected to be sent

## Vulnerability Detail
The mintPlayer function allows anyone to mint a player NFT without verifying whether the sender has provided the required minimum ETH amount to be sent. Lack of this check might lead to unintended minting and could be exploited by attackers to mint NFTs without adhering to the protocol's intended economic model.

## Impact
There is a potential security vulnerability in the `mintPlayer` function, allowing anyone to mint an `NFT` without a check for the minimum required `ETH` sent.

## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L96

```javascript

function mintPlayer(
    uint256 clubId,
    string calldata playerId,
    bytes32[] calldata mintProof
) external payable whenNotPaused nonReentrant {
    // Ensures that the player has not been minted before
    require(_mintedPlayers[playerId] == false, "Player already minted");

    // Ensures that the wallet performing the mint is the owner of the club
    // to which the player is being minted
    if (msg.sender != _footiumClub.ownerOf(clubId)) {
        revert NotClubOwner(clubId, msg.sender);
    }

    // @audit Missing check for minimum ETH sent as msg.value required to mint
    // Additional code should be added here to enforce the minimum ETH requirement
}

```

## Tool used

Manual Review

## Recommendation
It is strongly recommended to implement a check for the minimum required ETH needed to be sent  before allowing the minting of an NFT. This check should be designed to ensure that users adhere to the intended economic model and prevent potential abuse of the minting functionality. The specific minimum ETH amount to be sent and any related conditions should be defined in accordance with the protocol's requirements.