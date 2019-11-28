_This is a dynamic document and changes as my understanding of these vulnerability changes and as new vulnerabilities get discovered_

# 2014
### Juli:

Vendor: Trezor 

Title: Malicious ScriptSig in transaction  
Description: A specially crafted transaction could extract the private key  
Type: Transaction validation attack with authentication  
:poop: Bug: Buffer Overflow  
Reporter: Nicolas Bacca (Ledger)  
Patch: https://github.com/trezor/trezor-firmware/commit/524f2a957afb66e6a869384aceaca1cb7f9cba60  

# 2015
### February:

Vendor: Trezor  
Title: SpendMultisig malicious change in transaction  
Description: A specially crafted transaction could contain a change output of an attacker, which wasn't confirmed by the user  
Type: Transaction validation attack with authentication 
Bug: Insufficient checks  
Reporter: Nicolas Bacca (Ledger)  
Patch: https://github.com/trezor/trezor-firmware/commit/137a60ce017c402ac160258bcc4b5f7b5aba0560  

### March:

Vendor: Trezor  
Title: Possible key extraction with oscilloscope  
Detail: With physical access to the device and an oscilloscope, the private key could have been extracted from the device  
Type: Signal noise / power analysis side channel  
Bug: Insufficient PIN protection for derivation of keys, minimize the usage of nested loops to increase const'ness of execution time  
Reporter: Jochen Hoenicke  
Patch: https://github.com/trezor/trezor-firmware/commit/7c6d2fe395c8475efbc93257892f0efac3d1511c  
Explanation from reporter: https://jochen-hoenicke.de/crypto/trezor-power-analysis/  

# 2017
### August:

Vendor: Trezor  
Title: SRAM memory access  
Detail: The SRAM was not cleared on soft reset, allowing extraction using special firmware and direct access to the device board  
Type: Platform reset attack  
Bug: (see detail)  
Reporter: Sunny  
Patch: https://github.com/trezor/trezor-firmware/commit/98e617d8740b85ae01d7d6e0dd3f49e66057a210  
Explanation from vendor: https://blog.trezor.io/fixing-physical-memory-access-issue-in-trezor-2b9b46bb4522  
Explanation from reporter: https://saleemrashid.com/2017/08/17/extracting-trezor-secrets-sram/  

# 2018 
### February:

Vendor: Trezor  
Title: STM32F205 chip issue  
Detail: The bootloader memory write-protection is not working as intended in the STM32F205, which is used in the Trezor One. The issue was solved by activating the Memory Protection Unit, keeping the bootloader safe from unauthorized write-access.  
Type: Supply chain attack  
Bug: Bad chip configuration  
Reporter: Saleem Rashid  
Patch: https://github.com/trezor/trezor-firmware/commit/9588e8f2736b60916f51e470deb18f55112a6ebc  
Explanation from vendor: https://blog.trezor.io/trezor-one-firmware-update-1-6-1-eecd0534ab95  

### March:

Vendor: Ledger  
Title: Padding oracle attack on SCP  
Detail: A padding oracle attack was found on the Secure Channel established between the device and Ledger’s HSM. It allows an attacker to decrypt the firmware updates.   
Bug: Bad padding of messages between MCU and SC  
Reporter: Timothee Isnard  
Explanation from vendor: https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/  

Vendor: Leder  
Title: Supply chain attack.  
Detail: The signature verification of the MCU can be bypassed, allowing an attacker to perform supply chain attacks. It requires a physical access to the device before the generation of the seed.  
Bug: Overall authentication architecture in MCU, fixed with a bunch of small patches  
Reporter: Saleem Rashid  
Explanation from vendor: https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/  
Explanation from reporter: https://saleemrashid.com/2018/03/20/breaking-ledger-security-model/  

Vendor: Ledger  
Title: Isolation vulnerability  
Detail: A malicious app can break the isolation between apps and access sensitive data managed by specific apps such as GPG, U2F or Neo.   
Bug: Null pointer dereferencing, pointer length not properly checked, Flash zone not wiped properly after device reset.  
Reporter: Sergei Volokitin  
Explanation from vendor: https://donjon.ledger.com/lsb/003/  
Explanation from reporter: https://i.blackhat.com/us-18/Wed-August-8/us-18-Volokitin-Software-Attacks-On-Hardware-Wallets.pdf  

### May: 

Vendor: Trezor  
Title: Race condition in recovery  
Detail: Specially crafted USB communication could trigger a stack overflow in recovery which could lead to code execution.  
Type: Stack overflow  
Bug: USB buffer overflow, during dry-run recovery which recursivly handles packets, a stack overflow can be triggered  
Reporter: Christian Reiter  
Patch: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  
Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-6-2-a3b25b668e98  

Vendor: Trezor  
Title: Message processing error  
Detai: Specially crafted USB packet could trigger a buffer overflow which could lead to code execution on older firmwares.  
Type: Buffer overflow  
Bug: USB buffer overflow if the USB message buffer is flooded with specially crafted incoming messages  
Reporter: Chrisian Reiter  
Patch: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  
Explanation from vendor: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  

### August

Vendor: Trezor  
Title: MPU circumvention via SYSCFG registers  
Detail: Security fix deployed via the 1.6.1 firmware update could be circumvented via clever use of the SYSCFG registers. This was fixed by completely disabling the SYSCFG registers via the MPU.  
Type: Supply chain attack  
Bug: MPU rule could be circumnavigated  
Reporter: Sunny  
Patch: https://github.com/trezor/trezor-firmware/commit/fdd5cbe20271634dc9ba4424ae40f1d11332cdf2  
Explanation from vendor: https://blog.trezor.io/trezor-one-firmware-update-1-6-3-73894c0506d  

### September

Vendor: Trezor  
Title: Buffer overflow in bech32_decode  
Detail: The C reference implementation for bech32 has an unsigned integer overflow that can lead to a buffer overflow. The bug was fixed by preventing the out-of-bounds accesses in the code.  
Type: Buffer overflow  
Bug: No sufficient out of bounds check  
Reporter: Christian Reiter  
Patch: https://github.com/trezor/trezor-firmware/commit/5c6b47288323a6cafe331304d2708a3c2a45f4b0  
Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8  

### October

Vendor: Trezor  
Title: Buffer overflow in cash_decode  
Detail: The cash_decode function in the trezor-crypto library allowed an out-of-bounds write. The bug was fixed by preventing the out-of-bounds accesses in the code.  
Type: Buffer/stack overflow  
Bug: No sufficient out of bounds check  
Reporter: Gabrial Campana  
Patch: https://github.com/trezor/trezor-firmware/commit/2bbbc3e15573294c6dd0273d2a8542ba42507eb0  
Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8  

Vendor: Trezor  
Title: Side-channel analysis (SCA) of PIN comparison  
Detail: Using a SCA bench an attacker could create the database of power consumption and electromagnetic traces of a device. This database could later be used to unlock a locked device using the same SCA bench. The issue was fixed by rewriting the device storage to not compare PINs directly, but rather compare random data stretched by the PIN.  
Type: Information leak  
Bug: Naive implementation of PIN storage  
Reporter: Charles Guillemet  
Patch: https://github.com/trezor/trezor-firmware/commit/4f32cb508383ec0e65843d037f6ac6473a668359  

### November

Vendor: Trezor  
Title: Information leak via U2F  
Detail: The C/C++ reference implementation for U2F by Yubico contains broken definition of a struct which can leak bytes from RAM via USB. The bug was fixed by updating the structure definition to a new correct one.  
Type: Information leak  
Bug: Bad struct memory layout  
Reporter: Christian Reitter  
Patch: https://github.com/trezor/trezor-firmware/commit/0b26c529ec49daf584f322f3ef959c79694c8cf5  
Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-2-3c97adbf121e  
Explanation from reporter: https://blog.inhq.net/posts/u2fhid_init_resp-information-leak/  

Vendor: Ledger  
Title: Bitcoin change address injection  
Detail: A vulnerability was found in the Bitcoin app allowing an attacker to add an unverified output change address into a legit transaction. It can lead to sending funds to an arbitrary address without requiring an additional confirmation on the device. The original transaction still has to be confirmed though.  
Bug: Bad Bitcoin transcation information validation  
Reporter: Sergey Lappo  
Explanation from vendor: https://donjon.ledger.com/lsb/004/  
Explanation from reporter: https://sergeylappo.github.io/ledger-hack/  

### December

Vendor: Trezor  
Title: SRAM Dump during the firmware update  
Detail: Using a special glitching hardware an attacker could trick the device processor into Read Protection level 1 which allows readout of RAM. The issue was fixed by not storing sensitive data in RAM during the firmware update.  
Type: Information leak  
Bug: Sensitive values in RAM during firmware update  
Reporter: wallet.fail   
Patch: https://github.com/trezor/trezor-firmware/commit/07231d936e41335b3ec44c4c6eb336be006890d0  
Explanation from vendor: https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012  
Explanation from reporter: https://media.ccc.de/v/35c3-9563-wallet_fail  

Vendor: Ledger  
Title: MCU Bootloader verification bypass.  
Detail: The signature verification of the Ledger Nano S MCU can be bypassed, allowing an attacker to install an arbitrary firmware on the MCU.  
Bug: f00dbabe  
Reporter: wallet.fail  
Explanation from vendor: https://donjon.ledger.com/lsb/005/  
Explanation from reporter: https://media.ccc.de/v/35c3-9563-wallet_fail  

# 2019
### January

Vendor: Trezor  
Title: Secret information leak via USB Descriptors  
Detail: The attack used specialized hardware to inject a fault into the comparison function in the USB stack. When timed properly, an attacker could trick USB stack into returning sensitive data via USB in the USB descriptor.  
Type: Information leak  
Bug: Outgoing packets too big, MPU did not protect sectors around the actual storage sectors, which would have halted execution        
Reporter: Colin O'Flynn  
Patch: https://github.com/trezor/trezor-firmware/commit/22f37e81a3270da5e8e5d6c55abc8f15f3a35567  
Explanation from vendor: https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012  

### April

Vendor: Trezor  
Title: Information leak via OLED display  
Detail: The attack uses power analysis to read the information shown on the OLED display.  
Type: Information leak  
Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant  
Reporter: Christian Reiter  
Patch: https://github.com/trezor/trezor-firmware/commit/f16c941ed4ac3c2e2c401de931249d0b2f34c29b  
Explanation from vendor: https://blog.trezor.io/details-of-the-oled-vulnerability-and-its-mitigation-d331c4e2001a  
Explanation from reporter: https://blog.inhq.net/posts/oled-side-channel-status-summary/  

Vendor: Ledger  
Title: OLED screen side-channel vulnerability.  
Detail: A side-channel leakage on the row-based OLED display was found. The power consumption of each row-based display cycle depends on the number of illuminated pixels, allowing a partial recovery of display contents. For example, a hardware implant in the USB cable might be able to leverage this behavior to recover confidential secrets such as the PIN and BIP39 mnemonic. In other words, the side-channel is relevant only if the attacker has enough control over the device’s USB connection to make power-consumption measurements and advanced statistical analysis while the secret data is displayed. The side-channel is not relevant in other circumstances, such as a stolen device that is not currently displaying secret data.  
Type: Information leak  
Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant  
Reporter: Christian Reiter
Explanation from vendor: https://donjon.ledger.com/lsb/006/  
Explanation from reporter: https://blog.inhq.net/posts/oled-side-channel-status-summary/  


### October

Vendor: Trezor  
Title: Malicious change in a mixed transaction  
Detail: An attacker could create a specially crafted multisig transaction which would hide the multisig change address.  
Type: Missing Check  
Bug: Input and output Bitcoin transaction fingerprints were not sufficiently checked.  
Reporter: Marko Bencun  
Patch: https://github.com/trezor/trezor-firmware/commit/8eb6ce08995514c67d175b7197feeadeccc48ff0  
Explanation from vendor: https://blog.trezor.io/details-of-the-multisig-change-address-issue-and-its-mitigation-6370ad73ed2a  
Explanation from reporter: https://medium.com/shiftcrypto/a-remote-theft-attack-on-trezor-model-t-44127cd7fb5a  

Vendor: Ledger  
Title: Monero private key retrieval.  
Detail: The Monero App for Ledger Nano was found to be vulnerable to a private key retrieval through the use of a malicious Monero Client (desktop application). Some computational elements are encrypted by the Nano S with a key only known to the Monero application, and sent to the desktop client for later use, due to space limitations on the Nano. During the final step of the signature (MLSAG sign), the client sends back some sensitive encrypted elements which the app uses to compute a Schnorr signature. A malicious client can misuse this by replaying earlier elements of this computation, and induce a variant of a nonce-reuse attack (see for example the PS3 Fail). This replay of commands is possible because the key derived by the app to encrypt elements is static, and there is no message authentication.  
Bug: Bad MLSA signature implementation   
Patch: https://github.com/LedgerHQ/ledger-app-monero/commit/5d0658ad6369f3d0ff2d10ee9effa410eb185b98  
Explanation from vendor: https://donjon.ledger.com/lsb/007/  


## Footnotes
### Relevant blogs:
Christian Reiter: https://blog.inhq.net/  
Saleem Rashid: https://saleemrashid.com/  
wallet.fail: https://wallet.fail/  

### Vendor Security Programs:
Trezor: https://trezor.io/security/  
Ledger: https://donjon.ledger.com/bounty/  







