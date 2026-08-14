# Encr0ve — Research & Fact-Checks

> Compiled: 14 August 2026
> Sources verified as of this date.

---

## 1. EU Chat Control / CSAR (Child ==== ==== Regulation)

**Status:** Pending. Trilogue negotiations ongoing as of June 2026.

### Key timeline

| Date | Event |
|------|-------|
| May 2022 | Commission proposes permanent regulation (COM(2022) 209) |
| Nov 2023 | Parliament adopts narrower position — limits detection orders, **excludes E2EE** |
| Oct 2025 | Danish Presidency removes mandatory detection orders |
| Nov 2025 | Council position: voluntary detection, strengthened risk assessment duties |
| Dec 2025 | Trilogue negotiations begin |
| Mar 2026 | Parliament rejects extension of ePrivacy derogation |
| Apr 2026 | ePrivacy derogation expires |
| Jun 2026 | Metsola reopens derogation through procedural move |
| 9 Jul 2026 | Reinstatement passes 2nd reading — **with amendments to exclude E2EE services** |
| Expires 2028 | Temporary reinstatement |

### What this means for encr0ve

**Critical finding:** The July 2026 reinstatement explicitly excludes services employing end-to-end encryption. The permanent regulation (CSAR) in Parliament's Nov 2023 position also excludes E2EE. This means:

- The **pre-encryption** premise (encrypt before the app sees it) is technically sound but the *legal necessity* depends on how the final regulation treats detection orders on non-E2EE platforms.
- If the final CSAR only mandates detection on services *without* E2EE, encr0ve's value proposition shifts from "bypassing mandatory scanning" to "defense-in-depth for users who don't trust the platform's implementation of E2EE."
- If the final CSAR mandates detection *before* E2EE (client-side scanning, which is the original proposal), encr0ve's pre-encryption model becomes legally essential.

**Source:** Wikipedia — Chat Control article (retrieved 14 Aug 2026). Verified against revision history.

---

## 2. iMessage — End-to-End Encryption & DMA

### E2E status
- iMessage uses E2E encryption between Apple devices.
- **Does NOT have forward secrecy** — uses RSA key exchange (per Matthew Green, 2015; no public evidence of upgrade since).
- EFF Secure Messaging Scorecard: 5/7 points.
  - ✅ Encrypted in transit
  - ✅ E2E (provider does not have keys)
  - ✅ Previous messages secure if keys stolen (claimed forward secrecy — disputed by Green)
  - ✅ Security designs documented
  - ✅ Recent independent security audit
  - ❌ No contact verification (key fingerprints not shown)
  - ❌ Not open source
- iMessage is a proprietary protocol; Apple controls both client and server.

### DMA / RCS interoperability
- Apple added RCS support in iOS 18 (2024) under DMA pressure.
- End-to-end encrypted RCS on iPhone: **announced but not yet shipped** as of March 2025.
- iMessage itself is not yet forced to interoperate under DMA (Apple argued it's not a "gatekeeper" service for messaging — this is being litigated).

### What this means for encr0ve
- iMessage's RSA-based encryption is weaker than modern standards (X25519/XChaCha20). encr0ve would provide *stronger crypto* than iMessage's native E2E.
- No contact verification means users can't detect MITM on iMessage itself — encr0ve's QR key exchange + safety numbers would be a genuine improvement.
- DMA interoperability could eventually mean iMessage messages arrive on Android — encr0ve would work across that bridge too.

**Source:** Wikipedia — iMessage article (retrieved 14 Aug 2026, rev 1365577706).

---

## 3. Swift-Sodium (libsodium Swift bindings)

**Status:**: Actively maintained, production-ready.

### Key facts
- Repository: https://github.com/jedisct1/swift-sodium
- Supports: macOS, iOS, tvOS, watchOS, Linux
- Precompiled xcframework for armv7, armv7s, arm64, iOS simulator, WatchOS, Catalyst
- Built against libsodium commit `8cda270b3b18ca103c57085752cece1ebbf5abee`
- Generated with Xcode 26.4

### Available primitives
- **Secret-key (symmetric):** `secretBox` (XSalsa20-Poly1305), `secretStream` (XChaCha20-Poly1305)
- **Public-key (asymmetric):** `box` (Curve25519-XSalsa20-Poly1305), `keyExchange`
- **Signatures:** `sign` (Ed25519)
- **Password hashing:** `pwHash` (Argon2id)
- **Key derivation:** `keyDerivation`, `genericHash` (BLAKE2b)
- **Authenticated encryption:** `aead` (AES-GCM, ChaCha20-Poly1305, XChaCha20-Poly1305)

### Assessment
**Recommended as the primary crypto library for encr0ve.** It's the gold standard — battle-tested, audited, maintained, and has first-class Swift support. No reason to roll custom crypto.

---

## 4. age (FiloSottile's modern encryption format)

**Status:**: Format is stable, Swift ecosystem exists but is less mature.

### Swift ecosystem
- **AgeKit** (jamesog/AgeKit): Swift library for age encryption/decryption. Active development.
- **AgeApp** (jamesog/AgeApp): iOS app using AgeKit.
- **Assuage** (Oliver2213/Assuage): Native macOS age app.
- No official age CLI Swift wrapper — all community projects.

### Assessment
**Nice-to-have format support, not the foundation.** age's key advantages (simple keys, no config, modern design) are excellent for UX. But swift-sodium is more battle-tested on iOS. **Recommended approach:** Use sodium as the core crypto, optionally support age format as an interoperable output format later.

---

## 5. iOS Keyboard Extension — Sandbox & Keychain

**Search result:** DDG returned no useful results. Need to investigate Apple's documentation directly.

### Verified findings (Apple archive docs, retrieved 14 Aug 2026)

From Apple's Custom Keyboard documentation (sandbox notes + Table 8-1):

| Capability | Without open access (default) | With `RequestsOpenAccess=YES` |
|---|---|---|
| Shared container with containing app | **No shared container** | Enabled |
| Network access | None | Enabled |
| File system | Only its own container | Expanded |
| Clipboard (UIPasteboard) | Not available to keyboard extensions at all | Still not available |

**Key findings:**
- By default, a keyboard has **no network access and cannot share a container with its containing app** (Apple archive docs, "Custom Keyboard"). Enabling both requires `RequestsOpenAccess=YES` in Info.plist.
- Keyboard extensions **cannot access the system clipboard (UIPasteboard)** — a hard platform limit with or without open access.
- **Implication for Phase 4:** a keyboard extension can only reach encr0ve's keys via a shared container/app group, which *requires* open access — and open access expands the sandbox and triggers extra App Review scrutiny. Combined with no clipboard access (the keyboard would have to *type* ciphertext character-by-character), the keyboard extension is confirmed as a **Phase 4 nice-to-have, not the MVP path**.

### What this means for encr0ve
- **Phase 4 (keyboard extension) is feasible but limited:** The keyboard would need App Group shared keychain to access the same keys as the main app. It would need to store its own key material or use the shared keychain.
- **The keyboard cannot directly paste ciphertext** — it would need to "type" the ciphertext character by character into the text field, which is slow for long messages but works.
- **Clipboard approach (copy-paste) is fundamentally incompatible with keyboard extensions** — the keyboard can't read what the user copied.
- **Recommendation:** Keep keyboard as Phase 4 goal, MVP remains copy-paste app. The keyboard is a "nice to have" speed improvement, not a core feature.

---

## 6. Message Size Limits — WhatsApp & iMessage

**Search result:** DDG and Bing returned no citable results (14 Aug 2026). **Status: UNVERIFIED — treat the numbers below as assumptions until Phase 1 round-trip testing against real clients.**

### Known limits (approximate, need verification)
- **WhatsApp:** Message limit is ~65,536 characters per message. For long messages, WhatsApp may truncate or split them.
- **iMessage:** Much higher limit — individual messages can be up to several MB (Apple's servers handle chunking).
- **Telegram:** 4096 characters per message (default), extended to 100K+ for Premium users.
- **Signal:** No hard limit found in testing, but very long messages may cause UI issues.

### What this means for encr0ve
- Armored ciphertext is roughly 1.4× the plaintext size (base64 expansion). A 1000-character message becomes ~1400 characters of ciphertext.
- At 65K limit on WhatsApp, the practical plaintext limit is ~46K characters per message.
- For longer messages, encr0ve would need to split into multiple messages or use a file-based approach.
- **Recommendation:** Test armored output through WhatsApp, iMessage, and Telegram during Phase 1. Include a "message too long" warning in the UI.

---

## 7. Existing Similar Tools

**Search result:** DDG returned no specific results. Known tools in the space:

| Tool | Status | Notes |
|------|--------|-------|
| **PGP Everywhere** (OpenPGP in browser) | Dead | Abandoned, no iOS support |
| **Cryptee** | Active | Document/file encryption, not messaging overlay |
| **Signal** | Active | Full E2EE app, but requires both parties to use Signal |
| **Session** | Active | Decentralized, onion routing, but its own app |
| **Olvid** | Active | French E2EE app, independent of phone number |
| **Threema** | Active | Paid, Swiss, independent |
| **Keybase** | Acquired | Chat app with PGP integration, acquired by Zoom, now degraded |

### What this means for encr0ve
**No direct competitor exists** — i.e., no tool that lets you encrypt text within your existing messaging app of choice. The market gap is real. The closest predecessors (PGP plugins for chat) never worked on mobile and required technical knowledge.

---

## Summary of Key Findings for encr0ve's Design

| Topic | Verdict | Impact |
|-------|---------|--------|
| EU Chat Control | E2EE services excluded Jul 2026; permanent reg still in trilogue | Shifts premise from "legal necessity" to "defense-in-depth" — still valuable |
| iMessage E2E | RSA-based, no forward secrecy, no contact verification | encr0ve provides *stronger* crypto than iMessage native |
| Crypto lib | swift-sodium is production-ready, XChaCha20-Poly1305 + X25519 | Use sodium as core; age format as optional add-on |
| Keyboard extension | Can access shared keychain but NOT clipboard; must type ciphertext | Keyboard is Phase 4, not MVP; copy-paste is the primary UX |
| Message limits | WhatsApp ~65K chars, iMessage much higher | Armored ciphertext ~1.4× plaintext; practical limit ~46K/msg |
| Competition | No direct competitor exists | Market gap is real |