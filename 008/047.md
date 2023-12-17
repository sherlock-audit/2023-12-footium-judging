Jovial Coral Liger

medium

# Issue M-2: The Exact Amount For a Feature Is Not Checked In The MakePayment Function Which Could Lead To Users Buying New Features For Less Amount Than Expected.

## Summary

The exact amount needed to pay for a new feature in the ```makePayment```  is not validated on-chain in the function, this could allow any user to buy new features in the contract for less amount than expected.

## Vulnerability Detail

***Reporting this issue because the Readme file doesn't specify whether the amount asked for a new feature is predefined or if it's sent by the platform.***

The documentation mentions that the function ```makePayment``` is used to buy new features for a user's club in ETH.
This function can be called for any user by sending the ```_clubId```, the ```_message``` variable, and the payment in ETH.

When a user calls this function to buy a new feature for his club, the ```makePayment``` function checks if the sender is the owner of the club, and if the value sent is greater than 0, when these checks pass the function accepts the payment from the user, and sends it to the ```paymentReceiverAddress```, but the amount that the user paid for the new feature is never checked on the contract to validate on-chain that the amount paid is the exact amount requested for the purchased feature, this could allow any user to buy features for less than the expected amount.

let's imagine the next scenario:

1. A User wants to buy a new feature for his club, this new feature costs 1 ETH.
2. The user calls the ```makePayment``` and sends to the function his Club ID, the message, and 0.1 eth of value (or could be any amount less than 1 ETH) with bad intentions or by mistake.
3. The function ```makePayment``` checks the sender is the owner of the ```_clubId``` and the value sent is greater than 0, so the function passes and the amount is charged to the user.
4. The user now buys a feature for less than the expected amount.

## Impact

Any user using the Footium platform or going directly to the contract can buy features for his clubs for less than the amount requested for the feature, because this amount is not enforced on-chain.

## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumGeneralPaymentContract.sol#L84-L107

## Tool used

Manual Review

## Recommendation
Verified in the ```makePayment``` function the user is paying the correct amount requested to buy the feature, or revert the Tx. (on-chain enforce)
