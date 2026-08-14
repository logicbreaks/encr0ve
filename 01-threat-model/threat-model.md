# Encr0ve — Threat Model

> Version: v0.1 (August 2026)
> Scope: Pre-encryption layer for iOS users messaging over existing chat apps

---

## 1. Scope

**What encr0ve is:**
A user-friendly iOS app that encrypts plaintext messages before they enter any messaging client (WhatsApp, iMessage, Telegram, Signal). The resulting ciphertext (armored ASCII) is copy-pasted into the user's preferred chat app. The recipient decrypts by pasting the ciphertext back into encr0ve.

**What encr0ve is NOT:**
- A replacement for end-to-end encrypted messaging apps
- A metadata-hiding tool
- A secure communication channel (it relies on the transport channel for delivery)
- A device compromise solution
- A tool for hiding who you talk to or when

**Target users:**
- iOS users who want payload confidentiality beyond what their messaging app provides
- Non-technical users who should not need to understand keys, algorithms, or infrastructure
- Users who want open-source verifiable security without trusting a third-party server

---

## 2. Assets

| Asset | Sensitivity | Notes |
|-------|-------------|-------|
| Plaintext message (created by user) | **CRITICAL** | Only exists in encr0ve's memory, never persisted to disk in plaintext |
| Private key (Curve25519) | **CRITICAL** | Stored in iOS Keychain; never leaves the device |
| Contact's public keys | **HIGH** | Stored in encr0ve's local encrypted database |
| Ciphertext (armored blob) | **LOW** | Opaque to anyone without the private key; visible to the messaging app, its servers, and the network |
| Message metadata (who, when, size) | **LOW** | Fully visible to the messaging platform — encr0ve provides no metadata protection |
| Recovery code / seed phrase | **CRITICAL** | Used to regenerate keys; user must store offline (paper, password manager) |

---

## 3. Trust Boundaries

```
┌─────────────────────────────────────────────────────┐
│                 TRUSTED COMPUTING BASE               │
│  ┌─────────────────────────────────────────────────┐│
│  │  USER'S iOS DEVICE                              ││
│  │  ┌──────────────────┐  ┌────────────────────┐   ││
│  │  │  encr0ve app     │  │  iOS Keychain       │   ││
│  │  │  (encrypt/decrypt)│  │  (private key store) │   ││
│  │  └──────────────────┘  └────────────────────┘   ││
│  │  ┌────────────────────────────────────────────┐  ││
│  │  │  OS / Hardware (iOS sandbox, Secure Enclave)│  ││
│  │  └────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────┘
                        │
      ciphertext blob (copy-paste across boundary)
                        ▼
┌─────────────────────────────────────────────────────┐
│              UNTRUSTED SURFACE                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Messaging app │  │ Chat servers │  │ Network    │ │
│  │ (WhatsApp,    │──│ (Meta, Apple,│──│ (ISP,      │ │
│  │  iMessage,    │  │  Telegram)   │  │  carrier)  │ │
│  │  Telegram)    │  │              │  │            │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────┘
```

**Key boundary rule:** The only data crossing the trust boundary is the armored ciphertext blob. The messaging app, its servers, and the network never see plaintext or keys.

---

## 4. Threat Actors

| Actor | Motivation | Capability |
|-------|-----------|------------|
| **A1 — Messaging platform** (WhatsApp/Meta, Apple, Telegram) | Compliance with legal scanning mandates, internal policy enforcement | Full access to the messaging app's servers, client-side code, metadata |
| **A2 — Network observer** (ISP, carrier, VPN provider) | Traffic analysis, legal intercept | Sees encrypted traffic, metadata, timing |
| **A3 — Government / law enforcement** (EU Chat Control, §) | Mass surveillance, targeted intercept | Can compel the platform to add scanning, can serve demands for metadata |
| **A4 — Adversarial contact** | Impersonation, eavesdropping | Can try to intercept key exchange, send fake public keys |
| **A5 — Malicious app on device** | Data theft | Can read clipboard, screen, accessibility data |
| **A6 — The careless user themselves** | Key loss, accidental plaintext exposure | The most common "threat" in practice |

---

## 5. Threat List

| ID | Scenario | Likelihood | Impact | Mitigation | Status |
|----|----------|-----------|--------|------------|--------|
| **T1** | Platform scans plaintext pre-E2E (client-side scanning) | **HIGH** (EU Chat Control trajectory) | **CRITICAL** — plaintext exposed | Pre-encryption: platform only sees opaque armored blob | ✅ Mitigated |
| **T2** | Platform reads ciphertext from its servers | **CERTAIN** — they store everything | **LOW** — ciphertext is opaque | Strong encryption (XChaCha20-Poly1305) | ✅ Mitigated |
| **T3** | Metadata collection (who talks to whom, when, how often) | **CERTAIN** — platforms do this | **MEDIUM** — reveals relationships | **No mitigation provided by encr0ve** — clear limitation | ⚠️ Documented |
| **T4** | Ciphertext mangled by chat app (reflow, encoding, truncation) | **MEDIUM** — varies by platform | **HIGH** — message becomes undecryptable | Armored ASCII format, round-trip testing against real apps, error detection | 🏗️ Design phase |
| **T5** | Man-in-the-middle on key exchange | **LOW** if QR is used in person | **CRITICAL** — attacker can decrypt | QR code exchange on trusted physical channel (in-person) + safety numbers for out-of-band verification | 🏗️ Design phase |
| **T6** | Key loss (user deletes app, loses phone) | **HIGH** — most common failure | **HIGH** — all past messages unreadable | Recovery code / seed phrase backup, iCloud Keychain sync | 🏗️ Design phase |
| **T7** | Attacker gains physical access to unlocked device | **MEDIUM** | **CRITICAL** — all keys exposed | iOS Keychain protection (kSecAttrAccessibleWhenUnlockedThisDeviceOnly), app lock | 🏗️ Design phase |
| **T8** | Compromised device (malware, spyware like Pegasus) | **LOW** for average user, **HIGH** for targeted | **CRITICAL** — entire TCB compromised | **Out of scope** — encr0ve cannot protect against OS-level compromise | ⚠️ Out of scope |
| **T9** | Clipboard snooping (other apps reading clipboard) | **MEDIUM** — iOS 14+ shows clipboard read indicator | **MEDIUM** — plaintext briefly on clipboard | Auto-clear clipboard after X seconds, warn user, minimize time plaintext is on clipboard | 🏗️ Design phase |
| **T10** | Social engineering / fake public key | **MEDIUM** | **HIGH** — attacker can decrypt | TOFU (trust on first use) with key change detection and warnings (Signal-style) | 🏗️ Design phase |
| **T11** | Forward secrecy violation | **ALWAYS** — this is a design tradeoff | **MEDIUM** — if key is compromised, ALL past messages readable | **No mitigation** — pre-encryption over async messaging cannot provide forward secrecy | ⚠️ Tradeoff documented |
| **T12** | App Store review / Apple compelled to remove or backdoor app | **LOW** — but non-zero | **CRITICAL** — app becomes unavailable | Open source → anyone can self-build; distribute via alternative channels possible | 🏗️ Design phase |

---

## 6. Design Decisions & Trade-offs

### 6.1 Deliberate Absence of Forward Secrecy
Forward secrecy (FS) requires both parties to be online simultaneously to ratchet keys (Signal's model). encr0ve is a pre-encryption tool for *asynchronous* messaging — the sender encrypts now, the receiver decrypts later. FS is structurally incompatible with this model. **Accepted trade-off.**

### 6.2 Metadata Exposure
encr0ve does not hide who you talk to, when, or how often. This is a fundamental limitation of the "overlay on existing messaging" model. The platform sees everything about the *communication channel* except the content. **Accepted trade-off, clearly documented.**

### 6.3 Copy-Paste UX
The clipboard is the weakest link in the security chain. iOS 14+ shows a clipboard-read indicator, which helps. encr0ve will add auto-clear and timeout features. **The trade-off is UX simplicity vs. clipboard exposure** — the clipboard model is chosen because it works with every messaging app without requiring special permissions.

### 6.4 Key Verification via QR Codes
encr0ve uses QR code scanning for key exchange. This is secure when done in person (trusted visual channel). For remote verification, safety numbers allow out-of-band verification (Signal-style). **Attack surface:** if the QR is sent over the same messaging channel being protected, a MITM could swap keys. Mitigation: always verify safety numbers.

### 6.5 Crypto Algorithm Choice
**Primary:** X25519 (key agreement) + XChaCha20-Poly1305 (encryption) + Ed25519 (signatures) via libsodium.
**Rationale:**
- X25519 is the current gold standard for key agreement
- XChaCha20-Poly1305 is authenticated encryption with extended nonce (safe for random nonces)
- Ed25519 provides message signatures for sender verification
- All are NIST/CFRG recommended, audited, side-channel resistant
- All are available in swift-sodium (production-ready)

### 6.6 Open Source & Auditable
encr0ve is fully open source. Anyone can:
- Inspect the code
- Verify the crypto implementation
- Build from source
- Audit the binary against the source

This is the primary defense against T12 (compelled backdoor).

---

## 7. Threat Model Revision History

| Version | Date | Changes |
|---------|------|---------|
| v0.1 | 14 Aug 2026 | Initial draft — all threats identified, some mitigations in design phase |