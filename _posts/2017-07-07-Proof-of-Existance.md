---
layout: post
title: A Transaction with two keys and Checklocktimeverify
---

Alice and Bob each create their own keypair. 
Bob creates the following transaction script:

```javascript
IF
    <now + 3 months> CHECKLOCKTIMEVERIFY DROP
    <Bob's pubkey> CHECKSIGVERIFY
ELSE
    <Alice's pubkey> CHECKSIGVERIFY
ENDIF
```

Bob then hashes the script, encodes it as a P2SH address and sends the addresss to Alice.
If Alice sends any Bitcoin to this address, she can redeem it with the following scriptSig: 
```javascript 
0 <Alice's signature> 0
```
After 3 months have passed, Bob can also redeem the transaction output Alice created with the following
```
0 <Bob's signature> 1
```
