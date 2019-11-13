---
layout: post
title: A ransom attack on Coldcard's change and keypath verification
---

*Since this is the first time for me on the reporting end of a disclosure, I decided to write this blog post to document my experiences and help me with formulating future write-ups. Even though the exploits presented here are not particularly intricate, I do consider them serious.*

All items discussed here have been responsibly disclosed to [Coldcard](https://coldcardwallet.com/), made by [Coinkite](https://coinkite.com/), and have been fixed in their firmware version 3.0.2 release on the 1st of November. Coldcard has been cooperative with the disclosure and reacted quickly with all communications. Coinkite at the time of reporting had a [HackerOne account](https://hackerone.com/coinkite) whose bug bounty program we followed. However, it later turned out that this HackerOne account was unaffiliated and it was taken down shortly after. The bounty described there was not paid, and instead we received two Coldcard hardware wallets and a mug. I asked for Coldcard to attribute myself, Kaspar Etter and the company we work at, Shift Cryptosecurity, receiving attribution to myself in the Github release notes for one of the two vulnerabilities reported.

The Coldcard is a Bitcoin hardware wallet that has gained some popularity recently, especially through its early adoption of [Partially Signed Bitcoin Transactions (BIP174)](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki). They also advertise their air-gap feature, where information to the device is not passed through USB but by physically moving a microSD card between the device and host. However, the exploits presented below pass through the ‘air-gap’ but the funds can easily be recovered if the user keeps the Partially Signed Bitcoin Transaction (PSBT) on the microSD card (and no malware deletes the file).

### Preface

Hardware wallets not only need to protect against bad actors that want to steal your coins but also ensure that users' access to their coins is not unrecoverably lost. In principle, this means that whatever address is used to receive coins on the wallet should also allow you to spend those coins again. If an attacker can make a user receive coins on an address in the wallet that the user is not capable of spending from, this is a serious vulnerability. The attacker can then ransom the knowledge on how to spend the coins again.

I disclosed a similar ransom vulnerability to Coldcard as was already described in this [Siacoin
blog post](https://blog.sia.tech/a-ransom-attack-on-hardware-wallets-534c075b3a92). My colleague Kaspar Etter also discovered this attack independently on 7 February 2019 at Shift Cryptosecurity AG against the BitBox01. It was fixed with a release for the BitBox01 on [8 March 2019](https://medium.com/shiftcrypto/bitbox-desktop-app-4-5-0-with-firmware-6-0-2-release-fd77f8186a29). Later he also realised that a malicious desktop app can not just provide a non-standard keypath for receive addresses but also for change addresses. Shift released another fix thereafter for the BitBox01 on [28 March 2019](https://medium.com/shiftcrypto/bitbox-desktop-app-4-6-0-with-firmware-6-0-3-release-ec46937afe7c). The BitBox02 was still in development and thus not affected.

Hierarchical deterministic wallets (BIP32; BIP44) have become a de facto standard for cryptocurrency wallets because they allow a more convenient way to backup a wallet. However, they also allow an infinite number of BIP32 derivation paths (a.k.a keypaths) to derive wallet addresses from which to receive coins. The Coldcard firmware allowed generating addresses on arbitrary keypaths only limited by its available memory. While for the receive address, the full keypath was displayed (it still ended up being exploitable, as described in part II), the change keypath is neither shown nor restricted when signing a transaction. BIP32 key derivation is done with what is referred to as “extended” keys. These are keys that additionally contain the BIP32 chain code to allow derivation of child keys. Extended public keys are commonly referred to as “xpubs” and are usually derived and shared at the hardened account level, for example `m/44’/0’/0’`.

### Part I

During the Lightning Conference in Berlin, I had a conversation with HWI and Bitcoin Core developer Sjors Provoost, in which he made a passing remark that something may be weird with Coldcard’s BIP32 keypath verification. When I came home again after the conference, I decided to take a look at Coldcard’s firmware code and, surely enough, did not find a lot in relation to sanitizing and restricting BIP32 derivation paths.

To mark the change in a transaction for the Coldcard, the change address’ keypath, public key and fingerprint are additionally passed to the Coldcard. While the fingerprint (a shortened hash of the xpub and script) should ensure that the input and change output at least need a common keypath prefix, I did not verify this. Since I used wallet software that allowed transaction construction for a common xpub between input and output, I used the same xpub for both. As an example in this post, let’s take the xpub at the keypath prefix `m/44'/0'/0'`. To understand why using this common xpub between the input and change output keypaths is not good enough, we need to take a look at
[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) and
[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki).
These are the two standards used by wallets and hardware wallets alike
to generate and manage public and private keys from a single seed. While they do give good detail on hardening, derivation levels and their responsibilities, they give no indication on the
restriction of the total keypath space.

In practice, this means that for each keypath level we can generate 2^31 = 2'147'483'648 (4 byte numbers minus 1 bit for encoding) non-hardened and the same number of hardened keys. A keypath like `m/44'/0'/0'/0/2147483646` is perfectly valid. If the key at that index is used to generate an address and receive coins, the wallet would have to first scan all 2'147'483'647 addresses until it finds the funds.

It gets worse, since we can also go as deep in the keypath tree until the data structure that holds it goes out of memory to keypaths like:

`m/44'/0'/0'/0/2147483646/2147483646/2147483646/.../2147483646`

At this rate we quickly surpass the overall possible keyspace of Bitcoin keys, making it computationally infeasible to find the key to spend the coins again. An attacker can use this to create a transaction with a change keypath that is virtually unspendable without the
complete knowledge of the keypath. Hardware wallets should therefore limit the reachable indexes in the keypath.

To prepare these transactions and communicate with the Coldcard, I modified the [Electrum
wallet](https://electrum.org/#home). I would recommend that all wallet maintainers ensure that the keypaths only fit the described standards and do not cater for uncommon territory. Further, I would limit the address index at something fairly lower than 2^31. For comparison the total number of bitcoin addresses ever used is currently around 2^29. It might also be a consideration to display the change keypath to the user. In my opinion, the standards could be improved. They do not explicitly impose common sense barriers and leave enough room for interpretation to allow this unsafe behavior.

### Part II

Instead of ransoming the change of a sent transaction, an attacker could make the user  receive coins on addresses derived at keypaths unknown to the user in the first place. On the surface, Coldcard offers good protection against this for advanced users by displaying the full keypath to the user when verifying a receive address. The Coldcard receives a keypath string as an argument, generates an address at the given indexes, and displays it on the screen. But let’s take a look at the MicroPython code that parses the keypath on the Coldcard. It takes the keypath string, uses Python's `string.split()` on the slashes and then casts the indexes to integers. In [`shared/stash.py:201`](https://github.com/Coldcard/firmware/blob/d2a44fe4e06b32e779468ea45c25681057f45a33/shared/stash.py#L201):
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
Initially I did not look at this part of the code, but at the suggestion of Kaspar Etter, I passed in whitespace characters inside the keypath string. To our surprise, the keypath on the
Coldcard displayed with the whitespace inside! The pitfall is that python’s `int()` cast
accepts the whitespace characters `\t\n\r\v\f`, chiefly among them newline `\n`. If the int() cast is successful, the unchanged string is rendered on the Coldcard’s display. Additionally it also allows passing strings with as many `m/` at the beginning of the keypath as permitted by the maximum message length of the Coldcard communication protocol.

By inserting newlines, I was able to modify the displayed keypath, such that what the user saw could differ from what was used by the wallet code. Since the length of the string was unrestricted, I could add as many newlines as I wanted; basically as many until the device runs out of memory (~1250 newlines, depending on the memory available), thus making it hard for the user to reach the end of the keypath display. In one instance, scrolling to the bottom of the screen to see the rest of the keypath information took me 7 minutes. This means that an attacker could hide the ugly tail of a keypath, while making the user think that she was receiving on a legitimate address. I filmed a demo of the exploit, where I show some empty space and a 5x16px scene with a dog. The less common whitespace characters like `\f` are rendered as `x`, allowing me to draw on the screen.

![Coldcard](/images/coldcard.gif "Coldcard")

### Public disclosure

Let’s get to the hard part: public relations after patching a vulnerability. Once Coldcard
published and announced their release on
[Twitter](https://twitter.com/COLDCARDwallet/status/1190279744442568704), they
did not indicate that this is a security upgrade. At this point I should have contacted them again and asked for clarification, which I regret not doing. When a patch for a disclosure is released, it is common practice to announce its impact and ensure that users update as soon as possible. This is done across many vulnerability response frameworks. In my opinion, the vulnerabilities presented above are serious enough to warrant a public announcement, especially since the [public code commits](https://github.com/Coldcard/firmware/commit/fe5cced65e66f42b4a24d59d1c3a96a010ed4158#diff-6330798e4a428addab89059586a262b1R1120) on GitHub give sufficient hints of the problem. After seeing that the vulnerabilities became public through such commits, Shift Cryptosecurity and I proceeded to [announce](https://twitter.com/ShiftCryptoHQ/status/1190302736073547776) that the update contains fixes for the above disclosure and that users should update as soon as possible. This led to some unneeded back and forth on social media about the scope and impact of the vulnerability. I have since mended my relation to Coldcard and am looking forward to further collaboration. They also released a [blog post](https://blog.coinkite.com/troublesome-change/) describing part 1 of the issue with an overview of their fixes and mitigations.

