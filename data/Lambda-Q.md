## GroupBuy: DoS when someone made a lot of bids
`GroupBuy.claim` iterates over all bids that the caller has made. When this is a very large list, this iteration can use a lot of gas and it may result in situations where someone cannot call `claim`, leading to a loss of funds. Consider allowing partial claiming by specifying a range. 

## GroupBuy: Insertion timestamp ignored
The documentation states that "If the users have the same quantity as well, the bid that was placed later will have Raes removed.". However, with the current implementation, this is not always true. Within the priority queue, the insertion timestamp is not part of the ordering function `isGreater` and the queue implementation does not guarantee that a later inserted item will be before an earllier inserted one and vice-versa. This is the case because do not care about the exact position within a level of the binary heap, only about the relationship between the levels.

If this behavior should be enforced, the timestamp would need to be part of the ordering relationship.

## GroupBuy.contribute does not set `pendingBalances` for unused capital, leading to locked up money
When there is some unfilled quantity within `GroupBuy.contribute`, the corresponding amount is not subtracted from `userContributions` / `totalContributions` and `pendingBalances` is not increased by the corresponding value. Because of that, the user has to wait until the purchase is finished such that he can call `claim` to get back this money. This is very capital-inefficient, as it is already clear at this point that the money that was sent for the unfilled quantity will never be used (and can never be used), so it would be preferable to allow the user to withdraw it immediately.

## GroupBuy.printQueue: Debug function in production code
The function `GroupBuy.printQueue` is only intended for debugging (and does not do anything without log statements). Consider removing it before deployment to decrease the bytecode size (and therefore the deployment costs).

## Incentive for NFT seller to call `purchase` with highest possible value
Because anyone can call `GroupBuy.purchase`, there is a large incentive for the seller of an NFT to call this function with the highest possible value (`minReservePrices[_poolId] * filledQuantities[_poolId]`), even if his NFT is listed for a much lower price. Consider restricting this function to only wallets that contributed to the buy (which have an incentive to buy the NFT cheaply).

## OptimisticListingSeaport.updateFeeReceiver: Two-step process for changing fee receiver recommended
For actions like `updateFeeReceiver` that are only callable by the current address (current fee receiver in this case), a two step process for changing the addresses is recommended. Like that, it is ensured that the fee receiver is not lost irreversibly when an invalid address is provided.

## OptimisticListingSeaport._constructOrder: Wrong comment
The following comment is written in `_constructOrder`:
```solidity
// 1 Consideration for the listing itself + 1 consideration for the fees
orderParams.totalOriginalConsiderationItems = 3;
```
However, instead of one (like the comment says), there are two considerations for the fees (Tessera + OpenSea), resulting in 3 overall consideration items.

## OptimisticListingSeaport.propose: `activeListings[_vault]` instead of storage variable `activeListing` used
In the function `propose`, `activeListings[_vault]` is used when comparing the price per token:
```solidity
if (
            _pricePerToken >= proposedListing.pricePerToken ||
            _pricePerToken >= activeListings[_vault].pricePerToken
        ) revert NotLower();
```
However, a few lines above, this value is already loaded as a storage variable (together with `proposedListing`:
```solidity
Listing storage proposedListing = proposedListings[_vault];
Listing storage activeListing = activeListings[_vault];
```
To save this mapping read, using `activeListing` is recommended there.

## OptimisticListingSeaport.propose: Mismatch with documentation
According to the documentation, "The proposed listing price must be at least 5% cheaper than the active proposal price.". However, it is only checked if the price is lower than min(active proposal price, proposed proposal price), not if it is at least 5% lower:
```solidity
if (
            _pricePerToken >= proposedListing.pricePerToken ||
            _pricePerToken >= activeListings[_vault].pricePerToken
        ) revert NotLower();
```

## OptimisticListingSeaport: Calling `rejectProposal` and `rejectActive` with zero amount possible
It is possible to call these functions with an `_amount` of 0, even if there are no proposals for a vault. While this does not introduce any direct vulnerabilities, the corresponding events will still be emitted, which can be confusing for monitoring solutions. Consider validating that the amount is greater than 0.

## OptimisticListingSeaport ignores royalties / creator fees
The listings that are created only include the Tessera and the OpenSea fee, but do not include any royalty payments, even if the NFT requires them. This could become quite problematic with OpenSea's royalty enforcement (see [here](https://opensea.io/blog/announcements/on-creator-fees/), [here](https://support.opensea.io/hc/en-us/articles/1500009575482-How-do-creator-earnings-work-on-OpenSea-), or [here](https://github.com/ProjectOpenSea/operator-filter-registry)), where transfers can be blocked when royalties are ignored. It is not very clear to me how OpenSea will handle listings on Seaport that ignore royalties. Obviously, they will not completely block their own smart contract, but maybe they will start enforcing in a Seaport upgrade that royalties are enforced, even if the listing is done from another protocol (and not from the Opensea frontend, where the royalties seem to be enforced at the moment).

Because of that, it may be desirable to respect royalties.