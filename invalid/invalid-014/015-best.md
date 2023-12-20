Deep Canvas Otter

medium

# Missing payable when claiming ETH

## Summary
In `FootiumPrizeDistributor.sol` there is a function `claimETHPrize` 
```solidity
 function claimETHPrize(
        address _to,
        uint256 _amount,
        bytes32[] calldata _proof
    ) external whenNotPaused nonReentrant {
        // Ensure that the prize receiver is the sender
        if (_to != msg.sender) {
            revert InvalidAccount();
        }

        // Checks the Merkle proof, which is constructed as a hash of
        // the token address, the receiver address and the amount
        if (
            !MerkleProofUpgradeable.verify(
                _proof,
                ethMerkleRoot,
                keccak256(abi.encode(_to, _amount))
            )
        ) {
            revert InvalidETHMerkleProof();
        }

        // The amount to be claimed is the difference between the total amount
        // and the amount already claimed.
        // This means that the amount passed in the function call is the total
        // amount that was ever earned by the user.
        uint256 value = _amount - totalETHClaimed[_to];

        // If there is still an amount to be claimed, transfer the tokens
        if (value > 0) {
            totalETHClaimed[_to] += value;

            (bool sent, ) = _to.call{value: value}("");  //@audit - missing  payable
            if (!sent) {
                revert FailedToSendETH(value);
            }
        }

        emit ClaimETH(_to, value);
    }
}
```
 The claimETHPrize function is responsible for transferring ETH tokens from the contract to the address specified by the _to parameter. However, the function does not explicitly mark the _to address as payable before attempting to send ETH to it. This means that the function assumes that the _to address has a receive() or payable fallback function, which is not always the case.

## Vulnerability Detail
There is a missing payable before `_to` 
Explicit conversion from address to address payable is only possible if the contract has a receive or payable fallback function. 
## Impact
Not all addresses are payable, and if you try to send ethers to a non-payable address, the transaction will fail.
## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L155-L195
## Tool used

Manual Review

## Recommendation

Change:

```solidity
(bool sent, ) = _to.call{value: value}("");
```

To:

```solidity
(bool sent, ) = payable(_to).call{value: value}("");
```
