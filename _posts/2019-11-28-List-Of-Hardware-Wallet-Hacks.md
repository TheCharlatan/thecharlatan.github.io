_This is a dynamic document and changes as my understanding of these vulnerability changes and as new vulnerabilities get discovered_

What constitutes a hardware wallet hack?   
I count anything as a "hack" that allows a hacker to change a hardware wallet's intended behavior. This means it is not relevant to me if the hack was ever exploited, or if it has received a low likelihood rating from vendors.

# 2014
### Juli:

:office: Vendor: Trezor 

:scroll: Title: Malicious ScriptSig in transaction  
Description: A specially crafted transaction could extract the private key  
:eyes: Type: Transaction validation attack with authentication  
:poop: Bug: Buffer Overflow  
:sunglasses: Reporter: Nicolas Bacca (Ledger)  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/524f2a957afb66e6a869384aceaca1cb7f9cba60  

# 2015
### February:

:office: Vendor: Trezor  
:scroll: Title: SpendMultisig malicious change in transaction  
Description: A specially crafted transaction could contain a change output of an attacker, which wasn't confirmed by the user  
:eyes: Type: Transaction validation attack with authentication   
:poop: Bug: Insufficient transaction checks  
:sunglasses: Reporter: Nicolas Bacca (Ledger)  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/137a60ce017c402ac160258bcc4b5f7b5aba0560  

### March:

:office: Vendor: Trezor  
:scroll: Title: Possible key extraction with oscilloscope  
:nerd_face: Detail: With physical access to the device and an oscilloscope, the private key could have been extracted from the device  
:eyes: Type: Signal noise / power analysis side channel  
:poop: Bug: Insufficient PIN protection for derivation of keys, minimize the usage of nested loops to increase const'ness of execution time  
:sunglasses: Reporter: Jochen Hoenicke  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/7c6d2fe395c8475efbc93257892f0efac3d1511c  
:dart: Explanation from reporter: https://jochen-hoenicke.de/crypto/trezor-power-analysis/  

# 2017
### August:

:office: Vendor: Trezor  
:scroll: Title: SRAM memory access  
:nerd_face: Detail: The SRAM was not cleared on soft reset, allowing extraction using special firmware and direct access to the device board  
:eyes: Type: Platform reset attack  
:poop: Bug: (see detail)  
:sunglasses: Reporter: Sunny  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/98e617d8740b85ae01d7d6e0dd3f49e66057a210  
:mega: Explanation from vendor: https://blog.trezor.io/fixing-physical-memory-access-issue-in-trezor-2b9b46bb4522  
:dart: Explanation from reporter: https://saleemrashid.com/2017/08/17/extracting-trezor-secrets-sram/  

# 2018 
### February:

:office: Vendor: Trezor  
:scroll: Title: STM32F205 chip issue  
:nerd_face: Detail: The bootloader memory write-protection is not working as intended in the STM32F205, which is used in the Trezor One. The issue was solved by activating the Memory Protection Unit, keeping the bootloader safe from unauthorized write-access.  
:eyes: Type: Supply chain attack  
:poop: Bug: Bad chip configuration  
:sunglasses: Reporter: Saleem Rashid  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/9588e8f2736b60916f51e470deb18f55112a6ebc  
:mega: Explanation from vendor: https://blog.trezor.io/trezor-one-firmware-update-1-6-1-eecd0534ab95  

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01  
:scroll: Title: Bad BIP32 implementation  
:nerd_face: Detail: Accessing the 'xpub' API command for the master key path of both the hidden and the standard wallet allowed for the reconstructing of the private keys of the standard and hidden wallet.   
:eyes: Type: API remote attack   
:poop: Bug: Bad cryptography for the wallet vs hidden wallet derivation   
:sunglasses: Reporter: Saleem Rashid   
:mega: Explanation from vendor: https://shiftcrypto.ch/bitbox01/disclosure   
:dart: Explanation from reporter: https://saleemrashid.com/2018/11/26/breaking-into-bitbox   

### March:

:office: Vendor: Ledger  
:scroll: Title: Padding oracle attack on SCP  
:nerd_face: Detail: A padding oracle attack was found on the Secure Channel established between the device and Ledger’s HSM. It allows an attacker to decrypt the firmware updates.   
:poop: Bug: Bad padding of messages between MCU and SC  
:sunglasses: Reporter: Timothee Isnard  
:mega: Explanation from vendor: https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/  

:office: Vendor: Ledger  
:scroll: Title: MCU signature verification bypass   
:nerd_face: Detail: The signature verification of the MCU can be bypassed, allowing an attacker to perform supply chain attacks. It requires a physical access to the device before the generation of the seed.  
:eyes: Type:  Supply chain attack  
:poop: Bug: Overall authentication architecture in MCU, fixed with a bunch of small patches  
:sunglasses: Reporter: Saleem Rashid  
:mega: Explanation from vendor: https://www.ledger.com/firmware-1-4-deep-dive-security-fixes/  
:dart: Explanation from reporter: https://saleemrashid.com/2018/03/20/breaking-ledger-security-model/  

:office: Vendor: Ledger  
:scroll: Title: Isolation vulnerability  
:nerd_face: Detail: A malicious app can break the isolation between apps and access sensitive data managed by specific apps such as GPG, U2F or Neo.   
:poop: Bug: Null pointer dereferencing, pointer length not properly checked, Flash zone not wiped properly after device reset.  
:eyes: Type: Privilege escelation  
:sunglasses: Reporter: Sergei Volokitin  
:mega: Explanation from vendor: https://donjon.ledger.com/lsb/003/  
:dart: Explanation from reporter: https://i.blackhat.com/us-18/Wed-August-8/us-18-Volokitin-Software-Attacks-On-Hardware-Wallets.pdf  

### May

:office: Vendor: Trezor  
:scroll: Title: Race condition in recovery  
:nerd_face: Detail: Specially crafted USB communication could trigger a stack overflow in recovery which could lead to code execution.  
:eyes: Type: Stack overflow  
:poop: Bug: USB buffer overflow, during dry-run recovery which recursivly handles packets, a stack overflow can be triggered  
:sunglasses: Reporter: Christian Reiter  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  
:mega: Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-6-2-a3b25b668e98  

:office: Vendor: Trezor  
:scroll: Title: Message processing error  
:nerd_face: Detail: Specially crafted USB packet could trigger a buffer overflow which could lead to code execution on older firmwares.  
:eyes: Type: Buffer overflow  
:poop: Bug: USB buffer overflow if the USB message buffer is flooded with specially crafted incoming messages  
:sunglasses: Reporter: Chrisian Reiter  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  
:mega: Explanation from vendor: https://github.com/trezor/trezor-firmware/commit/c9113fd3f5fcd78e9e560dbac75ed5aae359eb2d  

### July

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Simulating the secure chip    
:nerd_face: Detail: After physically breaking apart the BitBox casing, attaching invasive probes, and manipulating the data sent to the BitBox's microcontroller, a BitBox could be reset but without erasing the wallet secrets. A patch was provided on 31 July 2018. If the bottom casing of your BitBox has not been removed or tampered with, you are not at risk.     
:eyes: Type: Information leak     
:poop: Bug: Secrets not cleared after wallet reset  
:sunglasses: Reporter: Saleem Rashid   
:mega: Explanation from vendor: https://shiftcrypto.ch/bitbox01/disclosure/   
:dart: Explanation from reporter: https://saleemrashid.com/2018/11/26/breaking-into-bitbox   

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox.   
:nerd_face: Detail: When initially pairing a BitBox to the mobile verification app, a man-in-the-middle (MITM) on a compromised computer could insert themselves and then later change the information to be displayed on the mobile app. We provided a patch on 31 July 2018 in firmware v4.0.0. The vulnerability existed only during the initial pairing and if your computer was compromised by an attacker aware of the issue.   
:eyes: Type: Information leak     
:poop: Bug: Bad Crypto  
:sunglasses: Reporter: Saleem Rashid   
:mega: Explanation from vendor: https://shiftcrypto.ch/bitbox01/disclosure/   
:dart: Explanation from reporter: https://saleemrashid.com/2018/11/26/breaking-into-bitbox   



### August

:office: Vendor: Trezor  
:scroll: Title: MPU circumvention via SYSCFG registers  
:nerd_face: Detail: Security fix deployed via the 1.6.1 firmware update could be circumvented via clever use of the SYSCFG registers. This was fixed by completely disabling the SYSCFG registers via the MPU.  
:eyes: Type: Supply chain attack  
:poop: Bug: MPU rule could be circumnavigated  
:sunglasses: Reporter: Sunny  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/fdd5cbe20271634dc9ba4424ae40f1d11332cdf2  
:mega: Explanation from vendor: https://blog.trezor.io/trezor-one-firmware-update-1-6-3-73894c0506d  

### September

:office: Vendor: Trezor  
:scroll: Title: Buffer overflow in bech32_decode  
:nerd_face: Detail: The C reference implementation for bech32 has an unsigned integer overflow that can lead to a buffer overflow. The bug was fixed by preventing the out-of-bounds accesses in the code.  
:eyes: Type: Buffer overflow  
:poop: Bug: No sufficient out of bounds check  
:sunglasses: Reporter: Christian Reiter  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/5c6b47288323a6cafe331304d2708a3c2a45f4b0  
:mega: Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8  

### October

:office: Vendor: Trezor  
:scroll: Title: Buffer overflow in cash_decode  
:nerd_face: Detail: The cash_decode function in the trezor-crypto library allowed an out-of-bounds write. The bug was fixed by preventing the out-of-bounds accesses in the code.  
:eyes: Type: Buffer/stack overflow  
:poop: Bug: No sufficient out of bounds check  
:sunglasses: Reporter: Gabrial Campana  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/2bbbc3e15573294c6dd0273d2a8542ba42507eb0  
:mega: Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-1-5c34278425d8  

:office: Vendor: Trezor  
:scroll: Title: Side-channel analysis (SCA) of PIN comparison  
:nerd_face: Detail: Using a SCA bench an attacker could create the database of power consumption and electromagnetic traces of a device. This database could later be used to unlock a locked device using the same SCA bench. The issue was fixed by rewriting the device storage to not compare PINs directly, but rather compare random data stretched by the PIN.  
:eyes: Type: Information leak  
:poop: Bug: Naive implementation of PIN storage  
:sunglasses: Reporter: Charles Guillemet  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/4f32cb508383ec0e65843d037f6ac6473a668359  

### November

:office: Vendor: Trezor  
:scroll: Title: Information leak via U2F  
:nerd_face: Detail: The C/C++ reference implementation for U2F by Yubico contains broken definition of a struct which can leak bytes from RAM via USB. The bug was fixed by updating the structure definition to a new correct one.  
:eyes: Type: Information leak  
:poop: Bug: Bad struct memory layout  
:sunglasses: Reporter: Christian Reiter  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/0b26c529ec49daf584f322f3ef959c79694c8cf5  
:mega: Explanation from vendor: https://blog.trezor.io/details-about-the-security-updates-in-trezor-one-firmware-1-7-2-3c97adbf121e  
:dart: Explanation from reporter: https://blog.inhq.net/posts/u2fhid_init_resp-information-leak/  

:office: Vendor: Ledger   
:scroll: Title: Bitcoin change address injection  
:nerd_face: Detail: A vulnerability was found in the Bitcoin app allowing an attacker to add an unverified output change address into a legit transaction. It can lead to sending funds to an arbitrary address without requiring an additional confirmation on the device. The original transaction still has to be confirmed though.  
:poop: Bug: Bad Bitcoin transcation information validation  
:sunglasses: Reporter: Sergey Lappo  
:mega: Explanation from vendor: https://donjon.ledger.com/lsb/004/  
:dart: Explanation from reporter: https://sergeylappo.github.io/ledger-hack/  

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Poking around the secure chip  
:nerd_face: Detail: Bad configuration of the secure chip leaves it redundant in the BitBox01 hardware design. This is not patchable.  
:eyes: Type: Break of existing security model, lead to re-assesment of public security claims  
:poop: Bug: Bad secure chip configuration.  
:sunglasses: Reporter: Saleem Rashid   
:dart: Explanation from reporter: https://saleemrashid.com/2018/11/26/breaking-into-bitbox/  

### December

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox  
:nerd_face: Detail: Encrypted USB communication, when not authenticated, can be modified by a man-in-the-middle (MITM) attacker in undesirable ways. A patch was provided on 4 December 2018 in firmware v5.0.0.   
:eyes: Type: Break on-hardware verification   
:poop: Bug: Usage of a plain AES-256-CBC cipher for authentication. Never use encryption for authentication.   
:sunglasses: Reporter: Saleem Rashid   
:mega: Explanation from vendor: https://shiftcrypto.ch/bitbox01/disclosure/  


:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Man-in-the-middle (MITM) between the mobile verification app and the BitBox  
:nerd_face: Detail: Encrypted USB communication, when not authenticated, can be modified by a man-in-the-middle (MITM) attacker in undesirable ways. A patch was provided on 4 December 2018 in firmware v5.0.0.  
:eyes: Type: Break on-hardware verification   
:poop: Bug: Usage of a plain AES-256-CBC cipher for authentication. Never use encryption for authentication.   
:sunglasses: Reporter: Saleem Rashid   
:mega: Explanation from vendor: https://shiftcrypto.ch/bitbox01/disclosure/  

:office: Vendor: Trezor  
:scroll: Title: SRAM Dump during the firmware update  
:nerd_face: Detail: Using a special glitching hardware an attacker could trick the device processor into Read Protection level 1 which allows readout of RAM. The issue was fixed by not storing sensitive data in RAM during the firmware update.  
:eyes: Type: Information leak  
:poop: Bug: Sensitive values in RAM during firmware update  
:sunglasses: Reporter: wallet.fail   
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/07231d936e41335b3ec44c4c6eb336be006890d0  
:mega: Explanation from vendor: https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012  
:dart: Explanation from reporter: https://media.ccc.de/v/35c3-9563-wallet_fail  

:office: Vendor: Ledger  
:scroll: Title: MCU Bootloader verification bypass.  
:nerd_face: Detail: The signature verification of the Ledger Nano S MCU can be bypassed, allowing an attacker to install an arbitrary firmware on the MCU.  
:poop: Bug: f00dbabe  
:sunglasses: Reporter: wallet.fail  
:mega: Explanation from vendor: https://donjon.ledger.com/lsb/005/  
:dart: Explanation from reporter: https://media.ccc.de/v/35c3-9563-wallet_fail  

# 2019
### January

:office: Vendor: Trezor  
:scroll: Title: Secret information leak via USB Descriptors  
:nerd_face: Detail: The attack used specialized hardware to inject a fault into the comparison function in the USB stack. When timed properly, an attacker could trick USB stack into returning sensitive data via USB in the USB descriptor.   
:eyes: Type: Information leak  
:poop: Bug: Outgoing packets too big, MPU did not protect sectors around the actual storage sectors, which would have halted execution        
:sunglasses: Reporter: Colin O'Flynn  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/22f37e81a3270da5e8e5d6c55abc8f15f3a35567  
:mega: Explanation from vendor: https://blog.trezor.io/details-of-security-updates-for-trezor-one-firmware-1-8-0-and-trezor-model-t-firmware-2-1-0-408e59dc012  

:office: Vendor: Coldcard  
:scroll: Title: Attack on Coldcard short PINs    
:nerd_face: Detail:  The attack is achieved by connecting a man-in-the-middle (MITM) to the bus the CCW uses to communicate with its secure element (SE). Then commands on the bus are modified to cause the MCU to not count failed PIN entry attempts.  This gives the attacker an unlimited number of attempts to guess the PIN.   
:eyes: Type: Bypass of authentication    
:poop: Bug: Bad secure chip answer verification  
:sunglasses: Reporter: Lazy Ninja  
:mega: Explanation from vendor: https://blog.coinkite.com/use-long-pins/  
:dart: Explanation from reporter: https://www.cryptolazyninja.com/2019/03/coldcard-wallet-short-pin-brute-force.html

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Information leak via U2F  
:nerd_face: Detail: The C/C++ reference implementation for U2F by Yubico contains broken definition of a struct which can leak bytes from RAM via USB. The bug was fixed by updating the structure definition to a new correct one.  
claims  
:poop: Bug: Bad struct memory layout  
:sunglasses: Reporter: Christian Reiter  
:mega: Explanation from vendor: https://medium.com/shiftcrypto/important-security-news-about-version-4-4-0-upgrade-2449b745be9  
:dart: Explanation from reporter: https://blog.inhq.net/posts/u2fhid_init_resp-information-leak/  


### March

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: BIP32 address derivation ransom attack  
:nerd_face: Detail: No restrictions on possible BIP32 key paths led to a ransom attack  
:poop: Bug: Bad interpretation of BIP32 and BIP44 standard  
:mega: Explanation from vendor: https://medium.com/shiftcrypto/bitbox-desktop-app-4-6-0-with-firmware-6-0-3-release-ec46937afe7c  https://medium.com/shiftcrypto/bitbox-desktop-app-4-5-0-with-firmware-6-0-2-release-fd77f8186a29


### April

:office: Vendor: Trezor  
:scroll: Title: Information leak via OLED display  
:nerd_face: Detail: The attack uses power analysis to read the information shown on the OLED display.  
:eyes: Type: Information leak  
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant  
:sunglasses: Reporter: Christian Reiter  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/f16c941ed4ac3c2e2c401de931249d0b2f34c29b  
:mega: Explanation from vendor: https://blog.trezor.io/details-of-the-oled-vulnerability-and-its-mitigation-d331c4e2001a  
:dart: Explanation from reporter: https://blog.inhq.net/posts/oled-side-channel-status-summary/  

:office: Vendor: Ledger  
:scroll: Title: OLED screen side-channel vulnerability.  
:nerd_face: Detail: A side-channel leakage on the row-based OLED display was found. The power consumption of each row-based display cycle depends on the number of illuminated pixels, allowing a partial recovery of display contents. For example, a hardware implant in the USB cable might be able to leverage this behavior to recover confidential secrets such as the PIN and BIP39 mnemonic. In other words, the side-channel is relevant only if the attacker has enough control over the device’s USB connection to make power-consumption measurements and advanced statistical analysis while the secret data is displayed. The side-channel is not relevant in other circumstances, such as a stolen device that is not currently displaying secret data.  
:eyes: Type: Information leak  
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant  
:sunglasses: Reporter: Christian Reiter  
:mega: Explanation from vendor: https://donjon.ledger.com/lsb/006/  
:dart: Explanation from reporter: https://blog.inhq.net/posts/oled-side-channel-status-summary/  

:office: Vendor: Coldcard  
:scroll: Title: Possible Display Information Leak    
:nerd_face: Detail: The attack uses power analysis to read the information shown on the OLED display.  
:eyes: Type: Information leak  
:poop: Bug: OLED screens consume power based on number of pixels that are on. Mitigated here by making the number of pixels that are on per row when displaying the seed constant  
:sunglasses: Reporter: Christian Reiter  
:mega: Explanation from vendor: https://blog.coinkite.com/noise-troll/   
:dart: Explanation from reporter: https://blog.inhq.net/posts/oled-side-channel-status-summary/  

### June

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Blinking pattern mismatch    
:nerd_face: Detail: The blinking patterns of the BitBox01 reveal important information on the behvaiour of the device  
:poop: Bug: Bad differentiation between modes for the user  
:mega: Explanation from vendor: https://medium.com/shiftcrypto/bitbox-desktop-app-4-9-0-with-bitbox01-firmware-6-1-1-release-1b84c5f9295f  


### October

:office: Vendor: Trezor  
:scroll: Title: Malicious change in a mixed transaction  
:nerd_face: Detail: An attacker could create a specially crafted multisig transaction which would hide the multisig change address.  
:eyes: Type: Missing Check  
:poop: Bug: Input and output Bitcoin transaction fingerprints were not sufficiently checked.  
:sunglasses: Reporter: Marko Bencun  
:clipboard: Patch: https://github.com/trezor/trezor-firmware/commit/8eb6ce08995514c67d175b7197feeadeccc48ff0  
:mega: Explanation from vendor: https://blog.trezor.io/details-of-the-multisig-change-address-issue-and-its-mitigation-6370ad73ed2a  
:dart: Explanation from reporter: https://medium.com/shiftcrypto/a-remote-theft-attack-on-trezor-model-t-44127cd7fb5a  

:office: Vendor: Ledger  
:scroll: Title: Monero private key retrieval.  
:nerd_face: Detail: The Monero App for Ledger Nano was found to be vulnerable to a private key retrieval through the use of a malicious Monero Client (desktop application). Some computational elements are encrypted by the Nano S with a key only known to the Monero application, and sent to the desktop client for later use, due to space limitations on the Nano. During the final step of the signature (MLSAG sign), the client sends back some sensitive encrypted elements which the app uses to compute a Schnorr signature. A malicious client can misuse this by replaying earlier elements of this computation, and induce a variant of a nonce-reuse attack (see for example the PS3 Fail). This replay of commands is possible because the key derived by the app to encrypt elements is static, and there is no message authentication.  
:poop: Bug: Bad MLSAG signature implementation   
:clipboard: Patch: https://github.com/LedgerHQ/ledger-app-monero/commit/5d0658ad6369f3d0ff2d10ee9effa410eb185b98  
:mega: Explanation from vendor: https://donjon.ledger.com/lsb/007/  

:office: Vendor: Coldcard    
:scroll: Title: Troublesome Change Outputs   
:nerd_face: Detail: It is possible to make a valid PSBT file that sends the change left from a transaction to a unknown location. If an attacker had your XPUB, and could change your PSBT file before you sign, they could modify the file so that the “change” (ie. the balance of Bitcoins you are sending back to yourself) goes to an effectively unknown address. If the attacker is profit motivated, they can ransom the knowledge of those change UTXO back to you.   
:poop: Bug: BIP32 address derivation ransom attack  
:sunglasses: Reporter: TheCharlatan   
:mega: Explanation from vendor: https://blog.coinkite.com/troublesome-change/   
:dart: Explanation from reporter: https://thecharlatan.github.io/Ransom-Coldcard/  

:office: Vendor: Coldcard    
:scroll: Title: Ransom attack on Coldcard's receive address verification  
:nerd_face: Detail: By inserting newlines in the derivation path string sent to the Coldcard, the displayed characters could be split. This could trick users into verifying an address for a BIP32 derivation path that is not easily accessible.  
:sunglasses: Reporter: TheCharlatan  
:poop: Bug: Bad input validation from host   
:dart: Explanation from reporter: https://thecharlatan.github.io/Ransom-Coldcard/  

:office: Vendor: Shift Cryptosecurity  
:iphone: Product: BitBox01   
:scroll: Title: Mobile pairing information leak BitBox01   
:nerd_face: Detail: ?  
:poop: Bug: Bad cryptography  
:mega: Explanation from vendor: https://medium.com/shiftcrypto/bitboxapp-4-14-0-5e72575b0819  


## Footnotes
### Relevant blogs:
Christian Reiter: https://blog.inhq.net/  
Saleem Rashid: https://saleemrashid.com/  
wallet.fail: https://wallet.fail/  

### :office: Vendor Security Programs:
Trezor: https://trezor.io/security/  
Ledger: https://donjon.ledger.com/bounty/  
Shift Cryptosecurity: https://shiftcrypto.ch/policies/bug-bounty-policy/






