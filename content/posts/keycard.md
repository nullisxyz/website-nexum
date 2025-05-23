+++
title = "Banking the unbanked with the technology developed by banks"
date = "2025-05-21"
updated = "2025-05-21"
slug = "keycard"

[taxonomies]
tags=["keycard", "javacard", "iso7816"]
+++

## Introduction

This is the first in a four part series on [Keycard](https://keycard.tech), a smart-card Web3 wallet implementation. This series will cover the following topics:

- The path to banking the unbanked (this article)
- A novel approach to hardware wallet protocol security and transaction verification
- Hierarchial accounts - organise your accounts neatly
- Cashcard - mutually authenticated off-chain transactions

## What is Keycard?

[Keycard](https://keycard.tech) is a [JavaCard](https://www.javacard.com/) smart-card Web3 wallet implementation designed for [Status.IM](https://status.im) and built by a team of developers under the umbrella of [IFT](https://free.technology).
It builds on open technology standards that have been around for many years, such as [ISO 7816](https://www.iso.org/standard/67495.html) and [JavaCard](https://www.javacard.com/) - the very same standards that are powering smart cards today used by giants such as Visa, Mastercard, and American Express.

In the smart-card form-factor, Keycard can be used via physical contact with a smart-card reader, or alternatively via an NFC-enabled device.
It's resilient to tampering and able to function in a wide range of environments.
It's actually a hardware *wallet*, in the sense that it can withstand the everyday harsh conditions of the real world.
It's arguably the most resilient hardware wallet available today - leaning on standards used by financial institutions that have been developed over many years as consumers put the technology through its paces.
Accidentally took your hardware wallet for a swim? No problem - if it's a Keycard.

## Where Keycard currently stands

[Keycard](https://keycard.tech) refers to the JavaCard Web3 wallet implementation.
It provides a protocol consisting of a secure channel, key management, and signing.
All the standards for the Keycard communication protocol are open source and [documented](https://keycard.tech/docs/).
The JavaCard applet source code is published on their [GitHub repository](https://github.com/keycard-tech/status-keycard).

There are two notable areas whereby Keycard differs significantly from other hardware wallets:

### No exporting of seed phrases

Keycard does **NOT** allow for exporting of seed phrases and only allows for exporting very select private keys under the ([EIP-1581](https://eips.ethereum.org/EIPS/eip-1581)) derivation path. This makes it a secure and convenient solution for managing Web3 accounts - you never have to write down a seedphrase and risk it falling into the wrong hands - but also presents the risk that if you lose your Keycard, access to any funds stored on accounts derived from the Keycard are lost forever.

### No physical buttons and/or screen

Keycard is a smart-card, which means it has no physical buttons and/or screen. It is therefore not possible to review transactions or confirm actions prior to signing. It's for this reason that the core developers of Keycard are working on the [Keycard Shell](https://keycard.tech/keycard-shell), a small form-factor smart-card reader that can be used to interact with the Keycard, and process Web3 transactions via QR code - with a screen to review transactions and confirm actions prior to signing.

## Where to from here?

Keycard is just getting started for mainstream adoption, but the offerings at the moment, while comparable to other hardware wallets, are limiting in terms of the full functionality of Keycard.

The following are a glimpse to the future of Keycard integration within Nexum (and the subjects of subsequent blog posts):

### Secure channel with viewers

Keycard implements its own Secure Channel Protocol, which allows for secure communication between the Keycard applet on the smart-card and the application connected to it via a smart-card reader or NFC. The channel is **encrypted** and **integrity protected**, ensuring that any data transmitted is secure and cannot be tampered with - a feature **not found in other hardware wallets**. By expanding on the transport layer for the connection, it is possible with Keycard's current implementation to pass view-only permissions to other "man in the middle" devices, such as a mobile phone application, or a web application, in order to view the transactions / messages prior to signing. Of significance here is that the **number of "man in the middle" devices is unbounded, able to be expanded to an arbitrary number of devices**, making it very difficult for an adversary to assess a target's security posture. This is unique as it allows for a user to change their security posture at will, without having to reconfigure their Keycard / wallet.

> **Example**: A Keycard user is signing a Safe transaction on a multi-sig that contains a significant sum of funds. Normally, the user only uses 1 viewing device for verification, however for this specific signing event, the user elects to use 3 devices through which to verify the transaction prior to signing - reflecting the heightened security posture. This could be a mobile phone application, a web application, or even a physical device such as a smart-card reader.

More on this will be covered in a future blog post.

### Account hierarchies and non-asset accounts

Keycard is a [BIP32](https://github.com/bitcoin/bips/blob/43d4a1ecec42d0d4160eafb8b7f37c14f279141b/bip-0032.mediawiki) compatible wallet, which means it supports hierarchical deterministic wallets. This allows for the creation of multiple accounts, each with its own set of addresses and keys. Most wallet software allows for a user to specify specific derivation paths for an account, but we feel that this is not always the best approach. Instead we will seek to abstract this from the user, allowing the user to create multiple accounts in a "folder and file" like structure.

> **Example**: A user chooses to derive their personal and business accounts from the same Keycard, giving the following account hierarchy:
>
> - Personal
>   - Savings
>  - Checking
> - Business
>   - Accounts Payable
>   - Accounts Receivable

In doing so, the derivation path can be automatically generated based on the account name and parent account name (or potentially randomised). Account metadata storing the derivation path and other relevant information will be stored in a client-side database (with optional encryption and cloud backup). As the metadata is non-critical (privacy concerns not-withstanding), and EIP-1581 subtree derived can be used for the encryption of the metadata, ensuring all credentials are retained on the Keycard.

Additionally, further EIP-1581 sub-accounts can be utilised by other low-security applications, such as [Waku](https://waku.org) for messaging and communication or [Swarm](https://ethswarm.org) for decentralised storage.

More on this will be covered in a future blog post.

### Unbounded TPS with mutual transactions

Keycards (smart-cards with the Keycard package installed) actually contain multiple "applets". These applets are:

1. Keycard (the main applet that we have been discussing)
2. NDEF (NFC Data Exchange Format)
3. Cashcard (a simple, permissionless signer)
4. Identity (a means of verifying the authenticity of the Keycard from manufacture)

In the development that we have been doing with Keycard, what caught our attention the most was *Cashcard*. In it's current state, Cashcard allows for signing transactions / messages with a user-configurable derivation path under the Keycard applet's main account. With this case, you could tap on a payment terminal or mobile phone to authorise some low-value transaction. This is the normal "buying a cup of coffee" scenario. While this is nice in theory, in practice there are several security concerns:

1. **Fraudulent transactions**: A malicious terminal could present a "real" payment request, but actually sign a different transaction.
2. **No verification**: All transactions signed by the Cashcard applet are not verified by the user or any other party. This means that the user cannot be sure that the transaction they are signing is the one they intended to sign.

The general idea that has been circulated to overcome these issues is to ensure that cashcard is only used for very low-value transactions. This in turn creates a poor user experience as the user is constantly in need of topping up the cashcard, but at the same time, has no way of knowing if there was a malicious transaction that has already been signed with some future `nonce`. This leaves a user's cashcard vulnerable to griefing. To mitigate this, we propose to design a solution that allows for mutual transactions between two Keycards - creating an effective **mutual transaction** system

By utilising a **smart-contract vault** to store balances of *all* cashcards, we propose to allow for "adding value" to a cashcard via a deposit to this contract. The balances for all deposits to the smart-contract vault are also **stored on the card itself** (addition of deposits to the card only happens via trusted channels). The card can then mutually authenticate other cashcards, verifying their authenticity, prior to accepting a transfer of a deposit from another cashcard (effectively creating a mutual transaction). To complete the loop, the cashcard can then request a withdrawal of part or all of the deposit from the smart-contract vault. This system can be thought of as a decentralised version of Japan's IC card system, but with the added benefit of being able to support arbitrary tokens, and not just one currency. By encouraging mutual transactions, we eliminate the blockchain bottleneck for TPS (**and allow off-chain, off-line transactions**), but maintain its security and decentralised nature.

This system would then be complimentary to Keycard, presenting a comprehensive "velocity of money" approach to wallet-to-account management. This allows for:
1. Cheap, low-value point-of-sale transactions using cashcard mutual transactions.
2. Personal, medium-value savings potentially held in Keycard accounts (requires PIN and pairing password for authentication).
3. High value savings held in smart-contract wallets with Keycard sub-accounts as signers, allowing for distributed control and management.

More on this will be covered in a future blog post.

## Conclusion

Keycard represents an intersection between Web3 and conventional form-factor technology and user experience that is ubiquitous in the traditional banking world. By preserving UX that is familiar to users, we can create a seamless transition from traditional banking to Web3. This system allows for mutual transactions, eliminating the need for blockchain transactions and reducing the cost of transactions. The additional benefits of a secure and decentralised system remain core to the architecture, but abstracted from everyday users.
