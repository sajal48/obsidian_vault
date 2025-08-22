---
connections: []
reference: 
tags:
  - documentation_note
  - cryptography
  - security
  - messaging
type: documentation_note
created: 2025-08-22 09:27
---

# Signal Protocol Documentation

## Overview
The Signal Protocol is an end-to-end encryption protocol used by Signal, WhatsApp, and other messaging applications. It provides forward secrecy and post-compromise security through a combination of cryptographic primitives.

## Key Types in Signal Protocol

### 1. Identity Keys
- **Purpose**: Long-term identity verification
- **Type**: Ed25519 key pair
- **Lifetime**: Permanent (unless device reset)
- **Function**: Used to sign prekeys and verify identity

### 2. Prekeys
- **Signed Prekey**: 
  - Curve25519 key pair
  - Signed by identity key
  - Rotated periodically (weekly)
  - Provides forward secrecy
  
- **One-time Prekeys**:
  - Curve25519 key pairs
  - Unsigned
  - Single-use only
  - Deleted after use

### 3. Ephemeral Keys
- **Purpose**: Session establishment
- **Type**: Curve25519 key pair
- **Lifetime**: Single message exchange
- **Function**: Part of X3DH key agreement

### 4. Chain Keys and Message Keys
- **Root Key**: Derives new chain keys
- **Chain Key**: Derives message keys and next chain key
- **Message Key**: Encrypts individual messages
- **Ratchet Keys**: Drive the Double Ratchet forward

## Message Flow: Device A → Server → Device B

### Phase 1: Initial Setup (Registration)

```
Device A                    Server                     Device B
   |                          |                          |
   |--- Upload Identity Key ---|                         |
   |--- Upload Signed Prekey --|                         |
   |--- Upload One-time Keys --|                         |
   |                          |                          |
   |                          |--- Store Keys Bundle ---|
   |                          |                          |
   |                          |--- Upload Identity Key ---|
   |                          |--- Upload Signed Prekey --|
   |                          |--- Upload One-time Keys --|
```

### Phase 2: Key Exchange (X3DH - Extended Triple Diffie-Hellman)

```
Device A                    Server                     Device B
   |                          |                          |
   |--- Request B's Keys -----|                         |
   |<-- B's Key Bundle -------|                         |
   |                          |                          |
   | Generate Ephemeral Key   |                          |
   | Perform X3DH            |                          |
   | Derive Shared Secret    |                          |
   |                          |                          |
```

#### X3DH Calculation:
1. **DH1** = DH(IK_A, SPK_B) - Identity Key A × Signed Prekey B
2. **DH2** = DH(EK_A, IK_B) - Ephemeral Key A × Identity Key B  
3. **DH3** = DH(EK_A, SPK_B) - Ephemeral Key A × Signed Prekey B
4. **DH4** = DH(EK_A, OPK_B) - Ephemeral Key A × One-time Prekey B (if available)

**Shared Secret** = KDF(DH1 || DH2 || DH3 || DH4)

### Phase 3: Message Encryption and Transmission

```
Device A                    Server                     Device B
   |                          |                          |
   | Encrypt with Double     |                          |
   | Ratchet Algorithm       |                          |
   |                          |                          |
   |--- Encrypted Message ----|                         |
   |                          |--- Forward Message -----|
   |                          |                          | Decrypt with Double
   |                          |                          | Ratchet Algorithm
   |                          |                          |
   |                          |<--- Delivery Receipt ----|
   |<--- Delivery Receipt ----|                          |
```

## Double Ratchet Algorithm

### Key Components

1. **DH Ratchet**: 
   - Updates encryption keys with each message
   - Provides forward secrecy
   - Uses new ephemeral key pairs

2. **Symmetric Ratchet**:
   - Derives new message keys
   - Prevents key reuse
   - Ensures unique keys per message

### Ratchet State

```
State = {
    DHs,         // DH sending key pair
    DHr,         // DH receiving public key  
    RK,          // Root key
    CKs,         // Chain key for sending
    CKr,         // Chain key for receiving
    Ns,          // Message number for sending
    Nr,          // Message number for receiving
    PN,          // Previous chain length
    MKSKIPPED    // Skipped message keys
}
```

## Security Properties

### Forward Secrecy
- Old messages remain secure even if current keys are compromised
- Achieved through key deletion after use
- Both DH and symmetric ratchets contribute

### Post-Compromise Security  
- Future messages are secure even after key compromise
- New DH key pairs heal from compromise
- Automatic key rotation provides recovery

### Deniability
- Messages cannot be cryptographically proven to originate from sender
- Shared MAC keys prevent non-repudiation
- Important for plausible deniability

## Message Format

```
EncryptedMessage = {
    header: {
        dh_public_key,    // Current DH public key
        previous_counter, // Number of messages in previous chain
        message_counter   // Position in current chain
    },
    ciphertext,          // Encrypted message content
    mac                  // Message authentication code
}
```

## Key Rotation Schedule

| Key Type | Rotation Frequency | Trigger |
|----------|-------------------|---------|
| Identity Key | Never (unless reset) | Device reset only |
| Signed Prekey | Weekly | Time-based |
| One-time Prekeys | Per use | Message initiation |
| Ephemeral Keys | Per message | Each DH ratchet step |
| Message Keys | Per message | Symmetric ratchet |

## Implementation Considerations

### Server Responsibilities
- Store and distribute key bundles
- Forward encrypted messages
- Rate limiting and spam protection
- **Cannot decrypt messages** (zero-knowledge)

### Client Responsibilities  
- Generate and manage all cryptographic keys
- Perform X3DH key agreement
- Implement Double Ratchet protocol
- Handle key rotation and cleanup

### Out-of-Order Message Handling
- Maintain skipped message keys
- Buffer future messages temporarily
- Implement reasonable limits on storage

## Security Assumptions

1. **Discrete Logarithm Problem**: Curve25519 security
2. **Server Honesty**: Server correctly delivers messages
3. **Device Security**: Private keys remain private
4. **Implementation Correctness**: Proper cryptographic implementation

## Common Attack Mitigations

### Man-in-the-Middle (MITM)
- **Protection**: Identity key verification (safety numbers)
- **User Action**: Manual verification required

### Replay Attacks
- **Protection**: Message counters and MAC verification
- **Automatic**: Built into protocol

### Key Compromise
- **Protection**: Forward secrecy and post-compromise security
- **Automatic**: Key rotation and deletion

## References

- [Signal Protocol Specification](https://signal.org/docs/)
- [X3DH Key Agreement Protocol](https://signal.org/docs/specifications/x3dh/)
- [Double Ratchet Algorithm](https://signal.org/docs/specifications/doubleratchet/)
- [XEdDSA and VXEdDSA Signatures](https://signal.org/docs/specifications/xeddsa/) 
