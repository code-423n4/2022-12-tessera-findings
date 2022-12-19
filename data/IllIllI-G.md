
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate | 2 |  - |
| [G&#x2011;02] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 4 |  480 |
| [G&#x2011;03] | Using `storage` instead of `memory` for structs/arrays saves gas | 1 |  4200 |
| [G&#x2011;04] | State variables should be cached in stack variables rather than re-reading them from storage | 4 |  388 |
| [G&#x2011;05] | Multiple accesses of a mapping/array should use a local variable cache | 4 |  168 |
| [G&#x2011;06] | `internal` functions only called once can be inlined to save gas | 5 |  100 |
| [G&#x2011;07] | `<array>.length` should not be looked up in every loop of a `for`-loop | 2 |  6 |
| [G&#x2011;08] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 2 |  120 |
| [G&#x2011;09] | Optimize names to save gas | 5 |  110 |
| [G&#x2011;10] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 2 |  10 |
| [G&#x2011;11] | Using `private` rather than `public` for constants, saves gas | 2 |  - |
| [G&#x2011;12] | Inverting the condition of an `if`-`else`-statement wastes gas | 1 |  - |
| [G&#x2011;13] | Division by two should use bit shifting | 3 |  60 |
| [G&#x2011;14] | Use custom errors rather than `revert()`/`require()` strings to save gas | 2 |  - |

Total: 39 instances over 14 issues with **5642 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There are 2 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

28        mapping(uint256 => address) public poolToVault;
29        /// @notice Mapping of pool ID to PoolInfo struct
30        mapping(uint256 => PoolInfo) public poolInfo;
31        /// @notice Mapping of pool ID to the priority queue of valid bids
32        mapping(uint256 => MinPriorityQueue.Queue) public bidPriorityQueues;
33        /// @notice Mapping of pool ID to amount of Raes currently filled for the pool
34        mapping(uint256 => uint256) public filledQuantities;
35        /// @notice Mapping of pool ID to minimum ether price of any bid
36        mapping(uint256 => uint256) public minBidPrices;
37        /// @notice Mapping of pool ID to minimum reserve prices
38        mapping(uint256 => uint256) public minReservePrices;
39        /// @notice Mapping of pool ID to total amount of ether contributed
40        mapping(uint256 => uint256) public totalContributions;
41        /// @notice Mapping of pool ID to user address to total amount of ether contributed
42:       mapping(uint256 => mapping(address => uint256)) public userContributions;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L28-L42

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

50        mapping(address => bytes32) public vaultOrderHash;
51        /// @notice Mapping of vault address to active listings on Seaport
52        mapping(address => Listing) public activeListings;
53        /// @notice Mapping of vault address to newly proposed listings
54        mapping(address => Listing) public proposedListings;
55        /// @notice Mapping of vault address to user address to collateral amount
56:       mapping(address => mapping(address => uint256)) public pendingBalances;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L50-L56

### [G&#x2011;02]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There are 4 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

/// @audit _purchaseProof
160       function purchase(
161           uint256 _poolId,
162           address _market,
163           address _nftContract,
164           uint256 _tokenId,
165           uint256 _price,
166           bytes memory _purchaseOrder,
167:          bytes32[] memory _purchaseProof

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L160-L167

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

/// @audit _order
46:       function execute(bytes memory _order) external payable returns (address vault) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L46

```solidity
File: src/seaport/targets/SeaportLister.sol

/// @audit _orders
26:       function validateListing(address _consideration, Order[] memory _orders) external {

/// @audit _orders
51:       function cancelListing(address _consideration, OrderComponents[] memory _orders) external {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L26

### [G&#x2011;03]  Using `storage` instead of `memory` for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

*There is 1 instance of this issue:*

```solidity
File: src/modules/GroupBuy.sol

408:              Bid memory bid = queue.bidIdToBidMap[index];

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L408

### [G&#x2011;04]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 4 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

/// @audit currentId on line 74
83:           minBidPrices[currentId] = _initialPrice / _totalSupply;

/// @audit currentId on line 83
86:           bidPriorityQueues[currentId].initialize();

/// @audit currentId on line 86
89:           emit Create(currentId, _nftContract, _tokenIds, msg.sender, _totalSupply, _duration);

/// @audit currentId on line 89
92:           contribute(currentId, _quantity, _raePrice);

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L83

### [G&#x2011;05]  Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

*There are 4 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

/// @audit bidPriorityQueues[_poolId] on line 299
320:                  bidPriorityQueues[_poolId].insert(msg.sender, _price, quantity);

/// @audit bidPriorityQueues[_poolId] on line 320
334:                  bidPriorityQueues[_poolId].delMin();

/// @audit bidPriorityQueues[_poolId] on line 334
336:                  bidPriorityQueues[_poolId].insert(msg.sender, _price, lowestBidQuantity);

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L320

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

/// @audit activeListings[_vault] on line 107
115:              _pricePerToken >= activeListings[_vault].pricePerToken

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L115

### [G&#x2011;06]  `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

*There are 5 instances of this issue:*

```solidity
File: src/modules/GroupBuy.sol

419       function _generateRoot(uint256[] calldata _tokenIds)
420           internal
421           pure
422:          returns (bytes32 merkleRoot)

466       function _verifySuccessfulState(uint256 _poolId)
467           internal
468           view
469           returns (
470               address,
471               uint48,
472               uint40,
473               bool,
474:              bytes32

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L419-L422

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

71        function _deployVault(uint256 _punkId)
72            internal
73:           returns (address vault, bytes32[] memory unwrapProof)

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L71-L73

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

368       function _constructOrder(
369           address _vault,
370           uint256 _listingPrice,
371:          OfferItem[] calldata _offer

481       function _getOrderHash(OrderParameters memory _orderParams, uint256 _counter)
482           internal
483           view
484:          returns (bytes32 orderHash)

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L368-L371

### [G&#x2011;07]  `<array>.length` should not be looked up in every loop of a `for`-loop
The overheads outlined below are _PER LOOP_, excluding the first loop
* storage arrays incur a Gwarmaccess (**100 gas**)
* memory arrays use `MLOAD` (**3 gas**)
* calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset

*There are 2 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

98:           for (uint256 i = 0; i < curUserBids.length; i++) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L98

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

390:              for (uint256 i = 0; i < _offer.length; ++i) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L390

### [G&#x2011;08]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

*There are 2 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

98:           for (uint256 i = 0; i < curUserBids.length; i++) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L98

```solidity
File: src/modules/GroupBuy.sol

248:              for (uint256 i; i < length; ++i) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L248

### [G&#x2011;09]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 5 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

/// @audit initialize(), isEmpty(), getNumBids(), getMin(), insert(), delMin()
12:   library MinPriorityQueue {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L12

```solidity
File: src/modules/GroupBuy.sol

/// @audit createPool(), contribute(), purchase(), claim(), withdrawBalance(), getBidInQueue(), getMinPrice(), getNextBidId(), getNumBids(), getBidQuantity(), getOwnerToBidIds(), printQueue()
20:   contract GroupBuy is IGroupBuy, MerkleBase, Minter {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L20

```solidity
File: src/punks/protoforms/PunksMarketBuyer.sol

/// @audit execute()
16:   contract PunksMarketBuyer is IPunksMarketBuyer, Protoform {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/punks/protoforms/PunksMarketBuyer.sol#L16

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

/// @audit propose(), rejectProposal(), rejectActive(), list(), cancel(), cash(), withdrawCollateral(), updateFeeReceiver()
23:   contract OptimisticListingSeaport is

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L23

```solidity
File: src/seaport/targets/SeaportLister.sol

/// @audit validateListing(), cancelListing()
15:   contract SeaportLister is ISeaportLister {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/targets/SeaportLister.sol#L15

### [G&#x2011;10]  `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
Saves **5 gas per loop**

*There are 2 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

60:                   j++;

98:           for (uint256 i = 0; i < curUserBids.length; i++) {

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L60

### [G&#x2011;11]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 2 instances of this issue:*

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

38:       bytes32 public immutable conduitKey;

44:       uint256 public immutable PROPOSAL_PERIOD;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L38

### [G&#x2011;12]  Inverting the condition of an `if`-`else`-statement wastes gas
Flipping the `true` and `false` blocks instead saves ***[3 gas](https://gist.github.com/IllIllI000/44da6fbe9d12b9ab21af82f14add56b9)***

*There is 1 instance of this issue:*

```solidity
File: src/seaport/modules/OptimisticListingSeaport.sol

289           if (!_verifySale(_vault)) {
290               revert NotSold();
291           } else if (activeListing.collateral != 0) {
292               uint256 collateral = activeListing.collateral;
293               activeListing.collateral = 0;
294               // Sets collateral amount to pending balances for withdrawal
295               pendingBalances[_vault][activeListing.proposer] = collateral;
296:          }

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L289-L296

### [G&#x2011;13]  Division by two should use bit shifting
`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the `SHR` opcode to accomplish both, the version that uses division incurs an overhead of [**20 gas**](https://gist.github.com/IllIllI000/ec0e4e6c4f52a6bca158f137a3afd4ff) due to `JUMP`s to and from a compiler utility function that introduces checks which can be avoided by using `unchecked {}` around the division by two

*There are 3 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

49:           while (k > 1 && isGreater(self, k / 2, k)) {

50:               exchange(self, k, k / 2);

51:               k = k / 2;

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L49

### [G&#x2011;14]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

*There are 2 instances of this issue:*

```solidity
File: src/lib/MinPriorityQueue.sol

42:           require(!isEmpty(self), "nothing to return");

91:           require(!isEmpty(self), "nothing to delete");

```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/lib/MinPriorityQueue.sol#L42
