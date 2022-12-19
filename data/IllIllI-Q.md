
## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Bid size is an unfair ordering metric | 1 | 
| [L&#x2011;02] | Users may DOS themselves with a lot of smalle payments | 1 | 
| [L&#x2011;03] | Empty `receive()`/`payable fallback()` function does not authorize requests | 2 | 
| [L&#x2011;04] | `require()` should be used instead of `assert()` | 2 | 
| [L&#x2011;05] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 12 | 

Total: 18 instances over 5 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Debugging functions should be moved to a child class rather than being deployed | 1 | 
| [N&#x2011;02] | Typos | 4 | 
| [N&#x2011;03] | `public` functions not called by the contract should be declared `external` instead | 10 | 
| [N&#x2011;04] | `constant`s should be defined rather than using magic numbers | 4 | 
| [N&#x2011;05] | Missing event and or timelock for critical parameter change | 1 | 
| [N&#x2011;06] | NatSpec is incomplete | 7 | 
| [N&#x2011;07] | Consider using `delete` rather than assigning zero to clear values | 4 | 
| [N&#x2011;08] | Contracts should have full test coverage | 1 | 
| [N&#x2011;09] | Large or complicated code bases should implement fuzzing tests | 1 | 

Total: 33 instances over 9 issues





## Low Risk Issues

### [L&#x2011;01]  Bid size is an unfair ordering metric
The README states that this is intentional, so I've filed it as Low rather than Medium, but giving priority to bids with the smaller quantity is not a fair ordering mechanic. A person with a lot of funds may have gotten that by pooling externally to the contract, and it's not fair to kick them out of the pool earlier than another address that came in later

*There is 1 instance of this issue:*

```solidity
File: /src/lib/MinPriorityQueue.sol

111      function isGreater(
112          Queue storage self,
113          uint256 i,
114          uint256 j
115      ) private view returns (bool) {
116          Bid memory bidI = self.bidIdToBidMap[self.bidIdList[i]];
117          Bid memory bidJ = self.bidIdToBidMap[self.bidIdList[j]];
118          if (bidI.price == bidJ.price) {
119              return bidI.quantity <= bidJ.quantity;
120          }
121          return bidI.price > bidJ.price;
122:     }

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L111-L122

### [L&#x2011;02]  Users may DOS themselves with a lot of smalle payments
If a user has to contribute to a pool via lots of dust payments (e.g. if they only have enough money each week to spend a few wei), they may eventually add enough payments that when it's time to claim their excess, their for-loop below exceeds the block gas limit

*There is 1 instance of this issue:*

```solidity
File: /src/modules/GroupBuy.sol

247          if (success) {
248              for (uint256 i; i < length; ++i) {
249                  // Gets bid quantity from storage
250:                 Bid storage bid = bidPriorityQueues[_poolId].bidIdToBidMap[bidIds[i]];

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L247-L250

### [L&#x2011;03]  Empty `receive()`/`payable fallback()` function does not authorize requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. `require(msg.sender == address(weth))`). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

*There are 2 instances of this issue:*

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

41:       receive() external payable {}

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L41

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

83:       receive() external payable {}

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L83

### [L&#x2011;04]  `require()` should be used instead of `assert()`
Prior to solidity version 0.8.0, hitting an assert consumes the **remainder of the transaction's available gas** rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that "The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix".

*There are 2 instances of this issue:*

```solidity
File: src/seaport/targets/SeaportLister.sol

45:           assert(ConsiderationInterface(_consideration).validate(_orders));

52:           assert(ConsiderationInterface(_consideration).cancel(_orders));

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L45

### [L&#x2011;05]  Missing checks for `address(0x0)` when assigning values to `address` state variables

*There are 12 instances of this issue:*

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

32:           registry = _registry;

33:           wrapper = _wrapper;

34:           listing = _listing;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L32

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

71:           registry = _registry;

72:           seaport = _seaport;

73:           zone = _zone;

75:           supply = _supply;

76:           seaportLister = _seaportLister;

77:           feeReceiver = _feeReceiver;

78:           OPENSEA_RECIPIENT = _openseaRecipient;

338:          feeReceiver = _new;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L71

```solidity
File: src/seaport/targets/SeaportLister.sol

20:           conduit = _conduit;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L20

## Non-critical Issues

### [N&#x2011;01]  Debugging functions should be moved to a child class rather than being deployed

*There is 1 instance of this issue:*

```solidity
File: /src/modules/GroupBuy.sol

402      function printQueue(uint256 _poolId) public view {
403          uint256 counter;
404          uint256 index = 1;
405          MinPriorityQueue.Queue storage queue = bidPriorityQueues[_poolId];
406          uint256 numBids = queue.numBids;
407          while (counter < numBids) {
408              Bid memory bid = queue.bidIdToBidMap[index];
409              if (bid.bidId == 0) {
410                  ++index;
411                  continue;
412              }
413              ++index;
414              ++counter;
415          }
416:     }

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L402-L416

### [N&#x2011;02]  Typos

*There are 4 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

/// @audit addreses
22:           ///@notice map addreses to bids they own

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L22

```solidity
File: src/modules/GroupBuy.sol

/// @audit equalt
179:          // Reverts if NFT contract is not equalt to NFT contract set on pool creation

/// @audit Verifes
208:              // Verifes vault is owner of ERC-721 token

/// @audit specifc
286:      /// @notice Attempts to accept bid for specifc quantity and price

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L179

### [N&#x2011;03]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 10 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

27:       function initialize(Queue storage self) public {

36:       function getNumBids(Queue storage self) public view returns (uint256) {

41:       function getMin(Queue storage self) public view returns (Bid storage) {

90:       function delMin(Queue storage self) public returns (Bid memory) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L27

```solidity
File: src/modules/GroupBuy.sol

346       function getBidInQueue(uint256 _poolId, uint256 _bidId)
347           public
348           view
349           returns (
350               uint256 bidId,
351               address owner,
352               uint256 price,
353:              uint256 quantity

371:      function getNextBidId(uint256 _poolId) public view returns (uint256) {

377:      function getNumBids(uint256 _poolId) public view returns (uint256) {

384:      function getBidQuantity(uint256 _poolId, uint256 _bidId) public view returns (uint256) {

402:      function printQueue(uint256 _poolId) public view {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L346-L353

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

218:      function list(address _vault, bytes32[] calldata _listProof) public {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L218

### [N&#x2011;04]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 4 instances of this issue:*

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

/// @audit 3
350:          permissions = new Permission[](3);

/// @audit 3
385:              orderParams.totalOriginalConsiderationItems = 3;

/// @audit 40
395:          uint256 openseaFees = _listingPrice / 40;

/// @audit 20
396:          uint256 tesseraFees = _listingPrice / 20;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L350

### [N&#x2011;05]  Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

*There is 1 instance of this issue:*

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

336       function updateFeeReceiver(address payable _new) external {
337           if (msg.sender != feeReceiver) revert NotAuthorized();
338           feeReceiver = _new;
339:      }

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L336-L339

### [N&#x2011;06]  NatSpec is incomplete

*There are 7 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

/// @audit Missing: '@return'
344       /// @param _poolId ID of the pool
345       /// @param _bidId ID of the bid in queue
346       function getBidInQueue(uint256 _poolId, uint256 _bidId)
347           public
348           view
349           returns (
350               uint256 bidId,
351               address owner,
352               uint256 price,
353:              uint256 quantity

/// @audit Missing: '@return'
363       /// @notice Gets minimum bid price of queue for given pool
364       /// @param _poolId ID of the pool
365:      function getMinPrice(uint256 _poolId) public view returns (uint256) {

/// @audit Missing: '@return'
369       /// @notice Gets next bidId in queue of given pool
370       /// @param _poolId ID of the pool
371:      function getNextBidId(uint256 _poolId) public view returns (uint256) {

/// @audit Missing: '@return'
375       /// @notice Gets total number of bids in queue for given pool
376       /// @param _poolId ID of the pool
377:      function getNumBids(uint256 _poolId) public view returns (uint256) {

/// @audit Missing: '@return'
382       /// @param _poolId ID of the pool
383       /// @param _bidId ID of the bid in queue
384:      function getBidQuantity(uint256 _poolId, uint256 _bidId) public view returns (uint256) {

/// @audit Missing: '@return'
389       /// @param _poolId ID of the pool
390       /// @param _owner Address of the owner
391       function getOwnerToBidIds(uint256 _poolId, address _owner)
392           public
393           view
394:          returns (uint256[] memory)

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L344-L353

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

/// @audit Missing: '@return'
44        /// @param _order Bytes value of the necessary order parameters
45        /// return vault Address of the deployed vault
46:       function execute(bytes memory _order) external payable returns (address vault) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L44-L46

### [N&#x2011;07]  Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*There are 4 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

253:                  bid.quantity = 0;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L253

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

293:              activeListing.collateral = 0;

325:          pendingBalances[_vault][_to] = 0;

437:          orderParams.totalOriginalConsiderationItems = 0;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L293

### [N&#x2011;08]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol


```

### [N&#x2011;09]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol


```
