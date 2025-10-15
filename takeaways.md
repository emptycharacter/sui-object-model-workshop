# PTB Workshop - Key Takeaways

**Completed:** October 15, 2025
**Workshop Focus:** Programmable Transaction Blocks (PTBs) on Sui

---

## What Are PTBs?

Programmable Transaction Blocks are Sui's killer feature - they let you chain multiple operations together atomically. If any step fails, the entire transaction rolls back. This is way more powerful than traditional blockchain transactions.

---

## Exercise 1: Handling Returned Objects

**Lesson:** Objects returned from Move functions MUST be handled (transferred, wrapped, or destroyed).

**What I Learned:**
- Move best practice: Return objects instead of self-transferring them
- Using `tx.moveCall()` to call Move functions
- Using `tx.transferObjects()` to handle returned objects
- If you don't handle a returned object, the transaction fails

**Transaction:** `6GXhP7c42wzX77KUWaC4uU1tBpTbDSkcrKGDmy5NW8Sy`

**Key Code:**
```typescript
const nft = tx.moveCall({
  target: `${PACKAGE_ID}::sui_nft::new`,
});
tx.transferObjects([nft], suiAddress);
```

---

## Exercise 2: Objects as Input

**Lesson:** How to pass existing on-chain objects to Move functions.

**What I Learned:**
- Using `tx.object()` to reference on-chain objects
- Using `tx.splitCoins()` to create fee payments from gas
- Shared objects can be accessed by anyone
- Mutable references (`&mut`) allow modifying objects on-chain

**Transaction:** `F6ANMeu8iAkb7ytzunJKkK75Xii4zLwPVpqo6REaG1hg`

**Key Code:**
```typescript
const [feeCoin] = tx.splitCoins(tx.gas, [10]);
tx.moveCall({
  target: `${PACKAGE_ID}::counter::increment`,
  arguments: [tx.object(COUNTER_OBJECT_ID), feeCoin],
});
```

---

## Exercise 3: Scavenger Hunt (PTB Power!)

**Lesson:** Chain multiple operations atomically - this is the power of PTBs.

**What I Learned:**
- Multiple moveCall operations in one transaction
- Results from one call can be used in the next call
- Type arguments for generic Move functions (e.g., `typeArguments: ['0x2::sui::SUI']`)
- Querying on-chain objects to find data (found secret code: 745223)
- ALL operations succeed or ALL fail - atomic execution

**Transaction:** `5mNSsBg7VE3Ebj1q7972A6hdv2LhDn55tZ9fY3LoVUU1`

**Key Code:**
```typescript
// All in ONE atomic transaction:
const key = tx.moveCall({ target: `${PACKAGE_ID}::key::new` });
tx.moveCall({
  target: `${PACKAGE_ID}::key::set_code`,
  arguments: [key, tx.pure.u64(745223)]
});
const coin = tx.moveCall({
  target: `${PACKAGE_ID}::vault::withdraw`,
  arguments: [tx.object(VAULT_ID), key],
  typeArguments: ['0x2::sui::SUI'],
});
tx.transferObjects([coin], keypair.toSuiAddress());
```

---

## Overall Takeaways

1. **PTBs are powerful:** Chain operations that would take 4 separate transactions on other blockchains into 1 atomic transaction
2. **Type safety:** Sui's Move VM ensures type correctness at runtime
3. **Gas efficiency:** One transaction = one gas payment (vs 4 transactions on other chains)
4. **Developer experience:** The TypeScript SDK makes building PTBs intuitive
5. **Object model:** Sui's object-centric model is different from account-based blockchains - takes some getting used to but very powerful

---

## Skills Gained

-  Building transactions with `@mysten/sui/transactions`
-  Calling Move functions from TypeScript
-  Handling returned objects properly
-  Passing objects as function arguments
-  Working with type arguments for generic functions
-  Chaining multiple operations atomically
-  Querying on-chain object data
-  Testing on Sui testnet

---

## Why This Matters

Understanding PTBs is essential for Sui development. They're not just a nice feature - they're how you build anything complex on Sui. Every DeFi swap, NFT mint, or game action will likely use PTBs to chain operations together safely and efficiently.
