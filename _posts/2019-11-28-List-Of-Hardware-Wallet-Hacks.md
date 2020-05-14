---
layout: post
title: List of Hardware Wallet Hacks
---

_This is a dynamic document and changes as my understanding of these vulnerabilities changes and as new vulnerabilities get discovered_

What constitutes a hardware wallet hack?<br>
I count anything as a "hack" that allows a hacker to change a hardware wallet's intended behavior. This means it is not relevant to me if the hack was ever exploited, or if it has received a low likelihood rating from vendors.

Know of a hack that is not included?<br>
Let me know here: <https://github.com/TheCharlatan/thecharlatan.github.io><br>

# 2014
### Juli:

:office: Vendor: Trezor<br>
:scroll: Title: Malicious ScriptSig in transaction<br>
:nerd_face: Detail: A specially crafted transaction could extract the private key<br>
:eyes: Type: Transaction validation attack with authentication<br>
:poop: Bug: Buffer Overflow<br>
:sunglasses: Reporter: Nicolas Bacca (Ledger)<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/524f2a957afb66e6a869384aceaca1cb7f9cba60><br>

# 2015
### February:

:office: Vendor: Trezor<br>
:scroll: Title: SpendMultisig malicious change in transaction<br>
:nerd_face: Detail: A specially crafted transaction could contain a change output of an attacker, which wasn't confirmed by the user<br>
:eyes: Type: Transaction validation attack with authentication<br> 
:poop: Bug: Insufficient transaction checks<br>
:sunglasses: Reporter: Nicolas Bacca (Ledger)<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/137a60ce017c402ac160258bcc4b5f7b5aba0560><br>

### March:

:office: Vendor: Trezor<br>
:scroll: Title: Possible key extraction with oscilloscope<br>
:nerd_face: Detail: With physical access to the device and an oscilloscope, the private key could have been extracted from the device<br>
:eyes: Type: Signal noise / power analysis side channel<br>
:poop: Bug: Insufficient PIN protection for derivation of keys, minimize the usage of nested loops to increase const'ness of execution time<br>
:sunglasses: Reporter: Jochen Hoenicke<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/7c6d2fe395c8475efbc93257892f0efac3d1511c><br> 
:dart: Explanation from reporter: <https://jochen-hoenicke.de/crypto/trezor-power-analysis/><br>

# 2017
### August:

:office: Vendor: Trezor<br>
:scroll: Title: SRAM memory access<br>
:nerd_face: Detail: The SRAM was not cleared on soft reset, allowing extraction using special firmware and direct access to the device board<br>
:eyes: Type: Platform reset attack<br>
:poop: Bug: (see detail)<br>
:sunglasses: Reporter: Sunny<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/98e617d8740b85ae01d7d6e0dd3f49e66057a210><br>
:mega: Explanation from vendor: <https://blog.trezor.io/fixing-physical-memory-access-issue-in-trezor-2b9b46bb4522><br>
:dart: Explanation from reporter: <https://saleemrashid.com/2017/08/17/extracting-trezor-secrets-sram/><br>

# 2018 
### February:

:office: Vendor: Trezor<br>
:scroll: Title: STM32F205 chip issue<br>
:nerd_face: Detail: The bootloader memory write-protection is not working as intended in the STM32F205, which is used in the Trezor One. The issue was solved by activating the Memory Protection Unit, keeping the bootloader safe from unauthorized write-access.<br>
:eyes: Type: Supply chain attack<br>
:poop: Bug: Bad chip configuration<br>
:sunglasses: Reporter: Saleem Rashid<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/9588e8f2736b60916f51e470deb18f55112a6ebc><br>
:mega: Explanation from vendor: <https://blog.trezor.io/trezor-one-firmware-update-1-6-1-eecd0534ab95><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br>
:scroll: Title: Bad BIP32 implementation<br>
:nerd_face: Detail: Accessing the 'xpub' API command for the master key path of both the hidden and the standard wallet allowed for the reconstructing of the private keys of the standard and hidden wallet.<br> 
:eyes: Type: API remote attack<br> 
:poop: Bug: Bad cryptography for the wallet vs hidden wallet derivation<br> 
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://shiftcrypto.ch/bitbox01/disclosure><br> 
:dart: Explanation from reporter: <https://saleemrashid.com/2018/11/26/breaking-into-bitbox><br> 

### March:

:office: Vendor: Ledger<br>
:scroll: Title: Padding oracle attack on SCP<br>
:nerd_face: Detail: A padding oracle attack was found on the Secure Channel established between the device and Ledger’s HSM. It allows an attacker to decrypt the firmware updates.<br> 
:poop: Bug: Bad padding of messages between MCU and SC<br>
:sunglasses: Reporter: Timothee Isnard<br>
:mega: Explanation from vendor: <https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/><br>

:office: Vendor: Ledger<br>
:scroll: Title: MCU signature verification bypass<br> 
:nerd_face: Detail: The signature verification of the MCU can be bypassed, allowing an attacker to perform supply chain attacks. It requires physical access to the device before the generation of the seed.<br>
:eyes: Type:<br>Supply chain attack<br>
:poop: Bug: Overall authentication architecture in MCU, fixed with a bunch of small patches<br>
:sunglasses: Reporter: Saleem Rashid<br>
:mega: Explanation from vendor: <https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/><br>
:dart: Explanation from reporter: <https://saleemrashid.com/2018/03/20/breaking-ledger-security-model/><br>

:office: Vendor: Ledger<br>
:scroll: Title: Isolation vulnerability<br>
:nerd_face: Detail: A malicious app can break the isolation between apps and access sensitive data managed by specific apps such as GPG, U2F or Neo.<br> 
:poop: Bug: Null pointer dereferencing, pointer length not properly checked, Flash zone not wiped properly after device reset.<br>
:eyes: Type: Privilege escalation<br>
:sunglasses: Reporter: Sergei Volokitin<br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/003/><br>
:dart: Explanation from reporter: <https://i.blackhat.com/us-18/Wed-August-8/us-18-Volokitin-Software-Attacks-On-Hardware-Wallets.pdf><br>

### May

:office: Vendor: Trezor<br>
:scroll: Title: Race condition in recovery<br>
:nerd_face: Detail: Specially crafted USB communication packets could trigger a stack overflow in recovery which could lead to code execution.<br>
:eyes: Type: Stack overflow<br>
:poop: Bug: USB buffer overflow, during dry-run recovery which recursively handles packets, a stack overflow can be triggered<br>
:sunglasses: Reporter: Christian Reitter<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-6-2-a3b25b668e98><br> 
:dart: Explanation from reporter: <https://blog.inhq.net/posts/trezor-one-dry-run-recovery-stack-overflow/><br>

:office: Vendor: Trezor<br>
:scroll: Title: Message processing error<br>
:nerd_face: Detail: Specially crafted USB packet could trigger a buffer overflow which could lead to code execution on older firmware.<br>
:eyes: Type: Buffer overflow<br>
:poop: Bug: USB buffer overflow if the USB message buffer is flooded with specially crafted incoming messages<br>
:sunglasses: Reporter: Christian Reitter<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d><br>
:mega: Explanation from vendor: <https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d><br>

### July

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Simulating the secure chip<br>
:nerd_face: Detail: After physically breaking apart the BitBox casing, attaching invasive probes, and manipulating the data sent to the BitBox's micro controller, a BitBox could be reset but without erasing the wallet secrets. A patch was provided on 31 July 2018.<br> 
:eyes: Type: Information leak<br>
:poop: Bug: Secrets not cleared after wallet reset<br>
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://shiftcrypto.ch/bitbox01/disclosure/><br> 
:dart: Explanation from reporter: <https://saleemrashid.com/2018/11/26/breaking-into-bitbox><br> 

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox.<br> 
:nerd_face: Detail: When initially pairing a BitBox to the mobile verification app, a man-in-the-middle (MITM) on a compromised computer could insert themselves and then later change the information to be displayed on the mobile app. We provided a patch on 31 July 2018 in firmware v4.0.0. The vulnerability existed only during the initial pairing and if your computer was compromised by an attacker aware of the issue.<br> 
:eyes: Type: Information leak<br>
:poop: Bug: Bad Crypto<br>
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://shiftcrypto.ch/bitbox01/disclosure/><br> 
:dart: Explanation from reporter: <https://saleemrashid.com/2018/11/26/breaking-into-bitbox><br> 


### August

:office: Vendor: Trezor<br>
:scroll: Title: MPU circumvention via SYSCFG registers<br>
:nerd_face: Detail: Security fix deployed via the 1.6.1 firmware update could be circumvented via clever use of the SYSCFG registers. This was fixed by completely disabling the SYSCFG registers via the MPU.<br>
:eyes: Type: Supply chain attack<br>
:poop: Bug: MPU rule could be circumnavigated<br>
:sunglasses: Reporter: Sunny<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/fdd5cbe20271634dc9ba4424ae40f1d11332cdf2><br>
:mega: Explanation from vendor: <https://blog.trezor.io/trezor-one-firmware-update-1-6-3-73894c0506d><br>

### September

:office: Vendor: Trezor<br>
:scroll: Title: Buffer overflow in bech32_decode<br>
:nerd_face: Detail: The C reference implementation for bech32 has an unsigned integer overflow that can lead to a buffer overflow. The bug was fixed by preventing out-of-bounds accesses in the code.<br>
:eyes: Type: Buffer overflow<br>
:poop: Bug: No sufficient out of bounds check<br>
:sunglasses: Reporter: Christian Reitter<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/5c6b47288323a6cafe331304d2708a3c2a45f4b0><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8><br>

### October

:office: Vendor: Trezor<br>
:scroll: Title: Buffer overflow in cash_decode<br>
:nerd_face: Detail: The cash_decode function in the trezor-crypto library allowed an out-of-bounds write. The bug was fixed by preventing the out-of-bounds accesses in the code.<br>
:eyes: Type: Buffer/stack overflow<br>
:poop: Bug: No sufficient out of bounds check<br>
:sunglasses: Reporter: Gabrial Campana<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/2bbbc3e15573294c6dd0273d2a8542ba42507eb0><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8><br>

:office: Vendor: Trezor<br>
:scroll: Title: Side-channel analysis (SCA) of PIN comparison<br>
:nerd_face: Detail: Using a SCA bench an attacker could create the database of power consumption and electromagnetic traces of a device. This database could later be used to unlock a locked device using the same SCA bench. The issue was fixed by rewriting the device storage to not compare PINs directly, but rather compare random data stretched by the PIN.<br>
:eyes: Type: Information leak<br>
:poop: Bug: Naive implementation of PIN storage<br>
:sunglasses: Reporter: Charles Guillemet<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/4f32cb508383ec0e65843d037f6ac6473a668359><br>

### November

:office: Vendor: Trezor<br>
:scroll: Title: Information leak via U2F<br>
:nerd_face: Detail: The C/C++ reference implementation for U2F by Yubico contains broken definition of a struct which can leak bytes from RAM via USB. The bug was fixed by updating the structure definition to a new correct one.<br>
:eyes: Type: Information leak<br>
:poop: Bug: Bad struct memory layout<br>
:sunglasses: Reporter: Christian Reitter<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/0b26c529ec49daf584f322f3ef959c79694c8cf5><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-2-3c97adbf121e><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/u2fhid_init_resp-information-leak/><br>

:office: Vendor: Ledger<br> 
:scroll: Title: Bitcoin change address injection<br>
:nerd_face: Detail: A vulnerability was found in the Bitcoin app allowing an attacker to add an unverified output change address into a legit transaction. It can lead to sending funds to an arbitrary address without requiring an additional confirmation on the device. The original transaction still has to be confirmed though.<br>
:poop: Bug: Bad Bitcoin transaction information validation<br>
:sunglasses: Reporter: Sergey Lappo<br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/004/><br>
:dart: Explanation from reporter: <https://sergeylappo.github.io/ledger-hack/><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Poking around the secure chip<br>
:nerd_face: Detail: Bad configuration of the secure chip leaves it redundant in the BitBox01 hardware design. This is not patchable.<br>
:eyes: Type: Break of existing security model, lead to re-assesment of public security claims<br>
:poop: Bug: Bad secure chip configuration.<br>
:sunglasses: Reporter: Saleem Rashid<br> 
:dart: Explanation from reporter: <https://saleemrashid.com/2018/11/26/breaking-into-bitbox/><br>

### December

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox<br>
:nerd_face: Detail: Encrypted USB communication, when not authenticated, can be modified by a man-in-the-middle (MITM) attacker in undesirable ways. A patch was provided on 4 December 2018 in firmware v5.0.0.<br> 
:eyes: Type: Break on-hardware verification<br> 
:poop: Bug: Usage of a plain AES-256-CBC cipher for authentication. Never use encryption for authentication.<br> 
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://shiftcrypto.ch/bitbox01/disclosure/><br>


:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox<br>
:nerd_face: Detail: Encrypted USB communication, when not authenticated, can be modified by a man-in-the-middle (MITM) attacker in undesirable ways. A patch was provided on 4 December 2018 in firmware v5.0.0.<br>
:eyes: Type: Break on-hardware verification<br> 
:poop: Bug: Usage of a plain AES-256-CBC cipher for authentication. Never use encryption for authentication.<br> 
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://shiftcrypto.ch/bitbox01/disclosure/><br>

:office: Vendor: Trezor<br>
:scroll: Title: SRAM Dump during the firmware update<br>
:nerd_face: Detail: Using a special glitching hardware an attacker could trick the device processor into Read Protection level 1 which allows readout of RAM. The issue was fixed by not storing sensitive data in RAM during the firmware update.<br>
:eyes: Type: Information leak<br>
:poop: Bug: Sensitive values in RAM during firmware update<br>
:sunglasses: Reporter: wallet.fail<br> 
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/07231d936e41335b3ec44c4c6eb336be006890d0><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012><br>
:dart: Explanation from reporter: <https://media.ccc.de/v/35c3-9563-wallet_fail><br>

:office: Vendor: Ledger<br>
:scroll: Title: MCU Bootloader verification bypass.<br>
:nerd_face: Detail: The signature verification of the Ledger Nano S MCU can be bypassed, allowing an attacker to install an arbitrary firmware on the MCU.<br>
:poop: Bug: f00dbabe<br>
:sunglasses: Reporter: wallet.fail<br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/005/><br>
:dart: Explanation from reporter: <https://media.ccc.de/v/35c3-9563-wallet_fail><br>

# 2019
### January

:office: Vendor: Trezor<br>
:scroll: Title: Secret information leak via USB Descriptors<br>
:nerd_face: Detail: The attack used specialized hardware to inject a fault into the comparison function in the USB stack. When timed properly, an attacker could trick USB stack into returning sensitive data via USB in the USB descriptor.<br> 
:eyes: Type: Information leak<br>
:poop: Bug: Outgoing packets too big, MPU did not protect sectors around the actual storage sectors, which would have halted execution<br>
:sunglasses: Reporter: Colin O'Flynn<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/22f37e81a3270da5e8e5d6c55abc8f15f3a35567><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012><br>

:office: Vendor: Coldcard<br>
:scroll: Title: Attack on Coldcard short PINs<br>
:nerd_face: Detail:<br>The attack is achieved by connecting a man-in-the-middle (MITM) to the bus the CCW uses to communicate with its secure element (SE). Then commands on the bus are modified to cause the MCU to not count failed PIN entry attempts.<br>This gives the attacker an unlimited number of attempts to guess the PIN.<br> 
:eyes: Type: Bypass of authentication<br>
:poop: Bug: Bad secure chip answer verification<br>
:sunglasses: Reporter: Lazy Ninja<br>
:mega: Explanation from vendor: <https://blog.coinkite.com/use-long-pins/><br>
:dart: Explanation from reporter: <https://www.cryptolazyninja.com/2019/03/coldcard-wallet-short-pin-brute-force.html><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Information leak via U2F<br>
:nerd_face: Detail: The C/C++ reference implementation for U2F by Yubico contains a broken definition of a struct which can leak bytes from RAM via USB. The bug was fixed by updating the struct definition to a new correct one.<br>
:poop: Bug: Bad struct memory layout<br>
:sunglasses: Reporter: Christian Reitter<br>
:mega: Explanation from vendor: <https://medium.com/shiftcrypto/important-security-news-about-version-4-4-0-upgrade-2449b745be9><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/u2fhid_init_resp-information-leak/><br>


### March

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: BIP32 address derivation ransom attack<br>
:nerd_face: Detail: No restrictions on possible BIP32 key paths led to a ransom attack<br>
:poop: Bug: Bad interpretation of BIP32 and BIP44 standard<br>
:mega: Explanation from vendor: <https://medium.com/shiftcrypto/bitbox-desktop-app-4-6-0-with-firmware-6-0-3-release-ec46937afe7c> <https://medium.com/shiftcrypto/bitbox-desktop-app-4-5-0-with-firmware-6-0-2-release-fd77f8186a29><br>

:office: Vendor: Satoshi Labs<br> 
:iphone: Product: Trezor 1<br>
:scroll: Title: Breaking Trezor One with Sice Channel Attacks<br>
:nerd_face: Detail: A Side Channel Attack on PIN verification allows an attacker with a stolen Trezor One to retrieve the correct value of the PIN within a few minutes.<br>
:eyes: Type: Information leak<br>
:poop: Bug: PIN validity was checked in constant time, but in sequence. The validity check thus exposed a unique side-channel signature during verification.<br>
:sunglasses: Reporter: Ledger Donjon<br>
:mega: Explanation from vendor: <https://blog.trezor.io/our-response-to-ledgers-mitbitcoinexpo-findings-194f1b0a97d4><br>
:dart: Explanation from reporter: <https://donjon.ledger.com/Breaking-Trezor-One-with-SCA/><br>

### April

:office: Vendor: Trezor<br>
:scroll: Title: Information leak via OLED display<br>
:nerd_face: Detail: The attack uses power analysis to read the information shown on the OLED display.<br>
:eyes: Type: Information leak<br>
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant<br>
:sunglasses: Reporter: Christian Reitter<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/f16c941ed4ac3c2e2c401de931249d0b2f34c29b><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-of-the-oled-vulnerability-and-its-mitigation-d331c4e2001a><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/oled-side-channel-status-summary/><br>

:office: Vendor: Ledger<br>
:scroll: Title: OLED screen side-channel vulnerability.<br>
:nerd_face: Detail: A side-channel leakage on the row-based OLED display was found. The power consumption of each row-based display cycle depends on the number of illuminated pixels, allowing a partial recovery of display contents. For example, a hardware implant in the USB cable might be able to leverage this behavior to recover confidential secrets such as the PIN and BIP39 mnemonic. In other words, the side-channel is relevant only if the attacker has enough control over the device’s USB connection to make power-consumption measurements and advanced statistical analysis while the secret data is displayed. The side-channel is not relevant in other circumstances, such as a stolen device that is not currently displaying secret data.<br>
:eyes: Type: Information leak<br>
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant<br>
:sunglasses: Reporter: Christian Reitter<br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/006/><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/oled-side-channel-status-summary/><br>

:office: Vendor: Coldcard<br>
:scroll: Title: Possible Display Information Leak<br>
:nerd_face: Detail: The attack uses power analysis to read the information shown on the OLED display.<br>
:eyes: Type: Information leak<br>
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant<br>
:sunglasses: Reporter: Christian Reitter<br>
:mega: Explanation from vendor: <https://blog.coinkite.com/noise-troll/><br> 
:dart: Explanation from reporter: <https://blog.inhq.net/posts/oled-side-channel-status-summary/><br>

### May

:office: Vendor: BC Vault<br>
:iphone: Product: BC Vault One<br>
:scroll: Title: BC Vault One button side channel<br>
:nerd_face: Detail: The attack uses H-field probing and a USB resistor shunt to detect button presses, like those made during initial PIN entry. While the report was received by the vendor, no mitigation was attempted and communication was aborted with the reporter.<br>
:poop: Bug: H-field Side Channel<br>
:sunglasses: Reporter: Christian Reitter<br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/bc-vault-one-button-side-channel/><br>

### June

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Blinking pattern mismatch<br>
:nerd_face: Detail: The blinking patterns of the BitBox01 reveal important information on the behvaiour of the device<br>
:poop: Bug: Bad differentiation between modes for the user<br>
:sunglasses: Reporter: Saleem Rashid<br> 
:mega: Explanation from vendor: <https://medium.com/shiftcrypto/bitbox-desktop-app-4-9-0-with-bitbox01-firmware-6-1-1-release-1b84c5f9295f><br>

### Juli

:office: Vendor: Shapeshift, Satoshi Labs<br>
:iphone: Product: Keepkey, Trezor One, Trezor T<br>
:scroll: Title: Unfixable Seed Extraction on Trezor - A practical and reliable attack<br>
:nerd_face: Detail: An attacker with a stolen device can extract the seed from the device. It takes less than 5 minutes and the necessary materials cost around 100$. This vulnerability affects Trezor One, Trezor T, Keepkey and all other Trezor clones. Unfortunately, this vulnerability cannot be patched and, for this reason, we decided not to give technical details about the attack to mitigate a possible exploitation in the field. However SatoshiLabs and Keepkey suggested users to either exclude physical attacks from their threat model, or to use a passphrase.<br>
:eyes: Type: Hardware Exploit<br>
:poop: Bug: Not clear, but seems to be a fundamental bug in the STM32F205 chip. The bug cannot be fixed and the vendors seemed to have changed their threatmodel now to not include localized hardware attacks. Hardware security is only guaranteed with the employment of an additional seed phrase.<br>
:sunglasses: Reporter: Ledger Donjon<br>
:mega: Explanation from vendor: No official explanation from Trezor; explanation from Keepkey: <https://medium.com/shapeshift-stories/responding-to-ledgers-2019-breakingbitcoin-findings-4213849a4fb><br> 
:dart: Explanation from reporter: <https://donjon.ledger.com/Unfixable-Key-Extraction-Attack-on-Trezor/><br>

### August

:office: Vendor: Shapeshift<br>
:iphone: Product: Keepkey<br>
:scroll: Title: OLED screen side-channel vulnerability.<br>
:nerd_face: Detail: Same as with Trezor, Ledger, Coldcard and BitBox02<br> 
:eyes: Type: Information leak<br>
:poop: Bug: OLED screens consume power based on number of pixels that are on. Keepkey alleges that since they show multiple seed words at once, the vulnerability does not apply to them.<br>
:sunglasses: Reporter: Christian Reitter<br>
:mega: Explanation from vendor: <https://medium.com/shapeshift-stories/shapeshift-security-update-5b0dd45c93db><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/oled-side-channel-status-summary/><br>

### October

:office: Vendor: Trezor<br>
:scroll: Title: Malicious change in a mixed transaction<br>
:nerd_face: Detail: An attacker could create a specially crafted multisig transaction which would hide the multisig change address.<br>
:eyes: Type: Missing Check<br>
:poop: Bug: Input and output Bitcoin transaction fingerprints were not sufficiently checked.<br>
:sunglasses: Reporter: Marko Bencun<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/8eb6ce08995514c67d175b7197feeadeccc48ff0><br>
:mega: Explanation from vendor: <https://blog.trezor.io/details-of-the-multisig-change-address-issue-and-its-mitigation-6370ad73ed2a><br>
:dart: Explanation from reporter: <https://medium.com/shiftcrypto/a-remote-theft-attack-on-trezor-model-t-44127cd7fb5a><br>

:office: Vendor: Ledger<br>
:scroll: Title: Monero private key retrieval.<br>
:nerd_face: Detail: The Monero App for Ledger Nano was found to be vulnerable to a private key retrieval through the use of a malicious Monero Client (desktop application). Some computational elements are encrypted by the Nano S with a key only known to the Monero application, and sent to the desktop client for later use, due to space limitations on the Nano. During the final step of the signature (MLSAG sign), the client sends back some sensitive encrypted elements which the app uses to compute a Schnorr signature. A malicious client can misuse this by replaying earlier elements of this computation, and induce a variant of a nonce-reuse attack (see for example the PS3 Fail). This replay of commands is possible because the key derived by the app to encrypt elements is static, and there is no message authentication.<br>
:poop: Bug: Bad MLSAG signature implementation<br> 
:clipboard: Patch: <https://github.com/LedgerHQ/ledger-app-monero/commit/5d0658ad6369f3d0ff2d10ee9effa410eb185b98><br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/007/><br>

:office: Vendor: Coldcard<br>
:scroll: Title: Troublesome Change Outputs<br> 
:nerd_face: Detail: It is possible to make a valid PSBT file that sends the change left from a transaction to a unknown location. If an attacker had your XPUB, and could change your PSBT file before you sign, they could modify the file so that the “change” (ie. the balance of Bitcoins you are sending back to yourself) goes to an effectively unknown address. If the attacker is profit motivated, they can ransom the knowledge of those change UTXO back to you.<br> 
:poop: Bug: BIP32 address derivation ransom attack<br>
:sunglasses: Reporter: TheCharlatan<br> 
:mega: Explanation from vendor: <https://blog.coinkite.com/troublesome-change/><br> 
:dart: Explanation from reporter: <https://thecharlatan.github.io/Ransom-Coldcard/><br>

:office: Vendor: Coldcard<br>
:scroll: Title: Ransom attack on Coldcard's receive address verification<br>
:nerd_face: Detail: By inserting newlines in the derivation path string sent to the Coldcard, the displayed characters could be split. This could trick users into verifying an address for a BIP32 derivation path that is not easily accessible.<br>
:sunglasses: Reporter: TheCharlatan<br>
:poop: Bug: Bad input validation from host<br> 
:dart: Explanation from reporter: <https://thecharlatan.github.io/Ransom-Coldcard/><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Mobile pairing information leak BitBox01<br> 
:nerd_face: Detail: ?<br>
:poop: Bug: Bad cryptography<br>
:sunglasses: Reporter: Saleem Rashid<br>
:mega: Explanation from vendor: <https://medium.com/shiftcrypto/bitboxapp-4-14-0-5e72575b0819><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox01<br> 
:scroll: Title: Base64 Parser Buffer Overflow<br> 
:nerd_face: Detail: The BitBox01 uses the NibbleAndAHalf library for base64 encoding. Among a bunch of potential issues, it contains a critical buffer overflow bug that would allow writing to adjacent heap memory. The NibbleAndAHalf library is not maintained for security bugs and should not be used by embedded projects where security is important. Since this bug could not be shown to critically change the program flow of the firmware, it received a low severity rating by the vendor (but was patched with the unmaintained NibbleAndAHalf library remaining in place).<br>
:poop: Bug: Buffer Overflow, Bad choice of dependency<br>
:sunglasses: Reporter: Christian Reitter<br> 
:dart: Explanation from reporter: <https://blog.inhq.net/posts/base64-parser-issues/><br> 

### December

:office: Vendor: Shapeshift<br>
:iphone: Product: Keepkey<br>
:scroll: Title: STM32 glitch attack<br>
:nerd_face: Detail: Same attack as executed by wallet.fail team on the Trezor, but now reproduced on Keepkey.<br>
:poop: Bug: STM32F205 hardware weakness<br>
:sunglasses: Reporter: Kraken<br>
:dart: Explanation from reporter: <https://blog.kraken.com/post/3248/flaw-found-in-keepkey-crypto-hardware-wallet-part-2/><br>

:office: Vendor: Shapeshift<br>
:iphone: Product: Keepkey<br>
:scroll: Title: USB Packet Handling Bug<br>
:nerd_face: Detail: Insufficient checks in the USB packet handling of the ShapeShift KeepKey hardware wallet before firmware 6.2.2 allow out-of-bounds writes on the stack via crafted messages. The vulnerability could allow code execution or other forms of impact. It can be triggered by unauthenticated attackers and the interface is reachable via WebUSB.<br>
:poop: Bug: USB buffer overflow<br>
:sunglasses: Reporter: Christian Reitter<br>
:mega: Explanation from vendor: <https://medium.com/shapeshift-stories/shapeshift-security-update-8ec89bb1b4e3><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/keepkey-CVE-2019-18671/><br>

:office: Vendor: Shapeshift<br>
:iphone: Product: Keepkey<br>
:scroll: Title: Mnemonic Wipe Bug<br>
:nerd_face: Detail: Insufficient checks in the finite state machine of the ShapeShift KeepKey hardware wallet before firmware 6.2.2 allow a partial reset of cryptographic secrets to known values via crafted messages. Notably, this breaks the security of U2F for new server registrations and invalidates existing registrations. This vulnerability can be exploited by unauthenticated attackers and the interface is reachable via WebUSB.<br> 
:poop: Bug: Secrets not wiped fully, unclear at this time how this was achieved.<br>
:sunglasses: Reporter: Christian Reitter<br> 
:mega: Explanation from vendor: <https://medium.com/shapeshift-stories/shapeshift-security-update-8ec89bb1b4e3><br>
:dart: Explanation from reporter: <https://blog.inhq.net/posts/keepkey-CVE-2019-18672/><br>

:office: Vendor: Shift Cryptosecurity<br>
:iphone: Product: BitBox02<br> 
:scroll: Title: Bypass of monotonic counter in MCU<br>
:nerd_face: Detail: The monotonic counter limiting the number of attempts to enter the correct password could be bypassed. The monotonic counter of the Secure Chip was still active though, thus limiting the number of available attempts to 730'500 attempts. Assuming a special made device for brute-forcing needs about 10 seconds to guess a password, reaching the upper limit would take approximately 85 days (non-stop). The probability of an attacker guessing, for example, a random 5 character password using lowercase, uppercase and digits is 0.08%, 6 characters is 0.012%, and 7 characters is 0.00002%. The vulnerability was patched with a series of robustness improvements to the firmware and by using the MCU's memory protection unit (MPU).<br>
:poop: Bug: Weakness in firmware hardening<br> 
:sunglasses: Reporter: Lazy Ninja<br>
:mega: Explanation from vendor: <https://medium.com/shiftcrypto/bitboxapp-4-16-0-with-bitbox02-firmware-5-0-0-release-7073ade23988><br>
:darf: Explanation from reporter: <https://www.cryptolazyninja.com/2019/12/bitbox02-weak-password-attack.html><br>

:office: Vendor: Coinkite<br>
:iphone: Product: Coldcard<br>
:scroll: Title: Multisig Change Script Vulnerability<br>
:nerd_face: Detail: The multisig change script could contain injected script opcodes. By adding a simple `OP_DROP` after the original multisig keys, the attacker could make the victim spend to an unintend address: `1 <pubA> <pubB> 2 CHECKMULTISIG DROP 1 <pubM0> <pubM1> 2 CHECKMULTISIG`. This was patched by ensuring that the redeem script remains the same.<br>
:poop: Bug: Bad transaction validation on device<br>
:sunglasses: Reporter: Dmitry Petukhov<br>
:clipboard: Patch: <https://github.com/Coldcard/firmware/commit/55f7cfd8ff6223a8f2a119519de2ee3c969bc06f/><br>
:mega: Explanation from vendor: <https://blog.coinkite.com/version-3.0.6-released/><br> 
:dart: Explanation from reporter: <https://gist.github.com/dgpv/c580080cd6984fb0121b61f1e1b5db51/><br>

# 2020

### January

:office: Vendor: Ledger<br>
:iphone: Product: Ledger Nano<br>
:scroll: Title: Monero Private Key Retrieval<br>
:nerd_face: Detail: Re-use of a parameter in the mlsag_sign function of the ledger monero app leads to possible spend key extraction by the host. Optimally, this parameter should be random and not re-used. In practice this was solved by keying the different HMACs used with specific values per operation. However there were multiple problems in the app that made the exploit easier. The writeup by ph4r05 gives a great overview of them.<br>
:poop: Bug: Bad Crypto Implementation<br>
:sunglasses: Reporter: ph4r05<br>
:mega: Explanation from vendor: <https://donjon.ledger.com/lsb/008/><br>
:dart: Explanation from reporter: <https://deadcode.me/blog/2020/04/25/Ledger-Monero-app-spend-key-extraction.html><br>

### March

:office: Vendor: Coinkite<br>
:iphone: Product: COLDCARD<br>
:scroll: Title: Supply Chain Attack with attacker controlled Firmware<br>
:nerd_face: Detail: The COLDCARD does a factory reset when an existing PIN is changed to an empty PIN , contrary to COLDCARD’s claims that a factory reset is impossible. This can be used to distribute tampered devices without much effort. COLDCARD has not patched the issue to date.<br>
:poop: Bug: Bad PIN check / zero condition<br>
:sunglasses: Reporter: TheCharlatan<br>
:mega: Explanation from vendor: <https://blog.coinkite.com/supply-chain-trust-minimized/><br>
:dart: Explanation from reporter: <https://thecharlatan.github.io/COLDCARD-Supply-Chain/><br>

:office: Vendor: Trezor<br>
:iphone: Product: Model T<br>
:scroll: Title: OP_RETURN treated as change output<br>
:nerd_face: Detail: By filling the address_n field with a change address in a Trezor protobuf message, an OP_RETURN transaction would be signed without user verification. This could potentially impact Omni Layer transactions that make use of the OP_RETURN data.<br>
:poop: Bug: Bad transaction validation on device<br>
:sunglasses: Reporter: Saleem Rashid<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/0903159d9b2df447434b9a5afdbca3eae8b4e52b><br>
:mega: Explanation from vendor:
<https://blog.trezor.io/details-of-firmware-updates-for-trezor-one-version-1-9-0-and-trezor-model-t-version-2-3-0-46deb141fc09><br>

:office: Vendor: Trezor<br>
:iphone: Product: Model T<br>
:scroll: Title: Malicious Change in Mixed Transactions<br>
:nerd_face: Detail: In Trezor's two stage transactiong validation and signing process claims about the addresses in the first stage were not sufficiently verified in the second stage. This could be used to insert a malicious 1of2 multisig change output into the transaction. This is very similar to an attack as discovered by Marko Bencun in October 2019.<br>
:poop: Bug: Bad transaction validation on device<br>
:sunglasses: Reporter: Saleem Rashid<br>
:mega: Explanation from vendor:
<https://blog.trezor.io/details-of-firmware-updates-for-trezor-one-version-1-9-0-and-trezor-model-t-version-2-3-0-46deb141fc09><br>

:office: Vendor: Trezor<br>
:iphone: Product: Model T<br>
:scroll: Title: Insufficient field size check in Protobuf<br>
:nerd_face: Detail: When signing a bitcoin transaction, the field length of the previous transaction output hash should always be 32 bytes long. The Trezor Model T did not check this field correctly. Hidden in this long prevhash could be an unrelated output that the Trezor would then sign as part of the transaction. The attacker can then spend coins on this signed output.<br>
:poop: Bug: Bad input validation and length restriction<br>
:sunglasses: Reporter: Saleem Rashid<br>
:clipboard: Patch: <https://github.com/trezor/trezor-firmware/commit/da89a17ce5c45972e5523dceb67ffbebf62d05c2><br>
:mega: Explanation from vendor:
<https://blog.trezor.io/details-of-firmware-updates-for-trezor-one-version-1-9-0-and-trezor-model-t-version-2-3-0-46deb141fc09><br>

:office: Vendor: Trezor<br>
:iphone: Product: Model T<br>
:scroll: Title: Inconsistent sanitization of transaction inputs<br>
:nerd_face: Detail: Yet another case of injecting a 1of2 multisig output as a change output. The attacker creates a single sig input and multisig output transaction. If the multisig field is sent in the protobuf message together with the single sig input, the device incorrectly marked the malicious multisig output as a change output.<br>
:poop: Bug: Bad input and transaction validation on device<br>
:sunglasses: Reporter: Saleem Rashid<br>
:mega: Explanation from vendor:
<https://blog.trezor.io/details-of-firmware-updates-for-trezor-one-version-1-9-0-and-trezor-model-t-version-2-3-0-46deb141fc09><br>


## Footnotes
### :sunglasses: Relevant blogs:
Christian Reitter: <https://blog.inhq.net/><br>
Saleem Rashid: <https://saleemrashid.com/><br>
wallet.fail: <https://wallet.fail/><br>
0xDEADC0DE / ph4r05: <https://deadcode.me/><br>
Lazy Ninja: <https://www.cryptolazyninja.com/><br>

### :office: Vendor Security Programs:
Trezor: <https://trezor.io/security/><br> 
Ledger: <https://donjon.ledger.com/bounty/><br>
Shift Cryptosecurity: <https://shiftcrypto.ch/policies/bug-bounty-policy/><br>
Shapeshift: <https://shapeshift.com/responsible-disclosure-program><br>
Coinkite: <https://coinkite.com/responsible-disclosure><br>

### :mega: Corporate Security Blogs:
Kraken Security: <https://blog.kraken.com/post/category/kraken-news/security/><br>
Ledger Donjon: <https://donjon.ledger.com/><br>

