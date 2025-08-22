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
- **Private Key Storage**: Securely stored on device, never transmitted
- **Public Key Distribution**: Uploaded to server for verification

### 2. Prekeys
- **Signed Prekey**: 
  - Curve25519 key pair
  - Signed by identity key
  - Rotated periodically (weekly)
  - Provides forward secrecy
  - **Private Key**: Stored locally, used for key agreement
  - **Public Key**: Uploaded to server with signature
  
- **One-time Prekeys**:
  - Curve25519 key pairs
  - Unsigned
  - Single-use only
  - **Private Key**: Deleted immediately after use
  - **Public Key**: Consumed from server after use

### 3. Ephemeral Keys
- **Purpose**: Session establishment
- **Type**: Curve25519 key pair
- **Lifetime**: Single message exchange
- **Function**: Part of X3DH key agreement
- **Private Key**: Generated temporarily, deleted after key agreement

### 4. Chain Keys and Message Keys
- **Root Key**: Derives new chain keys
- **Chain Key**: Derives message keys and next chain key
- **Message Key**: Encrypts individual messages
- **Ratchet Keys**: Drive the Double Ratchet forward

## Private Key Management

### Key Storage Principles
1. **Never Leave Device**: Private keys never transmitted over network
2. **Secure Storage**: Stored in device's secure keystore/keychain
3. **Memory Protection**: Cleared from memory after use
4. **No Persistence**: Ephemeral and message keys deleted immediately

### Private Key Lifecycle

| Key Type | Generation | Storage | Deletion |
|----------|------------|---------|----------|
| Identity Private Key | Device setup | Secure keystore | Device reset only |
| Signed Prekey Private | Weekly rotation | Secure storage | After rotation |
| One-time Prekey Private | Batch generation | Temporary storage | After single use |
| Ephemeral Private | Per session | Memory only | After key agreement |
| Message Private Keys | Per message | Memory only | Immediate after use |

### Security Measures for Private Keys

#### Device-Level Protection
- **Hardware Security Module (HSM)**: When available
- **Trusted Execution Environment (TEE)**: ARM TrustZone, Intel SGX
- **Keystore Encryption**: OS-level key protection
- **Biometric Protection**: Fingerprint/face unlock for key access

#### Application-Level Protection
- **Key Derivation**: PBKDF2/Argon2 for user-derived keys
- **Memory Clearing**: Explicit zeroing of key material
- **Anti-Debugging**: Protection against memory dumps
- **Root/Jailbreak Detection**: Additional security on compromised devices

#### Cryptographic Protection
- **Forward Secrecy**: Old private keys cannot decrypt new messages
- **Post-Compromise Security**: New key generation heals from compromise
- **Key Separation**: Different keys for different purposes

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

#### Private Key Usage in X3DH:
- **Device A** uses its private identity key and ephemeral private key
- **Device B** uses its private identity key, signed prekey private, and one-time prekey private
- **Server never sees**: Any private keys or the shared secret
- **Key Cleanup**: Ephemeral and one-time private keys deleted after use

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

### Private Key Compromise Scenarios

#### Identity Key Compromise
- **Impact**: Attacker can impersonate user and read messages sent to compromised identity
- **Mitigation**: Safety number verification, key rotation after detection
- **Recovery**: Generate new identity key, re-establish all sessions

#### Prekey Compromise  
- **Impact**: Limited to messages using compromised prekey
- **Mitigation**: Automatic weekly rotation of signed prekeys
- **Recovery**: Forward secrecy protects future messages

#### Session Key Compromise
- **Impact**: Attacker can read messages until next DH ratchet step
- **Mitigation**: Frequent key rotation in Double Ratchet
- **Recovery**: Next message with new DH key heals the session

#### Device Compromise
- **Impact**: All current private keys potentially compromised
- **Mitigation**: Remote wipe, session reset, safety number changes
- **Recovery**: Post-compromise security through new key generation

## References

- [Signal Protocol Specification](https://signal.org/docs/)
- [X3DH Key Agreement Protocol](https://signal.org/docs/specifications/x3dh/)
- [Double Ratchet Algorithm](https://signal.org/docs/specifications/doubleratchet/)
- [XEdDSA and VXEdDSA Signatures](https://signal.org/docs/specifications/xeddsa/) 
