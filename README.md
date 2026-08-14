# encr0ve

**Encrypt before it ever touches the app.** A user-friendly pre-encryption layer for iOS — type your message, encr0ve turns it into an opaque armored blob, you paste it into WhatsApp/iMessage/Telegram like a normal message. Your chat app — and everyone compelled to look inside it — only ever sees ciphertext.

> Status: **Phase 0 — threat modeling & research** (Aug 2026). No code yet.

---

## Why

In 2026 the EU is negotiating the Child Sexual Abuse Regulation ("Chat Control"): detection orders that can force messaging platforms to scan content. Even where end-to-end encryption survives, the pressure (and the problem) is to put the "listening end" *inside* the app — encryption you can't verify and scanning you can't opt out of.

encr0ve's answer is simple and structural: **the app is not the end.** Encryption happens in your own tool, before the message enters any messaging client. The platform gets a blob it cannot read, and there is nothing left to scan.

This is not anti-government nor pro-criminal. It is the position that a private chat between two people is nobody else's business — and that your belongings (your phone) shouldn't be forced to carry someone else's spyware, no matter how powerful they are.

## What it does

```
Alice types "see you at 8" ──► encr0ve encrypts ──► [opaque blob] ------>> *"https://en.wikipedia.org/wiki/Opaque_binary_blob"*
                                                      │ paste
                                                      ▼
                                     WhatsApp / iMessage / Telegram
                                                      │
Alice's chat app, its servers, the network ── all see:  [opaque blob]
                                                      │ copy
                                                      ▼
Bob pastes blob into encr0ve ──► decrypts ──► "see you at 8"
```

- **X25519 + XChaCha20-Poly1305** (libsodium / swift-sodium) — audited, modern, no hand-rolled crypto
- **QR-code key exchange** — scan a friend's screen, done. No fingerprint-typing, no .asc files
- **Keychain storage + recovery code** — careless-user-proof by design
- **Fully open source** — trust through transparency; anyone can build and audit it

More graphs and visualizations: ([Visualizations](01-threat-model))

## What it does NOT do (honest boundaries)

| Limitation | Why |
|---|---|
| ❌ No (full) metadata protection | The chat app still sees *who you talk to, when, how often*. encr0ve only hides *content*. |
| ❌ No forward secrecy | Asynchronous copy-paste can't ratchet keys like Signal. If a key leaks, past messages leak. |
| ❌ No defense against a compromised device | If your OS is owned (spyware), the "end" is owned. Out of scope. |
| ❌ Not a replacement for Signal | Signal is still the right tool for most people. encr0ve is for staying in the apps you already use. |
> Note: Some/Many of those already have potential solutions its just made clear that until these warnings vanish, nothing is prooven about any kind of security promise.

## Roadmap

| Phase | Milestone | Status |
|-------|-----------|--------|
| **0** | Research + threat model + diagrams | ... In progress |
| **1** | Crypto core + CLI (`enc "text"` / `dec "blob"`) — prove round-trips | ⬜ |
| **2** | iOS app (or more probable, some workaround), copy-paste MVP: QR exchange, contact list, keychain | ⬜ |
| **3** | Friction removal: share extension, auto-detect blobs, iCloud sync, recovery flow | ⬜ |
| **4** | Custom keyboard (encrypt-as-you-type) — *requires open access; research says: nice-to-have* | ⬜ |
| **5** | Open-source hardening: publish, independent audit, threat-model v1.0 | ⬜ |

## Repo layout

```
encr0ve/
├── 00-research/        # fact-checks with sources (EU status, iMessage, libsodium, iOS limits)
├── 01-threat-model/    # threat-model.md + diagrams.md (Mermaid)
├── 02-design/          # crypto design, UX flows (next)
├── 03-prototype/       # Phase-1 CLI lib (next)
└── README.md
```

## Key research findings so far

- **EU Chat Control:** permanent regulation still in trilogue; the July 2026 derogation reinstatement *excludes E2EE services*. Premise shifts from "legal necessity" to "defense-in-depth + user control" — still valuable. ([research](00-research/factcheck-research.md))
- **iMessage:** E2E is RSA-based (no forward secrecy, no contact verification). encr0ve would offer *stronger* crypto than iMessage's native E2E.
- **iOS keyboards:** no clipboard access, ever; shared container requires open access. Copy-paste app is the MVP, keyboard is Phase 4.

> You might spot signs of AI usage in this Project, this is not denied. Tho, be asured, this technology will not be used in any coding process except as watchdogs to make sure theres no obvious problems. Any code snippet will be humanly written as this is to much of a high responsibility project to just hand off.
---

*encr0ve — nobody reads your messages but you.*
