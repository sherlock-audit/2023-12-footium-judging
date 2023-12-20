Cold Macaroon Rook

medium

# Address restrictions in FootiumPrizeDistributor could lead to permanently inaccessible rewards

## Summary

The `claimETHPrize` and `claimERC20Prize` functions in `FootiumPrizeDistributor.sol` enforce that the `msg.sender` must be the `_to` parameter passed to the function and must be contained in the Merkle tree to validate the eligibility for the prize. This design could lead to issues where certain addresses, such as smart contracts that cannot receive ETH or ERC20 tokens, are unable to claim their prizes.

## Vulnerability Detail

In `FootiumPrizeDistributor.sol`, the `_to` parameter is used to validate the eligibility for claiming the prize and is also the address to which the prize is sent. This design assumes that the `_to` address is capable of receiving the prize, which may not always be the case. For instance, if `_to` is a smart contract that cannot receive ETH, the ETH transfer would fail, making it impossible for the prize to be claimed. Similarly, if the `_to` address is a smart contract that lacks the functionality to transfer ERC20 tokens, then the tokens sent to that address would effectively be lost because they could not be moved from there (though in this scenario, it is also likely the smart contract would not be capable of claiming the tokens in the first place). Similarly, if the address is blocked by the token e.g. due to a blacklist mechanism in the token contract, any attempt to transfer tokens to that address would fail.

## Impact

This issue could prevent certain addresses from claiming their prizes, leading to funds being stuck in the contract.

Consider the following scenario:
1. Footium launches a reward mechanism where each club owner gets some amount of ETH.
2. The prize distribution is based on a snapshot of all clubs, where each owner is assigned the same amount.
3. The contract owner creates the new Merkle root and updates it.
4. A club owner holds their NFTs in a contract that cannot receive ETH, so it is not possible for them to claim their rewards.

## Code Snippet

https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L116-L141
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L161-L187

## Tool used

Manual review

## Recommendation

Consider updating the `claimETHPrize` and `claimERC20Prize` functions to separate the address used for tracking the prize claims (`msg.sender`) from the address to which the prize is sent (`_to`). This would allow the caller to send the claimed prize to an address of their choice.

For example, the `claimETHPrize` function could be updated as follows:

```solidity
function claimETHPrize(
    address _to,
    uint256 _amount,
    bytes32[] calldata _proof
) external whenNotPaused nonReentrant {
    if (
        !MerkleProofUpgradeable.verify(
            _proof,
            ethMerkleRoot,
            keccak256(abi.encode(msg.sender, _amount))
        )
    ) {
        revert InvalidETHMerkleProof();
    }

    uint256 value = _amount - totalETHClaimed[msg.sender];

    if (value > 0) {
        totalETHClaimed[msg.sender] += value;

        (bool sent, ) = _to.call{value: value}("");
        if (!sent) {
            revert FailedToSendETH(_to, value);
        }
    }

    emit ClaimETH(msg.sender, _to, value);
}
```