---
layout: post
title: Monero timelock woes
image: /images/unlock_time_stats.png
---

*TLDR: In this last in a series of three monero unlock_time related posts, I dig
into the privacy considerations of current unlock_time use and how it can be
improved by either encrypting the field, restricting its content, tweaking
ring selection or removing it altogether.*

The last two posts (please read them first) detailed both [implementation [1]
](../Wallet-Timelock) and [protocol [2] ](../Monero-Unlock-Time-Vulns)
vulnerabilities with the monero transaction `unlock_time` field, both in
applications that should have validated the field and in the core monero
software. Research done in January 2020 by #monero-research-lab (a Freenode IRC
channel facilitating research discussion) data scientist Isthmus
([@Mitchellpkt0](https://twitter.com/Mitchellpkt0) on Twitter) showed that the
`unlock_time` is not well understood by many users of monero either. Its usage
leaves information fingerprints that may be detrimental to the privacy of its
users. After submitting the disclosures described in the last two articles, I
wrote a proposal in late April this year towards implementing encryption for
the `unlock_time` to solve this privacy issue. The proposal received little
backing at the time, for reasons I will dig into below, and I did not pursue
its implementation. Parts of that proposal, which was co-authored by Isthmus,
is now reworked into this article. Thanks also to Isthmus' colleague N3ptune
who provided the raw data.

### `unlock_time` usage patterns

Monero’s privacy is highly dependent upon transaction indistinguishability to
prevent transaction linkability. Any characteristic of the transaction
revealing something about the user or software that created it reduces
anonymity. One such example is the statistical analysis of `unlock_time`, which
partitions the Monero anonymity pool based on the software and the user that
generated the transactions. The data presented here was collected from block
1'000'000 to 2'197'574.

Roughly five different `unlock_time` usage patterns emerge:

1. unlock_time = 0
* Default behavior of the core wallet (and any that properly mimic it)
* An output that is “spendable” at the genesis block is never locked
* Only 12493 transactions have been recorded not setting 0
* Example:
  [e4098567e981e9596f8b2a449c7df24cc77268ff08280b5901b624c2de234202](https://localmonero.co/blocks/search/e4098567e981e9596f8b2a449c7df24cc77268ff08280b5901b624c2de234202)
2. unlock_time = {1...6,10,12,13,15}
* These low-integer `unlock_time` values do not make semantic sense, so the developer’s intent is unclear. Perhaps they thought block times were relative rather than absolute, or the field is used for something else unrelated to `unlock_time`.
* In total 12297 transactions, ~98% of current `unlock_time` usage
* Example: [bf800d30889423fafdf7cde841f1a61d3372667a0efc7c6e8784f220c0dcc3a8](https://localmonero.co/blocks/search/bf800d30889423fafdf7cde841f1a61d3372667a0efc7c6e8784f220c0dcc3a8)
3. unlock_time ~ 1'000'000+
*  `unlock_time` less than 500,000,000 is interpreted as block height
* 195 transactions, ~2% of current `unlock_time` usage
* Example:
  [93df46c18742ff6fd0ba86076bd360b0a32cda4f670b9944c9f176d5c9783959](https://localmonero.co/blocks/search/93df46c18742ff6fd0ba86076bd360b0a32cda4f670b9944c9f176d5c9783959)
4. unlock_time ~1'400'000'000
* Large `unlock_time` represents Unix epoch timestamps
* No transactions in this range recorded!
* Example:
  [012932593e59f21d10b7badc5f0556c1aaaefd60d0ebf05f1637361a66b17273](https://localmonero.co/blocks/search/012932593e59f21d10b7badc5f0556c1aaaefd60d0ebf05f1637361a66b17273)
5. unlock_time >1'400'000'000'000
* These outputs theoretically unlock far into the future
* A single transaction with 18446744073709551616, [created by Isthmus](https://twitter.com/Mitchellpkt0/status/1251621277179146240) and locked until the year 292'277'026'596.
* Example:
  [2c2762d8817ea4d1cb667752698f2ff7597a051d433043776945669043d908b5](https://localmonero.co/blocks/search/2c2762d8817ea4d1cb667752698f2ff7597a051d433043776945669043d908b5)

Isthmus also produced the following plot on `unlock_time` usage (and the one
below):

![unlock_time](/images/unlock_time_stats.png "unlock_time dot graph")

The usage distribution of low values is strange (top row `unlock_time`
values, bottom row counts of their occurrence):

<style>
table, th, td {
  border: 1px solid black;
  table-layout: fixed;
}
</style>

<table style="width:100%">
  <tr>
    <td>1</td>
    <td>2</td>
    <td>3</td>
    <td>4</td>
    <td>5</td>
    <td>6</td>
    <td>10</td>
    <td>12</td>
    <td>13</td>
    <td>15</td>
  </tr>
  <tr>
    <td>8141</td>
    <td>279</td>
    <td>677</td>
    <td>266</td>
    <td>6</td>
    <td>1</td>
    <td>297</td>
    <td>2600</td>
    <td>1</td>
    <td>1</td>
  </tr>
</table>

I have two explanations for this behavior: Either somebody forgot to add the
current block height on top of the desired amount of locked blocks, or they are
misusing the field to convey some extra information.

Peculiar patterns, like these low values, enable transaction linkability. Not
only the sender and the receiver are affected and might be de-anonymized, but
also any user whose ring signatures selected these outputs as mix-ins. In a
decoy-based anonymity scheme, a user cannot negatively impact privacy without
affecting others.

Monero is not the only cryptocurrency exhibiting information leaks due to the
erratic use of `unlock_time`. Other privacy coins have even seen their time
locks used for steganography ([see
thread)](https://twitter.com/f2pool_official/status/1246154346481381378).

The following graph shows the difference between a meaningful block height
`unlock_time` and its mined height, revealing the actual number of blocks each
transaction is locked for:

![unlock_time](/images/diff_height_lock.png "unlock_time difference from block height")

I have not found an explanation for the patterns revealed here. My expectation
would have been to see horizontal lines produced by services using the feature
to set a specific amount of blocks. Instead, some vertical patterns and a skew
cluster after the 2'000'000 block height appear.

### `unlock_time` and ring member selection

Apart from these clusters, transactions setting a meaningful `unlock_time`
introduce another privacy problem. Monero chooses decoy participants in its
ring signatures by sampling past transaction outputs from the monero
blockchain. The sampling tries to mimic spending behavior. Users tend to spend
younger outputs more than older outputs, so the selection prefers transactions
mined at a high block height than a lower block height. The selection does
however not calculate the `unlock_time` on top of the actual block height.
Assume the current block height is 400'000.  By the current rules, an output
with `unlock_time` 350'000 mined at block height 200'000 is treated the same as
an output at 200'000, even though the `unlock_time` encumbered output has only
been available for spending for 50'000 blocks, while the other output has been
for 200'000. This weakness in the selection algorithm leaks information,
allowing an observer of the monero blockchain to guess which member in the ring
signature is a decoy, and which might be the true spender.

The solution is to take `unlock_time` related statistical spend age
behavior into account when selecting decoy ring members. The spend age measures
the amount of time that passes between maturity of a transaction's output and
its subsequent consumption as a transaction input.  Statistical spend age could
be gathered from a transparent blockchain with timelocks, like bitcoin, and
applied as a heuristic on monero. This oversight in the selection algorithm is
currently not problematic, since the `unlock_time` is hardly used.

### timelock encryption

A brutish solution would be to remove the field entirely. Its usage is low and
may leak information. As an alternative, adding per-output timelocks and
restricting the field size to a more compact data type, for example a 2-byte
number encoding time in steps of hours with a 1-bit flag to choose block or
time-based, could significantly reduce misuse. Additionally transactions with
non-zero timelocks that are defined in the past could be blocked by consensus
rules.

A more sophisticated approach is the implementation of encrypted timelocks,
which won't leak user/software fingerprints since the ciphertext is uniformly
distributed. The timelock values would be encrypted similarly to the current
monero amount encryption. A commitment to zero proving that the timelock has
expired is added to the existing ring signature construction. A bulletproof
proving the timelock value in its integer range has to be provided as well.
The essence of these extra encryption steps is described in [monero research
lab issue # 65](https://github.com/monero-project/research-lab/issues/65) and
the [DLSAG paper](https://eprint.iacr.org/2019/595.pdf). 

DLSAG is a possible extension to the monero transaction signing algorithm that
uses the timelock to build powerful new transaction primitives allowing the
construction of payment channels and payment channel networks on monero. Not
encrypting the timelock values would be detrimental to the transaction
privacy of its users.

Besides my toy rust implementation of [timelock encryption with
CLSAG](https://github.com/TheCharlatan/rs-xmr-cryp/blob/master/timelock/src/main.rs),
\#monero-research-lab cryptographer Sarang Noether wrote a c++ implementation
for [CLSAG, the current monero signing
algorithm,](https://github.com/SarangNoether/monero/tree/3-clsag-update) and
[Triptych, a potential future signing
algorithm,](https://github.com/SarangNoether/monero/tree/3-triptych) from which
he generated the following benchmarks comparing signature verification time
(3-CLSAG and 3-Triptych are the unlock time variants):

![benchmarks](/images/encrypted_locktime_benchmarks.png "encrypted
`unlock_time` benchmarks")

Encrypted timelocks have serious performance downsides. Besides the shown
significant increase in verification time (around 2x) for both a current and
possible future algorithm, they also require an extra 64 bytes per Signature in
the transaction and 32 bytes per extra timelock encumbered output if
implemented on top of the CLSAG signing algorithm. A significant increase in
both size and verification time of the range proof is also expected. The design
of the optimal range proof construction is still unclear, so there are no
concrete numbers for the performance hit on the range proof.

Due to these performance drawbacks and low current usage of `unlock_time` among
the monero userbase, there was little enthusiasm for encrypted timelock
deployment.

### closing remarks

Let's recap on the problems discovered so far: `unlock_time` had a buggy
implementation, seems to be easily overlooked by developers, and is detrimental
to monero user's privacy. On top of that, it is difficult or rather dangerous
to use. It locks the entire transaction, not singular outputs. Once set, any
moneroj in a change or other recipient's output is also locked!

I find a penalty beyond a doubling in size and verification time for encrypted
timelocks unlikely. Not compromising on giving users the safest privacy and
security options by default has been a tenet of monero development so far. I
believe that if monero seeks to maintain its status as the dominant
privacy-oriented cryptocurrency, performance penalties on this order should not
stifle privacy and security improvements. However, this scale of engineering
work does seem overblown compared to the little amount of usage `unlock_time`
has. In my opinion, the best way forward currently is to remove it entirely.

