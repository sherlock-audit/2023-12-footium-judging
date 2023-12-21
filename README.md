# Issue M-1: FootiumClub::ERC721Upgradeable/Pausable Cannot pause transfers on FootiumClub NFT 

Source: https://github.com/sherlock-audit/2023-12-footium-judging/issues/8 

## Found by 
cergyk
## Summary
Pausing FootiumClub only pauses the calls to `safeMint` and not transfers. This means that when paused, clubs can still be traded on secondary markets such as OpenSea.

The pause feature is an emergency feature meant to protect users and the protocol in case of major breach for example. In the case of a major emergency the pausing implemented currently is useless, since all unpermissioned actions such as transferring and selling NFTs are still enabled. 

## Vulnerability Detail

FootiumClub inherits ERC721Upgradeable class, it is not aware of the pausable abilities of the child contract:
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L16

When the admin pauses the contract, only methods explicitely using whenNotPaused modifier would be impacted, in this case only safeMint:
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L56-L69

This means that `transfer` can still be called, and tokens can still be traded on secondary.

## Impact

Pausing the NFT contract still enables transfers, and selling on secondary.

As seen in recent events such as here:
https://x.com/0xfoobar/status/1736060236354498641?s=20

A non operating pausing feature will prevent the team to react accordingly following a hack, and may cause loss of NFTs to users of the protocol


## Code Snippet

## Tool used
Manual Review

## Recommendation

FootiumClub should inherit ERC721PausableUpgradeable class of openzeppelin-contracts-upgradeable, implementing a check on paused in _beforeTokenTransfer hook:
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/token/ERC721/extensions/ERC721PausableUpgradeable.sol#L37-L46



## Discussion

**sherlock-admin2**

1 comment(s) were left on this issue during the judging contest.

**darkart** commented:
> Contract will be deployedonly on Arbitrum



**nevillehuang**

At first glance, this seems to be a future type issue. But I think it is a good catch by the watson, as sponsor confirmed with me that their current contracts allows for full pausability as seen [here](https://etherscan.io/address/0x659cf1306edba213d1fb8f9352b4593a82b05d0c#code) and that pausing transfers would be the behaviour they want, which means this attack could be possible during migration when a snapshot is taken. However, it is also speculating that the game will continue running on L1 when migrated to L2.

Funny enough, in the latest edits, the migration scenario was removed, and both the scenario wasn't highlighted on #8 and #13, so can be argued that both issues are invalid given the impact described of NFT sales is not a valid issue as it is speculating on a emergency situation. On the other hand, this could also be seen as a core functionality of pausing not working.

I am keeping both #13 and  #8 as a valid medium given the root cause is valid, and that the watson previously highlighted a potential scenario that is concerning and of value to sponsor before, and this would constitue a undesired future type issue.

This is the previously highlighted scenario as seen in this diff:

https://github.com/rcstanciu/diffs/blob/main/008.diff

> Alice owns FootiumClub NFT of tokenId: 1234

> Alice makes an offer for selling this token for 0.2 eth on Opensea.

> Unknowingly to Alice & Bob, Footium team makes the snapshot for L2 migration.
Footium team pauses the FootiumClub NFT contract, in order to freeze token ownership.

> Since transfers are not actually paused, Bob takes Alice's offer on Opensea.

> The snapshot had already been taken, so only Alice can claim the equivalent club on L2, and Bob is rugged

**Czar102**

After an internal discussion, taking into account that:
- the only function is guarded by the `whenNotPaused` modifier, and
- that is a function guarded by an `onlyOwner` modifier, which makes the `whenNotPaused` modifier useless as the owner has control over the paused status,
I think it is a nice catch to note that the protocol intention was to actually guard the transfers and make transfers pausable (using `ERC721PausableUpgradeable`) and it was a protocol error to use a vanilla `Pausable`.

I think it could have been clear and can be a valid issue solely based on the resources available to Watsons during the audit. Hence, I think it should be Medium.

