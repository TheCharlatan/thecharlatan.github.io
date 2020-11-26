---
layout: post
title: Hardware Wallet Coin Isolation Bypass
image: /images/keepkey.jpeg
---

*TLDR: Security researcher monokh discovered a flaw in the isolation of the
transaction logic between coins. It allows coins from one cryptocurrency, e.g.
bitcoin, to be spent by the transaction flow of another, e.g. Litecoin. I found
two other hardware wallets that are vulnerable to this exploit, Keepkey and
Coldcard. To this day, Keepkey has not patched the issue.*

### Preface

Security and bitcoin researcher [monokh](https://twitter.com/mo_nokh) published
an [article](https://monokh.com/posts/ledger-app-isolation-bypass) on his blog
on August 04 2020 titled "Ledger App Isolation Bypass". It describes a flaw in
the isolation between the various apps that can be installed on a Ledger. From
monokh's post:

*It was discovered that for Bitcoin and Bitcoin forks, the device exposes its
functions for any of the assets. In other words, having unlocked the Litecoin
app, you will receive a confirmation request for a Bitcoin transfer while the
interface presents it as a transfer of Litecoins to a Litecoin address.
Accepting the confirmation produces a fully valid signed Bitcoin (mainnet)
transaction.*

Essentially keys intended for one cryptocurrency could be accessed by another,
allowing coins with the same signature format to be cross-spent by each other.
This breaks the most fundamental assumption of a hardware wallet: You sign what
you see.

A few days later Trezor acknowledged another security researcher discovering a
very similar vulnerability in the Trezor One. Again the user could be tricked
into signing a Bitcoin transaction while using the workflow of an Altcoin
transaction. Trezor describes this in a [blog
post](https://blog.trezor.io/firmware-updates-for-trezor-model-t-version-2-3-2-and-trezor-model-one-version-1-9-2-f4f9c0f1ed7c).

### Hardware wallet vulnerabilities across vendors

I started tracking vulnerabilities across most hardware wallet vendors a year
ago in my [list of hardware wallet hacks](../List-Of-Hardware-Wallet-Hacks). I
observed that vendors tend to repeat their mistakes a lot. These range from
non-constant time PIN-checks to insufficient checks on change addresses and
signature replay attacks.

With the two largest hardware wallet vendors, Trezor and Ledger, making the same
mistake, I suspected other vendors repeating their mistake as well. I also
saw many posts on Twitter and Reddit blaming the vulnerability on the
altcoins, concluding a hardware wallet not supporting altcoins is
not vulnerable to the attack. The owner of the Coldcard parent company coinkite
even tweeting in response to the vulnerability:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Shitcoins are a hardware wallet liability.  Coldcard unaffected 😉</p>&mdash; UNCORELETED OPTIMISM (@nvk) <a href="https://twitter.com/nvk/status/1290635643752767489">August 4, 2020</a></blockquote>
<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

However, the vulnerability also extends to Bitcoin testnet. The user could be
visually verifying a testnet transaction, while actually signing mainnet.

### Wallets with monokh's vulnerability

In the following weeks, I tested monokh's exploit on the Cobo, Keepkey,
Coldcard and BitBox02 wallet. Both Cobo and the BitBox02 were not vulnerable.
Keepkey is based on the Trezor One, so it was no surprise that it had no checks
enforcing the isolation.  Coldcard also had no checks enforcing isolation
between Bitcoin testnet and mainnet.  Independently, benma, my ex-colleague
from my days working on the BitBox02, had found and disclosed the vulnerability
to Coldcard already. Benma also publicly disclosed the issue in a [blog
post](https://benma.github.io/2020/11/24/coldcard-isolation-bypass.html)
without Coldcard having patched the issue yet.  This left me to contact
Keepkey's, or rather Shapshift's, security team on August 20. 

Keepkey was unable to deliver a fix within a typical 90 day disclosure period.
Though they were responsive to my messages, they were unable to commit to a
patch timeline. Since the outline of the vulnerability is public knowledge
already, I am now publicly disclosing the vulnerability.

Keepkey recently fixed a host of exploits in a recent release, which they
detail in a [blog
post](https://shapeshift.com/library/keepkey-firmware-update-6-5-1). I am
disappointed that the Keepkey team is not committed to patching vulnerabilities
in their hardware wallet within reasonable time. I therefore strongly recommend
against buying one or promoting its use.

### Updates

Coldcard provided a fix to the issue in their version 3.2.0 release by moving
the testnet screen behind an additional "Danger Zone" warning screen, which
they explain in another [blog
post](https://blog.coinkite.com/testnet-considered-useful/). Since this extra
step protects users from being duped into the testnet workflow, I consider this
a fix for the issue.

