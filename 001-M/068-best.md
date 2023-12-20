Teeny Chiffon Parakeet

medium

# No royalty collected on secondary market trades of FootiumClub NFTs

## Summary

The FootiumClub NFT contract is currently not implementing the ERC2981 standard. This omission can result in the inability of Footium to collect royalties from the trading of club NFTs on secondary markets.

## Vulnerability Detail

The FootiumClub NFTs are designed for trading in secondary markets as outlined in the documentation.

        How do I get a club?
        At the moment clubs are only available on secondary markets.

https://footium.gitbook.io/footium-wiki/overview/footium-overview

However, there is no ERC2981 implemented in the contract. So the external marketplaces that enforce the ERC2981 may not recognize to charge the royalty for creator, the royalty fee will be zero and Footium will lose this stream of income.

        contract FootiumClub is
            ERC721Upgradeable,
            AccessControlUpgradeable,
            PausableUpgradeable,
            ReentrancyGuardUpgradeable,
            OwnableUpgradeable
        {

https://github.com/sherlock-audit/2023-12-footium/blob/b9cde4c9ec72bc3687e4d8cd3a4f451b02266537/footium-eth-shareable/contracts/FootiumClub.sol#L14-L20


## Impact

Footium loses the royalty fee in trading club NFT in secondary markets.

## Code Snippet

https://github.com/sherlock-audit/2023-12-footium/blob/b9cde4c9ec72bc3687e4d8cd3a4f451b02266537/footium-eth-shareable/contracts/FootiumClub.sol#L14-L20

## Tool used

Manual Review

## Recommendation

Implement ERC2981 for FootiumClub contract.
