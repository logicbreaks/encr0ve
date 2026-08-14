# Encr0ve — Diagrams

## 1. Message Flow (Sequence Diagram)

```mermaid
sequenceDiagram
    participant Alice as Alice (encr0ve)
    participant AKey as Alice's Keychain
    participant Chat as WhatsApp / iMessage
    participant Bob as Bob (encr0ve)
    participant BKey as Bob's Keychain

    Note over Alice, BKey: 0. Key Exchange (in person / QR)
    Alice->>AKey: Generate X25519 keypair
    Bob->>BKey: Generate X25519 keypair
    Alice->>Bob: Scan Bob's QR (public key)
    Bob->>Alice: Scan Alice's QR (public key)
    Note over Alice, BKey: Keys stored locally

    Note over Alice, BKey: 1. Encrypt
    Alice->>Alice: Type plaintext message
    Alice->>AKey: Fetch Bob's public key
    Alice->>Alice: Encrypt: X25519 + XChaCha20-Poly1305
    Alice->>Alice: Armor: base64-encode ciphertext
    Alice->>Alice: Copy to clipboard

    Note over Alice, BKey: 2. Send over chat
    Alice->>Chat: Paste blob, send
    Chat->>Bob: Deliver blob

    Note over Alice, BKey: 3. Decrypt
    Bob->>Bob: Copy blob from chat
    Bob->>Bob: Paste into encr0ve
    Bob->>BKey: Fetch own private key
    Bob->>Bob: Decrypt & verify signature
    Bob->>Bob: Display plaintext
```

---

## 2. Trust Boundary Diagram

```mermaid
flowchart TB
    subgraph TCB[Trusted Computing Base - User's Device]
        direction TB
        subgraph Enc[encr0ve App]
            E1[Encrypt/Decrypt Engine]
            E2[Key Manager]
            E3[QR Code Scanner]
            E4[Clipboard Manager]
        end
        subgraph iOS[iOS Platform]
            KC[iOS Keychain - Keys]
            SE[Secure Enclave]
            SB[Sandbox]
        end
        E1 --- KC
        E2 --- KC
        E2 --- SE
    end

    subgraph Boundary[Trust Boundary]
        CB[Clipboard - armored blob]
    end

    subgraph Untrusted[Untrusted Surface]
        direction TB
        subgraph Chat[Chat Apps]
            WA[WhatsApp]
            IM[iMessage]
            TG[Telegram]
        end
        subgraph Network[Network]
            CH[Chat Servers]
            ISP[ISP / Carrier]
        end
    end

    Enc -->|armored blob| CB
    CB -->|paste| Chat
    Chat -->|transit| Network
    Network -->|deliver| Chat
    Chat -->|copy| CB
    CB -->|paste| Enc

    style TCB fill:#1a3a1a,stroke:#4caf50,color:#fff
    style Boundary fill:#333,stroke:#ff9800,color:#fff,stroke-dasharray: 5 5
    style Untrusted fill:#3a1a1a,stroke:#f44336,color:#fff
```

---

## 3. Key Management Lifecycle

```mermaid
stateDiagram-v2
    [*] --> FirstLaunch
    FirstLaunch --> GenerateKeypair: First run
    GenerateKeypair --> KeyInKeychain: Store securely
    KeyInKeychain --> ShareQR: User taps "Share my key"
    ShareQR --> KeyInKeychain: QR displayed

    KeyInKeychain --> ScanContactQR: User taps "Add contact"
    ScanContactQR --> ContactAdded: QR scanned successfully
    ContactAdded --> KeyInKeychain: Public key stored

    KeyInKeychain --> KeyLost: User lost phone / deleted app
    KeyLost --> VerifyRecovery: Enter recovery phrase
    VerifyRecovery --> RegenerateKeypair: Phrase valid
    RegenerateKeypair --> KeyInKeychain: New keypair

    KeyInKeychain --> KeyCompromised: Suspect key leaked
    KeyCompromised --> GenerateKeypair: Revoke and regenerate
    GenerateKeypair --> ShareQR: Share new key with contacts

    KeyInKeychain --> ExportRecovery: User requests backup
    ExportRecovery --> RecoveryPhrase: Show 12-word seed
    RecoveryPhrase --> KeyInKeychain: User stored offline
```

---

## 4. Deployment / System Architecture

```mermaid
flowchart LR
    subgraph iOS[Apple iOS Device]
        direction TB
        SW[SwiftUI App]
        Crypto[Swift-Sodium Lib]
        QC[AVFoundation - QR Scanner]
        KC[Keychain Services]
        Paste[UIPasteboard]
    end

    subgraph External[No Server-Side Infrastructure]
        GH[GitHub - Source Code]
        Test[Test Matrix - Real Device Farm]
    end

    SW --> Crypto
    SW --> QC
    SW --> KC
    SW --> Paste
    GH -.->|User builds / audits| SW

    style iOS fill:#1a1a2e,stroke:#4361ee,color:#fff
    style External fill:#2d1a2e,stroke:#e94560,color:#fff
```

---

## 5. Encryption & Decryption Flow

```mermaid
flowchart LR
    subgraph Encrypt[Encryption]
        PT[Plaintext] --> KX[Key Exchange: X25519]
        KX --> Shared[Shared Secret]
        Shared --> Sym[Encrypt: XChaCha20-Poly1305]
        Sym --> CT[Ciphertext]
        CT --> Armor[Base64 Armor]
        Armor --> Out[Armored Blob]
        Sig[Ed25519 Sign] -.->|optional| Out
    end

    subgraph Decrypt[Decryption]
        In[Armored Blob] --> DeArmor[Base64 Decode]
        DeArmor --> CT2[Ciphertext]
        CT2 --> Sym2[Decrypt: XChaCha20-Poly1305]
        Sym2 --> PT2[Plaintext]
        Ver[Ed25519 Verify] -.->|optional| PT2
    end

    Out -->|copy-paste through chat| In
```