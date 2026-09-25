# Velvet Veil — Private Messaging & Post-Quantum Encrypted Communication

> **Legal & Compliance Notice**:
> Velvet Veil is an independent private messaging software project implementing end-to-end encryption based on the Signal Protocol (including the PQXDH post-quantum hybrid key exchange specification).
> **Velvet Veil is NOT affiliated with, sponsored by, or endorsed by the Signal Technology Foundation or Signal Messenger LLC.**
> This project operates its own independent infrastructure and servers, and does not connect to signal.org network servers.
> All code is licensed under the **GNU Affero General Public License v3.0 (AGPLv3)**.

---

## Overview

Velvet Veil is a quiet, private corner of the internet for two people in love. It pairs WhatsApp-level usability with Signal-grade privacy and Post-Quantum cryptography.

- **Philosophy**: *"Privacy is not hiding. It’s choosing who gets in."*
- **Security Priority**: `SECURITY > CORRECTNESS > PRIVACY > USABILITY > PERFORMANCE`
- **Crypto Core**: Post-Quantum Extended Diffie-Hellman (**PQXDH**) combining **X25519** + **ML-KEM-1024 (Kyber-1024)** + **HKDF-SHA256** + **Double Ratchet** + **AES-256-GCM**.
- **Blind Relay**: The server is a zero-knowledge store-and-forward relay. The server never holds private keys and cannot decrypt message envelopes.

---

## Monorepo Architecture

```
project24/
├── web/                       # Velvet Veil React Landing Page (PDF Spec)
│   ├── src/components/        # Editorial typography, Rose Quartz aura layers, chat preview, modal
│   └── src/index.css          # Rose Quartz CSS aura tokens, animations, responsive design
├── server/                    # Minimal Blind Relay API
│   ├── src/routes/auth.ts     # OTP request & verification (JWT, rate-limiting)
│   ├── src/routes/keys.ts     # Pre-key bundle management (ML-KEM-1024 + X25519)
│   ├── src/routes/messages.ts # Sealed envelope store & forward
│   └── src/db/schema.sql      # PostgreSQL 15 schema
├── crypto/                    # PQXDH Cryptographic Suite & Handshake Runner
│   └── src/pqxdh_handshake.ts # Executable PQXDH verification test (PQXDH: ACTIVE)
├── .gitignore                 # Strict secret scanning & leak prevention rules
├── .env.example               # Template environment variables
├── LICENSE                    # AGPLv3 License
└── README.md
```

---

## Quick Start

### Prerequisites
- Node.js 20+
- pnpm 9+ (`corepack enable`)

### 1. Install Dependencies
```bash
corepack pnpm install
```

### 2. Verify Cryptographic PQXDH Handshake
Run the post-quantum hybrid key exchange test:
```bash
corepack pnpm --filter crypto test
```
Expected output:
```
[SECURITY AUDIT] PQXDH: ACTIVE
```

### 3. Run the Blind Relay Server
```bash
corepack pnpm --filter server dev
```
Server listens on `http://localhost:3000`.

### 4. Run the Velvet Veil Landing Page
```bash
corepack pnpm --filter web dev
```
Open `http://localhost:5173` to explore the landing page.

---

## Cryptographic Design (PQXDH Flow)

1. **Key Generation**:
   - Identity Key Pair (Ed25519 / X25519)
   - Signed PreKey (X25519) + Identity signature
   - Post-Quantum Last-Resort PreKey (ML-KEM-1024 / Kyber-1024, ~1568 bytes)
   - Pool of Post-Quantum One-Time PreKeys (ML-KEM-1024)
2. **Key Upload**: Only public keys are registered with `POST /v1/keys/upload`. Private keys are sealed in device secure storage (Android Keystore / iOS Keychain).
3. **Session Establishment**: Initiator fetches recipient's public pre-key bundle via `GET /v1/keys/:userId` and computes the PQXDH hybrid secret combining both classical ECDH and post-quantum KEM encapsulation.
4. **Double Ratchet**: Messages advance the ratchet per transmission, guaranteeing forward secrecy and break-in recovery.
5. **Sealed Sender Relay**: Envelopes sent to `POST /v1/messages/send` are opaque ciphertexts; the relay server has no visibility into message content or sender identities.

---

## License

GNU Affero General Public License v3.0 (AGPLv3). See [LICENSE](./LICENSE) for terms.
