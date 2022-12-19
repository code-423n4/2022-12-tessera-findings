# Gas Optimization Report

Disclaimer: report extended from 4naly3er tool

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 2 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 2 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables rather than re-reading them from storage | 5 |
| [GAS-4](#GAS-4) | Use calldata instead of memory for function arguments that do not get mutated | 5 |
| [GAS-5](#GAS-5) | Use Custom Errors | 2 |
| [GAS-6](#GAS-6) | Don't initialize variables with default value | 2 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 4 |
| [GAS-8](#GAS-8) | Use shift Right/Left instead of division/multiplication if possible | 5 |
| [GAS-9](#GAS-9) | Use != 0 instead of > 0 for unsigned integer comparison | 5 |
| [Gas-10](#GAS-10) | Use unchecked when it is safe | ? |
| [Gas-11](#GAS-11) | Refund contribution and pending balance in the same call | 1 |
| [Gas-12](#GAS-12) | Generate merkle tree offchain | 1 |
| [Gas-13](#GAS-13) | Pack Bid Struck | 1 |
| [Gas-14](#GAS-14) | Pack Queue Struck | 1 |

### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (2)*:
```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

106:             proposedListings[_vault].proposer == address(0) &&

107:             activeListings[_vault].proposer == address(0)

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (2)*:
```solidity
File: src/lib/MinPriorityQueue.sol

98:         for (uint256 i = 0; i < curUserBids.length; i++) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

390:             for (uint256 i = 0; i < _offer.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (5)*:
```solidity
File: src/modules/GroupBuy.sol

92:         contribute(currentId, _quantity, _raePrice);

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol)

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

37:         proxy = IWrappedPunk(wrapper).proxyInfo(address(this));

64:         IERC721(wrapper).safeTransferFrom(address(this), vault, tokenId);

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

362:             seaportLister,

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

```solidity
File: src/seaport/targets/SeaportLister.sol

38:                         IERC1155(token).setApprovalForAll(conduit, true);

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol)

### <a name="GAS-4"></a>[GAS-4] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (5)*:
```solidity
File: src/modules/GroupBuy.sol

166:         bytes memory _purchaseOrder,

167:         bytes32[] memory _purchaseProof

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol)

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

46:     function execute(bytes memory _order) external payable returns (address vault) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol)

```solidity
File: src/seaport/targets/SeaportLister.sol

26:     function validateListing(address _consideration, Order[] memory _orders) external {

51:     function cancelListing(address _consideration, OrderComponents[] memory _orders) external {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol)

### <a name="GAS-5"></a>[GAS-5] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (2)*:
```solidity
File: src/lib/MinPriorityQueue.sol

42:         require(!isEmpty(self), "nothing to return");

91:         require(!isEmpty(self), "nothing to delete");

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

### <a name="GAS-6"></a>[GAS-6] Don't initialize variables with default value

*Instances (2)*:
```solidity
File: src/lib/MinPriorityQueue.sol

98:         for (uint256 i = 0; i < curUserBids.length; i++) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

390:             for (uint256 i = 0; i < _offer.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="GAS-7"></a>[GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (4)*:
```solidity
File: src/lib/MinPriorityQueue.sol

60:                 j++;

77:         insert(self, Bid(self.nextBidId++, owner, price, quantity));

98:         for (uint256 i = 0; i < curUserBids.length; i++) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

```solidity
File: src/modules/GroupBuy.sol

74:         poolInfo[++currentId] = PoolInfo(

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol)

### <a name="GAS-8"></a>[GAS-8] Use shift Right/Left instead of division/multiplication if possible

*Instances (5)*:
```solidity
File: src/lib/MinPriorityQueue.sol

49:         while (k > 1 && isGreater(self, k / 2, k)) {

50:             exchange(self, k, k / 2);

51:             k = k / 2;

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

395:         uint256 openseaFees = _listingPrice / 40;

396:         uint256 tesseraFees = _listingPrice / 20;

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="GAS-9"></a>[GAS-9] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (5)*:
```solidity
File: src/modules/GroupBuy.sol

124:         if (fillAtAnyPriceQuantity > 0) {

139:         if (filledQuantity > 0) minReservePrices[_poolId] = getMinPrice(_poolId);

268:         if (pendingBalances[msg.sender] > 0) withdrawBalance();

297:         while (quantity > 0) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

469:         if (isValidated && !isCancelled && totalFilled > 0 && totalFilled == totalSize) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="GAS-10"></a>[GAS-10] Use unchecked when it is safe

For example, those dealing with eth value will basically never overflow:

```solidity
        userContributions[_poolId][msg.sender] += msg.value;
        totalContributions[_poolId] += msg.value;
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L115-L116)

```solidity
            filledQuantities[_poolId] += fillAtAnyPriceQuantity;
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L128)

Also those with explicit check before

```solidity
                lowestBid.quantity -= quantity;
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L309)

### <a name="GAS-11"></a>[GAS-11] Refund contribution and pending balance in the same call

```solidity
        // Transfers remaining contribution balance back to caller
        payable(msg.sender).call{value: contribution}("");

        // Withdraws pending balance of caller if available
        if (pendingBalances[msg.sender] > 0) withdrawBalance();
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L264-L268)

### <a name="GAS-12"></a>[GAS-12] Generate merkle tree offchain

Generate merkle tree onchain is expensive espically when you want to include a large set of value. Consider generating it offchain a publish the root when creating a new pool.

```solidity
        bytes32 merkleRoot = (length == 1) ? bytes32(_tokenIds[0]) : _generateRoot(_tokenIds);
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L71)

### <a name="GAS-13"></a>[GAS-13] Pack Bid Struct

The Bid struct can be packed tighter

```solidity
struct Bid {
    uint256 bidId;
    address owner;
    uint256 price;
    uint256 quantity;
}
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L4-L9)
to
```solidity
struct Bid {
    uint96 bidId;
    address owner;
    uint256 price;
    uint256 quantity;
}
```

### <a name="GAS-14"></a>[GAS-14] Pack Queue Struct

```solidity
    struct Queue {
        ///@notice incrementing bid id
        uint256 nextBidId;
        ///@notice array backing priority queue
        uint256[] bidIdList;
        ///@notice total number of bids in queue
        uint256 numBids;
        //@notice map bid ids to bids
        mapping(uint256 => Bid) bidIdToBidMap;
        ///@notice map addreses to bids they own
        mapping(address => uint256[]) ownerToBidIds;
    }
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L13-L24)
to
```solidity
    struct Queue {
        ///@notice incrementing bid id
        uint96 nextBidId;
        ///@notice total number of bids in queue
        uint160 numBids;
        ///@notice array backing priority queue
        uint256[] bidIdList;
        //@notice map bid ids to bids
        mapping(uint256 => Bid) bidIdToBidMap;
        ///@notice map addreses to bids they own
        mapping(address => uint256[]) ownerToBidIds;
    }
```
