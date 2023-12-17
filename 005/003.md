Smooth Hotpink Trout

medium

# User will fail to claim Prize despite correct merkle proofs

## Summary

There a logical error in the way `totalERC20Claimed` & `totalETHClaimed` variables are handled which can lead to User getting DoSed from claiming their prize.

## Vulnerability Detail

The `FootiumPrizeDistributor` contract is designed in such a way that users can claim the prize using distinct Merkle trees proofs.

The issue affects both `claimERC20Prize` and `claimETHPrize` functions but to simplify explanation considering `claimERC20Prize` function here:

```solidity
File: FootiumPrizeDistributor.sol

    function claimERC20Prize(
        address _to,
        IERC20Upgradeable _token,
        uint256 _amount,
        bytes32[] calldata _proof
    ) external whenNotPaused nonReentrant {
        
        //-------SNIP: Merkle Proof Validation------------//

        // The amount to be claimed is the difference between the total amount
        // and the amount already claimed.
        // This means that the amount passed in the function call is the total
        // amount that was ever earned by the user.
@->     uint256 value = _amount - totalERC20Claimed[_token][_to];

        // If there is still an amount to be claimed, transfer the tokens
        if (value > 0) {
            totalERC20Claimed[_token][_to] += value;
            _token.safeTransfer(_to, value);
        }

        emit ClaimERC20(_token, _to, value);
    }

```
[Link to Code](https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L136)

This Bug exposes users to DoS in claiming their Prize tokens, particularly when attempting to claim rewards for the same token multiple times. The issue arises from the lack of differentiation between different roots. Check out PoC Steps:

**Proof of Concept:**

Consider the following Scenario:

1. Alice, the owner, initiates the distribution of `1000 USDT` to 5 recipients (including Bob), setting the initial `erc20MerkleRoot`.
2. Bob successfully claims his share using the `claimERC20Prize` function, updating the state: `totalERC20Claimed[USDT Addr][Bob's Addr] = 1000`.
3. After some time, Alice decides to distribute another `1000 USDT` to another 10 individuals (including Bob), so she submits new root.
4. Bob, equipped with correct proofs, attempts to claim again.
5. The call fails it's purpose at line 136 where `value` will turn out to be zero as `totalERC20Claimed[USDT Addr][Bob's Addr]` is already `1000`.

The problem comes from not keeping track of different `erc20MerkleRoot` in the `totalERC20Claimed` variable. Because of this, the contract wrongly thinks Bob has already received the tokens. This mix-up stops users from successfully claiming rewards more than once, causing DoS of Funds.

## Impact

Due to the improper handling of the `totalERC20Claimed`, users will lose at least the previously claimed amount of tokens when attempting to claim rewards for the same token multiple times.

## Code Snippet

Shown Above

## Tool used

Manual Review

## Recommendation

I would recommend to update the `totalERC20Claimed` & `totalETHClaimed` state variable as follow to differentiate between different roots.

```diff

-29:    mapping(IERC20Upgradeable => mapping(address => uint256))
-30:        private totalERC20Claimed;
-31:    mapping(address => uint256) private totalETHClaimed;

+29:    mapping(IERC20Upgradeable => mapping(address => mapping(bytes32 => uint256)))
+30:        private totalERC20Claimed;
+31:    mapping(address => mapping(bytes32 => uint256) private totalETHClaimed;


-136:    uint256 value = _amount - totalERC20Claimed[_token][_to];
+136:    uint256 value = _amount - totalERC20Claimed[_token][_to][erc20MerkleRoot];

-140:    totalERC20Claimed[_token][_to] += value;
+140:    totalERC20Claimed[_token][_to][erc20MerkleRoot] += value;

-181:    uint256 value = _amount - totalETHClaimed[_to];
+181:    uint256 value = _amount - totalETHClaimed[_to][ethMerkleRoot];

-185:    totalETHClaimed[_to] += value;
+185:    totalETHClaimed[_to][ethMerkleRoot] += value;

```