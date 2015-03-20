# Signet Overview

This document describes a standardized methodology for generating identity certificates in OpenPGP. It outlines specifications for an identity document, including the formatting of identity information and choice of cryptographic algorithms and key sizes; guidelines for how artifacts of the identity generation process should be stored and used; as well as a description of the identity lifecycle, including information about certification and trust. 

Collectively, these are called the **Signet Guidelines**. 

## Conceptual Overview

Before we get into the specifics of the Signet guidelines, this section offers a high-level overview of the nature and structure of an OpenPGP identity. It is presented to offer an understanding of how the pieces fit together. 

Colloquially, people often refer to the artifact published at the end of this procedure as their "public key". This is a bit of a misnomer; a public key merely is half of a key pair, which is just a set of related numbers that enable cryptographic operations. The artifact that is produced at the end of the key generation process is more akin to a **Cryptographic Identity** that contains several key pairs, alongside identity information and certifications. 

### Terminology: Keys and Identities

* An OpenPGP **Certificate** is a document that contains a cryptographic key pair. It consists of a **public key** and a **secret key**, as well as a **User ID** and a set of **Certifications** that the key belongs to the user. 
* An OpenPGP **subkey** is a cryptographic key pair consisting of a **public key** and a **secret key**, which is cryptographically bound to an OpenPGP certificate. 
* A **User ID** consists of a name and an email address, plus an optional comment. 
* An OpenPGP key or subkey can have four capabilities: 
	* it may **certify**, which means the keypair can be used to sign keys, subkeys and user IDs; 
	* it may **sign**, which means the keypair can be used to attest to and verify authorship of a document;
	* it may **encrypt**, which means the keypair can encipher and decipher a document;
	* finally, it may **authenticate**, which means the keypair can be used to prove identity via cryptographic means. 

When we speak colloquially of "generating an OpenPGP key" in the context of this system, we're actually talking about generating an **identity**, which is supported by a set of keys. 

## Key Structure

GnuPG offers many options — some obvious, some hidden — for creating identity certificates. This can easily cause information overload for someone who is not familiar with cryptographic concepts, but merely wants to secure their communications. Signet's goal in this arena is to make a standardized set of choices that balance security and usability. 

Anyone with an understanding of the GnuPG CLI can generate a certificate that adheres to this specification. This document, however, envisions most people using a basic GUI that invokes GnuPG with appropriate options to generate their cryptographic identity. 

### User ID

First and foremost, an OpenPGP identity document establishes a user's identity via a User ID, or **UID**. This traditionally follows the form of an RFC 2822 name-addr, with the user's full name followed by an email address in angle brackets, like so: 

    John Doe <jdoe1109@example.com>

When other users in the web of trust sign a user's key, they certify that they have validated the information in this UID. Such certification is not always easy to come by: either the user must personally know someone in the web of trust, or must show identifying documents to someone within the web of trust. This represents a significant barrier to scalable deployment of the public key infrastructure. 

To ameliorate this, UIDs in the Signet system do not include the user's name, but instead include only the email address: 

    <jdoe1109@example.com>

In a later section we will go into more depth about identity certification, but for now, we note that certifying a UID in this form requires certifying only that the key owner controls the email address in question; no assertion needs to be made about the user's name or personal information. 

### Key Algorithms and Sizes

Signet's goal is to perform as many cryptographic operations as possible on a smart card, in order to keep sensitive key data isolated from network-connected computers. That means one decision is made for us: we will use the RSA algorithm for public key cryptography, as it is the only algorithm currently supported by the OpenPGP smart card spec. 

Furthermore, all RSA keys under the Signet guidelines are specified to be 2048 bits in length. 2048-bit RSA is considered sufficiently strong at this time, and larger key sizes, like 4096-bit RSA, do not offer a significant enough security increase to justify significantly slower cryptographic operations. 

### Key Structure

All cryptographic identities generated under the Signet guidelines will include one key and two subkeys: 

 * One 2048-bit RSA key with signing and certification capabilities.
 * One 2048-bit RSA subkey with encryption capabilities.
 * One 2048-bit RSA subkey with authentication capabilities.

### Key Expiration

All keys are generated with a validity period of one year. At any time, the user can extend their key validity by an additional year; the one year validity merely ensures that old or disused keys expire, without forcing the user to deal with revocation certificates. 

## Storage and Usage of Key Material

The Signet guideline envisions users generating their key offline, on a removable medium such as a USB Flash drive, and copying it to a smart card for day to day use. The USB drive containing secret key material is a backup, intended to be sealed in an envelope and stored in a secure place like a locked drawer or safe deposit box. 

### Publishing Public Key Material

During the key generation process, both public key material and secret key material is generated. We will address the secret key material in a moment; but first, we need to ensure that the public key material is made available even after the removable USB media is removed. 

The automated key generation application will generate an ASCII-armored export of the public keyring at the end of the key generation process, and prompt the user for a place to save it on thier hard drive. This file can be easily imported to Gpg4win on Windows or or GPGTools on the Mac. 

### Securing Secret Key Material

During key generation, GnuPG offers an option to protect secret key material with a passphrase. Generally, a passphrase must have a lot of entropy to protect against brute force attacks; a weak passphrase is as good as no passphrase at all. The Signet guidelines call for no passphrase at all to be used on the secret key backup; instead, we direct the user to physically secure the USB drive containing their key. 

Hear me out. A passphrase is intended to be a cryptographic barrier between an attacker and the secret key material we wish to protect. If a user's key file is compromised, the argument goes, the safety of the secret key material depends on the strength of the passphrase. The Signet guidelines replace this cryptographic barrier with a physical barrier: the safety of the secret key material depends on the user's ability to secure a physical object. This is eminently understandable to the average user; they are much more likely to correctly "put it in a safe" than "choose and memorize a strong passphrase". 

In a perfect world, we would supplement the physical barrier of a locked safe with the cryptographic barrier of a passphrase. ("¿Por qué no las dos?") In practice, this presents significant usability challenges. Since the user will be interacting with a smart card for day to day cryptographic tasks, they will not utilize the passphrase often enough to commit it to memory — especially if the passphrase is complex enough to withstand brute force attack. If the user forgets the passphrase, the backup of their secret key material is rendered useless. And if we have the user store the passphrase with the backup, either on paper or on the USB drive itself, it provides no protection whatsoever. 

### Using Secret Key Material

When the key is generated, it is copied to an OpenPGP smart card. This is the user's primary cryptographic device. While the card holds on to the secret keys and can perform cryptographic math with them, it cannot divulge these secret keys. This makes it safer to use for day to day cryptographic operations. 

An OpenPGP Smart Card uses two PINs: an Admin PIN and a User PIN. During the automated setup process, a random Admin PIN is generated and stored on the same USB stick where the user's keys are generated. This Admin PIN is used behind the scenes to perform card personalization tasks; the end user doesn't even need to be aware of it. During the same process, the user is prompted to enter a User PIN. It is this PIN that the user will enter when performing day to day cryptographic tasks. 

If the user's PIN is compromised — say, via shoulder surfing — the user is able to reset the User PIN on their own using only the PIN itself. If the PIN becomes blocked — say, by entering the wrong PIN too many times — the user can use a Resetting Code, which is also set at the time of card personalization. 

## Certification and Trust

Merely generating a cryptographic identity is not enough to create a trusted public key infrastructure. As alluded to above, entry into the web of trust is one of the major stumbling blocks to wide-scale deployment of an OpenPGP-based public key cryptosystem. 

### Identity Certification

The simplified UID format — email address only — is part of Signet's solution to this problem. Once a user has generated their cryptographic identity, they can sign a message and send a request for email verification. An automated email verification service can confirm that the user in question is in control of the both the key and the email address by encrypting a verification link, and sending it to the address in the UID. 

If the key owner confirms control of the email address by clicking the link, an ultimately trusted Email Verification Authority can sign their key for a duration of either one year, or through the expiration date of the domain in the email address, whichever validity window is shorter. 

The automated key signing service is able to do this because the only information it is verifying is the email address in the UID; it makes no attestations about the user's name or legal identity, and will not sign UIDs that contain more information than a simple, valid email address. 

### Identity Lifecycle

A user's identity within the public key cryptosystem is established once their identity certificate is signed by the email verification service. Their identity remains valid until either their key or the email verification signature expires, whichever comes first. 

Validity of the key can be extended at any time, but the validity window should always remain at one year; this ensures that disused keys fall out of usage. 

Similarly, the expiration of the email verification signature can be extended at any time by submitting a signed request to the Email Verification Authority; but it will still only issue a signature valid for one year, or through the expiration date of the email address domain, whichever period is shorter. 

If the user's card is lost, they may generate a revocation certificate using the backup key on the USB stick. 

Provided that the key is not revoked, and both the self-signature and the email verification signature are not expired, the user's identity should be considered valid; if any of these conditions are not met, the user's identity is considered invalid. 

## Conclusions

The world does not need a new, proprietary cryptographic toolkit. Over the course of a quarter century, the open source community has developed a phenomenal set of open standards for cryptography, and open source implementations of those standards. Signet does not aim to invent new tools; rather, it aims to create a standardized methodology that is both simple enough to deploy widely, and secure enough to improve users' privacy and security online. 

 * Email-only UIDs make it easier to verify the identity information in an OpenPGP certificate — and map more closely to the way modern Internet users communicate. 
 * Physical protection of the key backup empowers the user to secure their secret key material in a manner consistent with their security needs. 
 * Smart cards provide users with an air gapped secure element that is capable of completing cryptographic tasks without divulging secret key material. 
 * 2048-bit RSA strikes a balance that provides both high performance and high security. 
 * A centralized email verification service provides a simpler and more scalable method for signing keys — one that does not require in-person meetings or the exchange of personally identifying information. 

I readily admit that this looks different from the way that OpenPGP has been deployed in the past. It makes tradeoffs to achieve its goals of simplifying what has been considered a daunting set of tools. If those tradeoffs lead to more wide-scale usage and understanding of OpenPGP-based tools, this project will be a success. 
