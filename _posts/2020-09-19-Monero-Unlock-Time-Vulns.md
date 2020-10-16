---
layout: post
title: Monero timelock vulnerabilities
image: /images/timestamp_diff.png
---

*TLDR: Monero, and in general cryptonote based cryptocurrencies, transaction
unlock_time values were interpreted with local time, allowing a host of
non-critical exploits targeting the integrity of the blockchain. The issue is
patched as of monero version 0.17.0.0. No harm towards user funds was found
related to the issue.*

Next to containing code activating the new CLSAG signature algorithm, the
v0.17.0.0 release of monero also contains a security patch. The patch was
called "Deterministic Unlock Times" in the release notes. The reason for this
new functionality was not provided before, which led to some speculation on its
nature. It is now given both in this article and the HackerOne disclosure. This
is the second article in my monero unlock time series. The first article is
[here](../Wallet-Timelock).

## Description

Let's quickly recap what a monero transaction roughly contains. Similarly to
bitcoin, monero transactions have inputs and outputs. Outputs represent the
amounts potentially available for further spending, inputs represent the now
consumed outputs from a previous transaction. Additionally, a monero
transaction selects as an input from n previous transaction outputs (ring
members) and proves with a ring signature that the author of the transaction
owns one of them. The transaction author then constructs new outputs
from these inputs and cryptographically proves that the amounts balance.

The unlock\_time field in a monero transaction dictates when coins in a
transaction can be spent again, or more exactly when a transaction's outputs
can be spent again. No transactions can use these outputs until the timelock
has timed out. Money transferred in time-locked transactions is only spendable
after a certain time. 

The rule is enforced by the consensus code in
[`src/cryptonote_core/blockchain.cpp`](https://github.com/monero-project/monero/blob/release-v0.16/src/cryptonote_core/blockchain.cpp#L3554)
and has two parts. If the `unlock_time` is below 500'000'000, it is interpreted
as a block height and compared with the current block height. If the
`unlock_time` in a used transaction input's previous output is below the
current block height, a transaction using that output is valid. Otherwise, it
is invalid and not relayed.

When the unlock time is above 500'000'000, it is interpreted as Unix time in
seconds rather than block height. Surprisingly these time-based `unlock_times`
were not compared to the network or rather block time, but to the local time of
the machine running the monero node. Using this local time is problematic.
Nodes should use as much shared blockchain data as possible when verifying
transactions, or concisely: *Consensus rules must react to consensus
variables*. Since time varies from machine to machine, it is not a consensus
variable.

I shared this issue with monero research lab regular contributor Isthmus
([@Mitchellpkt0](https://twitter.com/Mitchellpkt0) on twitter) and we explored
the following vulnerabilities. I do consider them serious enough to have
warranted a report and some degree of secrecy, but I don't believe them to be
critical, nor particularly dangerous.

## Exploits

By consuming outputs locked with carefully chosen `unlock_time` values, it is
possible to poll the local time of other nodes. The attacker can generate any
number of transactions encumbered with a spread of `unlock_time` values, for
example with an `unlock_time` 20 blocks per the expected block time into the
future and then spread out over intervals of seconds. The attacker then creates
transactions consuming these locked outputs and connects two nodes to the node
she wants to spy on. She sends the transaction from one of the nodes to the
node she wants to spy on and then checks if the other node receives the
transaction. If the consumed `unlock_time` is invalid by the consensus rules for
the attacker and the greater network, but valid for the surveilled node, the
attacker will get the transaction relayed to his other node as well. This then
indicates the local time on the surveilled node matching the time on the
attacker's transaction. Using a binary search methodology, the attacker can
then pinpoint the node's time within a precision of seconds.

If such a node is found with a different clock (running late or early), or if
the attacker can manipulate a node's local time through another channel, we
identified two additional ways to exploit this `unlock_time` validation.

Assume a node has a clock running forward (having a higher Unix time). The
attacker creates a transaction consuming at least one output with a high enough
`unlock_time` such that it is invalid by the clocks of most nodes on the
network, but valid for the node where the clock runs forward. If the attacker
relays this transaction to the victim's node, the node will validate it as a
valid transaction. This is especially useful for the attacker in the context of
mining. For example, if the victim's node is a large mining pool, this can be
used to make the pool expend work on a block that is not valid with the locked
transaction included by the rules of the rest of the network. If the
attacker is a mining pool herself, she can use this to increase her profit.

If a node has a clock running behind (showing a lower Unix time), transactions
consuming outputs encumbered by a recently expired `unlock_time` are valid and
minable for most nodes on the network, but not for nodes with a lower local
Unix time. In fact, any nodes with a lower local Unix time will reject any
transactions and blocks containing transactions spending outputs that are still
locked from their point of view. Explaining this in words may be confusing:
Assume someone creates a transaction spending an output encumbered with
`unlock_time` 1500000020, while the actual current Unix time is 1500000040.
Monero nodes running on a system with the correct actual Unix time will accept
this transaction since the `unlock_time` has expired. Nodes with a lower local
Unix time, say 1500000000 will not accept this transaction. From their
perspective the `unlock_time` is not expired, rendering the transaction
invalid.

An attacker can continuously submit transactions consuming an `unlock_time`
encumbered output with a value just later than the victim node's clock,
ensuring the node will never catch up to the tip of the network's best chain.
This could be used for a short term eclipse attack, even potentially allowing
the attacker enough time to feed it a "slower" malicious chain.  Once the
node's time has eventually surpassed the `unlock_time`, the node can accept the
block containing the time-locked transaction from the best chain again.  Since
connections with peers are dropped if they feed 'invalid' blocks, there is no
telling for how long exactly the attack could be sustained, because the node
might first have to accept connections with its peers again. This form of
eclipse attack could be used against an exchange (if the exchange node's clock
is wrong) to double spend.

## Some Mitigating Factors and Data

To assess the risk of this happening without a further exploit enabling NTP
time or machine time manipulation, I produced the following graph:

![Timestamps](/images/timestamp_diff.png
"Successive Block Timestamp Differences")

It shows the timestamp difference between two successive blocks. Negative time
indicates at least one of the two blocks having a block timestamp
differentiating from the actual time. Blocks produced in the past cannot appear
after new blocks! We can use all these negative differences as a rough
estimate, bar any real understanding of the statistical underpinning, for
blocks produced by a machine with a significant time discrepancy. This amounts
to about 2.6% of all produced blocks, which gives us a guestimate of vulnerable
miners to the above exploit without further manipulation.

## Resolution

Following the discovery, we submitted a report to monero's HackerOne outlining
the above problems and containing a potential patch for the issue. 

A similar problem used to exist in bitcoin's `nlocktime` field. However, this
problem was less serious at the time, since the `nlocktime` semantics are
different compared to monero's `unlock_time` (`nlocktime's` influence is on the
finality of the transaction itself, not a later transaction), and since the
last block's timestamp was used instead of the local time. It was fixed through
[BIP113](https://github.com/bitcoin/bips/blob/master/bip-0113.mediawiki) and
now uses the median timestamp of the past 11 blocks to determine if an unlock
time is valid. The fix was also a required step for the deployment of the CSV
and CLTV opcodes, which adopt a similar output locking mechanism to the one of
monero's `unlock_time`.

The newest version of monero contains a change in the consensus rules making
the Unix time-based `unlock_time` dependent on the timestamp of previous blocks.
The original patch was much inspired by how bitcoin handles its time-based
unlock time values. It took the median block timestamp of the past 60 blocks
and compared it to the unlock time value.

This patch was then expanded upon by other reviewers until settling for the
formula 

`min(median(block timestamps of the past 60 blocks) + 62 minutes,`
`current tip block timestamp + 2 minutes)`

It aims to more accurately represent the current time. 

The full contents and discussion are [available on
GitHub](https://github.com/monero-project/monero/pull/6745). These new
validation rules were activated with the latest monero hardfork together with
the CLSAG transaction signature and format validation.

## Closing notes on the monero code

Interestingly the code already contained a function `get_adjusted_time` that
just returned the local time of the system running monerod. It did have some
ominous comments indicating this was not its [intended form](
https://github.com/monero-project/monero/blob/release-v0.16/src/cryptonote_core/blockchain.cpp#L3638).
The function has now been repurposed to calculate this actual "adjusted" time.

I found this vulnerability while reading through the TODO comments in the code.
Monero inherited a largely uncommented source code from the original cryptonote
developers. Particularly hazy areas are commented with either //TODO or
//FIXME. I would strongly recommend other security researchers to take a tour
through these. There are roughly 120 TODOs and 30 FIXMEs left, so there may be
more to find. 

