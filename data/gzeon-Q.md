# QA Report

Disclaimer: report extended from 4naly3er tool

## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Unsafe ERC20 operation(s) | 1 |
| [L-2](#L-2) | Contribution to GroupBuy should check against minReservePrices | 1 |
| [L-3](#L-3) | Onchain merkle tree generation limited the size of tokenId | 1 |

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 9 |
| [NC-2](#NC-2) | Return values of `approve()` not checked | 1 |
| [NC-3](#NC-3) | Functions not used internally could be marked external | 11 |
| [NC-4](#NC-4) | minBidPrices is rounded down | 1 |
| [NC-5](#NC-5) | Member of _tokenIds might not be unique | 1 |
| [NC-6](#NC-6) | Do not use assert() in production | 1 |
| [NC-7](#NC-7) | Hardcoded fees | 1 |
| [NC-8](#NC-8) | Wrong comment | 1 |

### <a name="L-1"></a>[L-1] Unsafe ERC20 operation(s)

*Instances (1)*:
```solidity
File: src/seaport/targets/SeaportLister.sol

40:                         IERC20(token).approve(conduit, type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol)

### <a name="L-2"></a>[L-2] Contribution to GroupBuy should check against minReservePrices

If the bid is below minReservePrices it will be filtered

*Instances (1)*:
```solidity
        if (msg.value < _quantity * minBidPrices[_poolId] || _quantity == 0)
```

### <a name="L-3"></a>[L-3] Onchain merkle tree generation limited the size of tokenId

Generate merkle tree onchain is expensive espically when you want to include a large set of value. For example, it is not possible to include all Punk in the tokenId becuase the merkle root is generated onchain. Consider generate it offchain and publish the root.

```solidity
        bytes32 merkleRoot = (length == 1) ? bytes32(_tokenIds[0]) : _generateRoot(_tokenIds);
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L71)

### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (9)*:
```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

32:         registry = _registry;

33:         wrapper = _wrapper;

34:         listing = _listing;

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

71:         registry = _registry;

72:         seaport = _seaport;

73:         zone = _zone;

75:         supply = _supply;

76:         seaportLister = _seaportLister;

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

```solidity
File: src/seaport/targets/SeaportLister.sol

20:         conduit = _conduit;

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol)

### <a name="NC-2"></a>[NC-2] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (1)*:
```solidity
File: src/seaport/targets/SeaportLister.sol

40:                         IERC20(token).approve(conduit, type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol)

### <a name="NC-3"></a>[NC-3] Functions not used internally could be marked external

*Instances (11)*:
```solidity
File: src/lib/MinPriorityQueue.sol

27:     function initialize(Queue storage self) public {

36:     function getNumBids(Queue storage self) public view returns (uint256) {

41:     function getMin(Queue storage self) public view returns (Bid storage) {

90:     function delMin(Queue storage self) public returns (Bid memory) {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol)

```solidity
File: src/modules/GroupBuy.sol

346:     function getBidInQueue(uint256 _poolId, uint256 _bidId)

371:     function getNextBidId(uint256 _poolId) public view returns (uint256) {

377:     function getNumBids(uint256 _poolId) public view returns (uint256) {

384:     function getBidQuantity(uint256 _poolId, uint256 _bidId) public view returns (uint256) {

402:     function printQueue(uint256 _poolId) public view {

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol)

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

218:     function list(address _vault, bytes32[] calldata _listProof) public {

344:     function getPermissions()

```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol)

### <a name="NC-4"></a>[NC-4] minBidPrices is rounded down

minBidPrices is rounded down and can be 0, which will cause precision issue

```solidity
        minBidPrices[currentId] = _initialPrice / _totalSupply;
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L83)

### <a name="NC-5"></a>[NC-5] Member of _tokenIds might not be unique

```solidity
        uint256[] calldata _tokenIds,
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L59)

### <a name="NC-6"></a>[NC-6] Do not use assert() in production

It will consume all gas and does not provide a revert string, use `require()` instead

```solidity
        assert(ConsiderationInterface(_consideration).validate(_orders));
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L45)

```solidity
        assert(ConsiderationInterface(_consideration).cancel(_orders));
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L52)

### <a name="NC-7"></a>[NC-7] Hardcoded fees

Would require migration if either decided to change fee structure in the future.

```solidity
        uint256 openseaFees = _listingPrice / 40;
        uint256 tesseraFees = _listingPrice / 20;
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L395-L396)

### <a name="NC-8"></a>[NC-8] Wrong comments

Here should be `+ 2 consideration for the fees`

```solidity
            // 1 Consideration for the listing itself + 1 consideration for the fees
```
[Link to code](https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L384)
