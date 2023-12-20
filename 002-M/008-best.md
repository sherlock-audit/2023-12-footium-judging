Creamy Obsidian Gibbon

high

# FootiumClub::ERC721Upgradeable/Pausable Cannot pause transfers on FootiumClub NFT

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