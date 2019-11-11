---
layout: post
title: A ransom attack on Coldcard's keypath verification
---

All items discussed here have been disclosed to
[Coldcard](https://coldcardwallet.com/)/[Coinkite](https://coinkite.com/) and
have been fixed in their firmware version 3.0.2 release on the 1st of November.
Coldcard has been cooperative with the disclosure and reacted quickly with all
communications. They did not pay a bounty, even though Coinkite at the time of
reporting had a (hackerone account)[https://hackerone.com/coinkite] promising a
bounty, which was supposedly unaffiliated and has been removed in the meantime.
I asked for attribution to myself, Kaspar Etter and Shift Cryptosecurity (the
company him and I work at), but we only received attribution in the GitHub
release notes to myself and only for one of the two vulnerabilities that we
reported.

The Coldcard is a Bitcoin hardware wallet that has gained some popularity
recently, especially through its early adoption of [Partially Signed Bitcoin
Transactions
(BIP174)](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki). They
also advertise their air-gap feature, where information to the device is not
passed through USB but by physically moving a microSD card between device and
host. However, the exploits presented below pass through the ‘air-gap’. 

### Preface

This is the first time for me to be on the reporting end of a disclosure, so I
wanted to write this post to document my experience.

Bitcoin wallets not only need to protect against bad actors that want to steal
your coins.  Wallets also need to ensure that users' access to their coins is
not unrecoverably lost.  In principle, this means that whatever address is used
to receive coins on the wallet should also allow you to spend those coins again.
If an attacker can make a user receive on an address that the user does not have
the keys for, this is a serious vulnerability, since he can ransom the knowledge
on how to spend the coins again.

I disclosed a similar ransom vulnerability to Coldcard as described in this [sia
coin blog
post](https://blog.sia.tech/a-ransom-attack-on-hardware-wallets-534c075b3a92).
My colleague Kaspar Etter also discovered this attack independently on 7
February 2019 at Shift Cryptosecurity AG against the BitBox01. It was fixed with
a release for the BitBox01 on [8 March
2019](https://medium.com/shiftcrypto/bitbox-desktop-app-4-5-0-with-firmware-6-0-2-release-fd77f8186a29).
Later he also realised that a malicious desktop app can not just provide a
non-standard keypath for receive addresses but also for change addresses. Shift
released another fix thereafter for the BitBox01 on [28 March
2019](https://medium.com/shiftcrypto/bitbox-desktop-app-4-6-0-with-firmware-6-0-3-release-ec46937afe7c).

The Coldcard firmware allowed generating addresses on arbitrary BIP32 derivation
paths (a.k.a keypaths). While for the receive address verification the full
keypath was displayed (it still ended up being exploitable, hang on for part
II), the change keypath is not shown when signing a transaction.

### Part I

During the lightning conference in Berlin, I had a conversation with HWI and
Bitcoin Core developer Sjors Provoost, during which he made a passing remark
that something may be weird with Coldcard’s BIP32 keypath verification. When I
came home again after the conference, I decided to take a look at Coldcard’s
firmware code and surely enough did not find a lot in relation to sanitizing and
restricting BIP32 derivation paths.

The Coldcard did have some restrictions on the change keypath when signing a
transaction, namely checking that public keys were generated under a common
keypath prefix. As an example in this post let’s take the prefix `m/44'/0'/0’`.
To understand why checking this common prefix between the input and change
output keypaths is not good enough, we need to take a look at
[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) and
[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) .  These
are the two de-facto standards used by wallets and hardware wallets alike to
generate and manage public and private keys from a single seed. While they do
give good detail on hardening, derivation levels and their responsibilities,
they give no indication on the restriction of the total keypath space.

In practice this means that for each keypath level we can generate `2^31` , or
`2'147'483'648` (4 byte numbers minus 1 bit for encoding) non-hardened and the
same number of hardened keys. A keypath like `m/44'/0'/0'/0/2147483646` is
perfectly valid. If the key at that index is used to generate an address and
receive coins, the wallet would have to first scan all `2'147'483'647` addresses
until it finds the funds.

It gets worse, since we can also go as deep in the keypath tree until the data
structure that holds it goes out of memory to keypaths like:
`m/44'/0'/0/2147483646/2147483646/2147483646/2147483646/2147483646/.../2147483646`.

At this rate we quickly surpass the overall possible keyspace of Bitcoin keys,
making it computationally infeasible to find the key to spend the coins again.
An attacker can use this to create a transaction with a change keypath that is
virtually unspendable without the complete knowledge of it. Hardware wallets
should therefore limit the reachable indexes in the keypath.

To prepare these transactions and communicate with the Coldcard, I modified the
[Electrum wallet](https://electrum.org/#home). I would recommend that all wallet
maintainers ensure that the keypaths only fit the described standards and do not
cater for uncommon territory. Further, I would limit the address index at
something lower than `2^31`, which is absurdly high. It might also be a
consideration to display the change keypath to the user. In my opinion, the
standards are to blame here. They do not impose common sense barriers and leave
enough room for interpretation to allow this unsafe behavior.

### Part II

Instead of ransoming the change of a sent transaction, an attacker could just
receive coins on public generated addresses derived at keypaths unknown to the
user in the first place. On the surface, Coldcard offers good protection against
this for advanced users by displaying the full keypath to the user when
verifying a receive address. For this the Coldcard receives a keypath string as
an argument, generates an address at the given indexes and displays it on its
screen. But let’s take a look at the MicroPython code that interprets the
keypath on the Coldcard. It takes the keypath string, uses Python's
`string.split()` on the slashes and then casts the indexes to integers. In
`shared/stash.py:201`:
``` python
    def derive_path(self, path, master=None, register=True):
        # Given a string path, derive the related subkey
        rv = (master or self.node).clone()

        if register:
            self.register(rv)

        for i in path.split('/'):
            if i == 'm': continue
            if not i: continue      # trailing or duplicated slashes

            if i[-1] == "'":
                assert len(i) >= 2, i
                here = int(i[:-1])
                assert 0 <= here < 0x80000000, here
                here |= 0x80000000
            else:
                here = int(i)
                assert 0 <= here < 0x80000000, here

            rv.derive(here)

        return rv
```

Looks harmless, right?

Initially I did not look at this part of the code, but at the suggestion of
Kaspar Etter, I passed in whitespace characters inside the keypath string. To
our surprise, the keypath on the Coldcard displayed with the whitespace inside!
The pitfall is that python's `int()` cast accepts the whitespace characters
`\t\n\r\v\f`, chiefly among them newline `'\n'`.

By inserting newlines, I was able to split the keypath in half. Since the length
of the string was unrestricted, I could add as many newlines as I wanted to;
basically as many until the device runs out of memory (~1250 newlines), thus
making it hard for the user to reach the end of the keypath display. This means
that an attacker could hide the ugly tail of a keypath, while making the user
think that he was receiving on a legitimate address. I filmed a demo of the
exploit, but instead of showing about 7 minutes of blank scrolling on the
Coldcard, I’d rather show a Coldcard powered dog simulator:

![Coldcard](/images/coldcard.gif "Coldcard")

### Public Disclosure

Lets get to the hard part: public relations after patching a disclosure. Once coldcard
published and announced their release on
[twitter](https://twitter.com/COLDCARDwallet/status/1190279744442568704), they
did not mention the disclosure in their announcement. At this point I should
have contacted them again and asked for clarfication. When a patch for a
disclosure is released, it is common practice to announce its impact and ensure
that users update as soon as possible. This is done across many vulnerability
response frameworks. In my opinion the vulnerabilities presented above are
serious enough to warant a public announcement and I and Shift Cryptosecurity
proceded to announce that the update contains fixes for the above disclosure.
This led to some unneeded back and forth on social media. I have since ammended
my relation to Coldcard and promised to withhold publishing the full disclosure
until now.


