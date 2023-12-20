Alert Citron Pangolin

medium

# **FootiumGeneralPaymentContract.sol:makePayment()** can potentially lead to a loss of funds for the user

## Summary
From the docs: 
> The makePayment function allows club owners to make payments for future features using ETH. It takes two arguments, the club ID and a message describing the feature to pay for.

Upon further discussion with the sponsor, the way this works is that their off-chain indexer indexes all the **PaymentMade** events and unlocks features for players based on these emitted events. 

This is the event: 
```solidity
event PaymentMade(
        uint256 indexed clubId,
        uint256 indexed amount,
        string message
    );
```

The important parameter here is the string **message** which is used to identify the different features.
The problem with this function is that a user can use an arbitrary input for this parameter in the **makePayment()** call. Even the default zero value for example which can lead to a potential loss of funds for the user.

## Vulnerability Detail
Heavily relying on off-chain mechanisms for unlocking new features to users can lead to a loss of funds for users in multiple different ways:

Example:
Alice wants to pay for **Feature X** with a price of **1 ether**. The **_message** input she should use for this feature is "training feature" 
The current implementation opens up multiple possibilities for stuff to go wrong here.

1. Alice uses either the wrong **_message** input or none at all. (This could be because of mistyping for example)
-> off-chain indexer wouldn't index the emitted event correctly. Alice does not unlock the desired feature and loses funds.

2. Alice uses the correct **_message** input but sends too much eth along with the function call. 
-> off-chain indexer correctly indexes the emitted event but Alice would not get a refund on the excess eth. Alice loses funds.

3. Alice accidentally pays for the same feature multiple times.
-> because of the lack of any on-chain checks regarding this she can and loses funds.

## Impact
Potential loss of funds

## Code Snippet
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
        // Require that the sender is the owner of the club
        if (msg.sender != footiumClub.ownerOf(_clubId)) {
            revert NotClubOwner(_clubId, msg.sender);
        }

        // Require that the payment amount is greater than 0
        if (msg.value <= 0) {
            revert IncorrectETHAmount(msg.value);
        }

        // Send the payment to the payment receiver address
        (bool sent, ) = paymentReceiverAddress.call{value: msg.value}("");
        if (!sent) {
            revert FailedToSendETH(msg.value);
        }

        emit PaymentMade(_clubId, msg.value, _message);
    }
```

## Tool used

Manual Review

## Recommendation
Use state variables to track features, their price and the users that paid for these features and add additional checks in the **makePayment()** function.