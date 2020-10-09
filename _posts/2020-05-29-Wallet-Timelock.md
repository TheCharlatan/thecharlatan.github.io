---
layout: post
title: Destroying your moneroj for fun and not for profit
image: /images/trezor_unlocktime.JPG
---

*TLDR: A lack of unlock time verification on the monero hardware wallets could
have allowed a compromised host to permanently lock up a user's XMR after a
transaction. Both Trezor and Ledger patched the issue.*

Every monero transaction has a field called `unlock_time`. It indicates how
long the outputs have to mature after transaction creation before they can be
spent again.  The outputs of a transaction with an `unlock_time` of 0 can be
spent immediately, or at least immediately after the normal 10 block maturity
period.  If the `unlock_time` is 100 blocks above the current block height, the
recipient needs to wait for 100 blocks until he can finally spend the outputs
again.

The maximum number of blocks that can be input is 500'000'000-1, which will
probably be mined in about 1900 years. If the number surpasses this limit, the
`unlock_time` gets interpreted as Unix time in seconds. Since this is a 64bit
field, it allows locking up coins for over a hundred billion years.

Hardware wallets should protect users against theft, ransom, and loss of their
coins by malware on their computers. For a monero wallet using either a Trezor
Model T or Ledger Nano S/X, the same expectation needs to hold.  Hardware
wallets must check all information that is passed to them from the host to
ensure that the user's coins are and remain safe.

The Ledger and Trezor hardware wallets did not check the `unlock_time` field
when signing a transaction. An adversary with malware on the user's host
machine could have permanently locked-up the user's coins with a very high
`unlock_time`. Since change amounts are not verified on the devices, the entire
balance of the user could be locked-up, even if the user only made a small
transaction. 

Both Trezor and Ledger patched the issue. Trezor could simply [patch their
firmware](https://github.com/trezor/trezor-firmware/commit/7944c1a837cb8b7c398be0ab6361d229ac591eab)
since they already parsed the `unlock_time` field as part of their transaction
logic. Both Trezor and Ledger display the raw number on their screen, so block
height and time based `unlock_times` display in the same manner.

![Some blocks](/images/trezor_unlocktime.JPG
"Trezor unlock time verification")

Ledger did not implement any logic for the `unlock_time` on their firmware, so
they not only had to [patch their
firmware](https://github.com/LedgerHQ/app-monero/commit/2e2692f31a0e904fb6f6c907a52842696f5352e0)
but also write a [patch for the monero
wallet](https://github.com/monero-project/monero/commit/688a3e87e712123d182ae6715610c461988f9e74).

Sadly, their initial patch introduced more problems. The parsed `unlock time`,
which can be any 64-bit number, was cast to a signed 32-bit integer causing an
integer overflow. Depending on the value, the device would either abort or
worse, show the wrong `unlock_time`. After some trouble finding the actual
culprit, this additional issue [was patched as
well](https://github.com/LedgerHQ/app-monero/pull/81).

![Forever](/images/ledger_unlocktime.JPG
"Ledger unlock time verification")

Both
[Trezor](https://blog.trezor.io/details-of-firmware-updates-for-trezor-one-version-1-9-0-and-trezor-model-t-version-2-3-0-46deb141fc09)
and [Ledger](https://donjon.ledger.com/lsb/009/) published security advisories
on the issue, gave me attribution and paid out a generous bug bounty.

