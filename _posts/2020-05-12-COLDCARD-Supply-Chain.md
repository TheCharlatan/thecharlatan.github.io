---
layout: post
title: A practical supply chain attack on the COLDCARD
---

*TLDR: The COLDCARD does a factory reset when an existing PIN is
changed to an empty PIN , contrary to COLDCARD's claims that a factory reset is
impossible. This can be used to distribute tampered devices without much
effort. COLDCARD has not patched the issue to date.*

### Preface

The COLDCARD hardware wallet has a dual chip architecture, where some of the
entropy used to generate keys and the PIN used by the user to authenticate is
stored on a secure chip (an atecc608a), while the actual signing, transaction
logic and i/o is done on a general purpose MCU (STM32L496RG).

I recently read that the COLDCARD allows loading firmware that was not
officially signed by the wallet's developers. This is intended by them and
even one of their advertised features. On their website coldcardwallet.com they
state:

*We have so much internal protection for the master secret, that we feel it's
safe to allow potentially hostile firmware onto this platform. If you don't
feel safe doing that, then it's a choice you can make.*

So what protects their users from malicious firmware updates in general?
Firmware that is not signed by Coinkite will light up the red warning light the
first time it is run. It also displays a "Danger" screen for 30 seconds to make
sure users understand that something is wrong with the device. These scenarios
are also covered in their [extensive documentation](<https://github.com/Coldcard/firmware/blob/master/docs/pin-entry.md#obvious-hack-attack>):

*Obvious Hack-Attack*

*Idea: Find or steal a Coldcard. Load your trojan firmware onto it. Profit.*

- *but you don't know the main PIN (or else you'd have already stolen the funds)*
- *so changing the firmware is not easy since it does little without the main PIN: you will have to crack the welded case, and do some difficult soldering.*
- *regardless, once you change the firmware, red caution light will show*
- *since you can't set the genuine light without the PIN, and your trojan is signed with key zero, when the victim gets back, they will see the big "unauthorized firmware" warning, plus the red light and probably some scratches on the case, etc.*
- *weak solution: "helpfully" power up the Coldcard for them, and say... "Here it is ready for your pin, sir. No idea why the light is red today."*
- *so we need to warn users to power up the Coldcard themselves every time they enter the PIN.*

### Finding a loophole

But what if for example a reseller of the COLDCARD flashes his own firmware? In
this case the user would not see the warning screen, or the red warning light,
since it is only displayed on the first time the firmware is run. The main
protection against this is that once the wallet has been setup, it cannot be
reset again. This means that if the reseller would load his own firmware, the
user would notice, because he would receive an already setup device locked with
the reseller's PIN. As quoted from the their faq:

*Is there a factory reset?*

*There isn't a factory reset due to the secure element. Most of the fields in
that chip cannot be quickly reset. However, you can clear the wallet seeds and
remove secondary PIN code individually. It's a lot of typing and all
corresponding PIN codes must be already known to you.*

They also go on [to
say](https://github.com/Coldcard/firmware/blob/master/docs/pin-entry.md#limitation):

*if new device is intercepted from our factory (ie. without a main pin set),
new code cannot be loaded until the PIN is set, and there is no way to clear
main PIN.*

In the following I will show that both these claims are not true. There is
indeed a simple way to clear the main PIN in the firmware, which in turn does a
factory reset. This allows for a supply chain attack (or as they call it
interception from the factory).

### The exploit

I then checked in the COLDCARD [source
code](https://github.com/coldcard/firmware) if this claim actually holds up,
attempting to find a way with my own, changed firmware to put the device back
in what I would call "Blank - fresh off the factory" mode - meaning in all
practicality "factory resetting" it, or putting it back in the same state as
when the device arrived in the mail. This would allow an attacker to load
tampered firmware and distribute the tampered device in "mint" condition to his
victim. 

Its bootloader code is not that straight forward. It consists of multiple state
machines written in C, that produce state values that are read out by some
micropython code. While the C (or bulk) part of the bootloader is in
`stm32/bootloader`, the micropython (or user facing) code is in `shared`.

When booting up, the wallet first goes through a setup function that checks
the PIN monotonic counter and initializes communication with the secure chip.
It then checks if the main PIN is blank (filled with zeroes) in
`stm32/bootloader/pins.c`:

```
575 // need to know if we are blank/unused device
576 if(pin_is_blank(KEYNUM_main_pin)) {
577     args->state_flags |= PA_SUCCESSFUL | PA_IS_BLANK;
```

If it is blank, the `PA_IS_BLANK` state is set and is read out by the
micropython code. In the relevant case, this is in `shared/actions.py`:

```
681 elif pa.is_blank():
682     # let them play a little before picking a PIN first time
683     m = MenuSystem(VirginSystem, should_cont=lambda: pa.is_blank()
```

As we can see, if the PIN is indeed blank, the system is set into a "virgin",
or fresh off the factory state. The user is then displayed the menu to set the
PIN, or show the Bag Number. This means that if an attacker loads tampered
firmware and then lets that firmware change the PIN to zero, the device goes
back to this "blank" state. The PIN relevant PIN change code  is in shared/actions.py: 

```
1281    if pin is None:
1282        return await ux_aborted()
1283
1284    is_clear = (pin == '999999-999999')
```

In the tampered firmware I tried to set the PIN under this line to `pin = ''`.
Indeed, it did not check if the changed PIN provided is empty, and will
write it as the new PIN to the secure chip memory slot.

On next startup it will then the go through the normal setup
procedure - as if for a mint condition device. If we just change the PIN it
will however not show the accept terms screen that is usually shown on a fresh
device. The value keeping track of whether the terms have been accepted before
or not can be manipulated from our malicious firmware as well.  Meaning if we
add `settings.set('terms_ok', 0)` to the tampered firmware, the terms will
display again on next startup.

At this point a reseller could then distribute the COLDCARD to his customers,
without a trace that it has been tampered with. They later state in their
faq that: 

*All legitimate resellers should be providing the Coldcard unused and still in
it's original tamper-evident bag. As part of the first-use sequence, you will
verify the bag number matches the factory bag number.*

Since the device is the same, and the bag number is not changed in the device's
memory, the user will still be able to verify the device with the same bag
number. This means the attack also breaks a key feature of their supply
chain security. To summarize: 

- The attack has virtually no costs, scales easily and is quick to execute. 
- It reduces the supply chain security to only that of the plastic sleeve.
- There are no limits to what this tampered firmware could do other than the wallet's hardware constraints.
- Compromising a future user's funds would be as easy as setting the BIP32 root private key to that of the attacker.

### Mitigation

There don't seem to be any easy mitigations against this attack on current
COLDCARD devices, since it would require changes to the bootloader, which is
locked-down and therefore cannot be updated easily. A patch can only be
deployed to new devices shipped with a patched bootloader. As a future
mitigation for new devices, they could not allow its bootloader to run
arbitrary firmware. From what I gathered so far, few users load their own,
self-compiled firmware. Another potential mitigation might be to have a
different flag, or even a slot in the secure chip to track if the device has
been setup before. It should also check that the changed pin fulfills
some minimum requirements, especially being non-zero.

So how can a current user protect herself from this exploit? In theory, even
when the user attempts to upgrade the firmware with the genuine, developer
signed-off binary, the malicious firmware could play nice and update it by
putting it into a flash region that is not checked by the bootloader. The
bootloader only checks regions as defined in the header of the firmware (which
sits at the start of the firmware chunk in memory). Writing such a malicious
firmware would be involved though (I screwed up on first attempt and now it's a
brick), especially because of the 1 MB flash limit set in place by the
STM32L496RG mcu and the code execution protections on the chip. This means that
as a band aid solution all users should update their firmware first before
setting up the device. 

Normally I'm not interested in the physical security of hardware wallets, but
since they argued to me that enough physical security is provided as is, I
wanted to highlight that I strongly contest this claim. Not only is the factory
bag number left intact by the exploit, but the plastic sleeve itself is easily
compromised with minimal traces and household tools by cutting it open at the
bottom with a sharp knife and resealing it again with heat in a similar way, or
if laminating equipment is available in the same way, like the sides of the
sleeve.

### COLDCARD's response

After initially acknowledging the issue, no fix was provided for it and no
bounty was paid out. Since they have no intention of fixing the issue, I am now
publishing this report. I did ask for prior permission to post about the issue,
which they gave me as long as I mentioned their PIN code developer
documentation (also linked and quoted further above):
<https://github.com/Coldcard/firmware/blob/master/docs/pin-entry.md#how-to-develop-professional-code-on-coldcard>

