---
connections: []
reference: 
tags:
  - documentation_note
  - flutter
  - signal_protocol
  - firebase
  - encryption
type: documentation_note
created: 2025-08-22
---

# Flutter Signal Protocol Implementation Flow
**Using libsignal_protocol_dart with Firebase Backend**

## Architecture Overview

```
Flutter App (libsignal_protocol_dart)
           ↕
Firebase Cloud Functions (Key Management)
           ↕
Firebase Firestore (Key Storage & Message Relay)
           ↕
Firebase Realtime Database (Message Queue)
```

## Scenario 1: New User Registration

### Step-by-Step Flow

#### 1.1 User Registration Initiation
```
User Input (Phone/Email) → Firebase Auth → Generate User ID
```

#### 1.2 Signal Protocol Key Generation (Local Device)
```
Generate Identity Key Pair
├── Identity Private Key → Store in Flutter Secure Storage
└── Identity Public Key → Prepare for upload

Generate Signed Prekey Bundle
├── Signed Prekey Private → Store in Flutter Secure Storage
├── Signed Prekey Public → Prepare for upload
└── Signature → Sign with Identity Private Key

Generate One-Time Prekeys (100 keys)
├── One-Time Private Keys → Store in Flutter Secure Storage
└── One-Time Public Keys → Prepare for upload
```

#### 1.3 Key Upload to Firebase
```
Firebase Cloud Function: uploadUserKeys()
├── Input: User ID, Key Bundle
├── Store in Firestore: /users/{userId}/keys/
│   ├── identityKey: {publicKey, timestamp}
│   ├── signedPrekey: {publicKey, signature, timestamp}
│   └── oneTimeKeys: [{keyId, publicKey}, ...]
└── Return: Success/Failure
```

#### 1.4 Local Storage Setup
```
Flutter Secure Storage Structure:
├── identity_private_key
├── signed_prekey_private
├── one_time_keys_private: {keyId: privateKey}
├── registration_id
└── device_id
```

#### 1.5 Session Store Initialization
```
Initialize SQLite Database:
├── sessions_table
├── pre_keys_table
├── signed_pre_keys_table
└── identity_keys_table
```

---

## Scenario 2: User Deletes App → Reinstall → Login

### Step-by-Step Flow

#### 2.1 App Reinstallation Detection
```
Firebase Auth Login → Check if keys exist in Firestore
├── Keys Exist → Old Installation
└── No Keys → New Installation (Go to Scenario 1)
```

#### 2.2 Key Recovery Decision
```
User Choice:
├── Restore Previous Session
│   ├── Download existing keys (if backup enabled)
│   └── Warning: Cannot decrypt old messages
└── Fresh Start
    ├── Generate new keys
    └── Warning: Previous conversations lost
```

#### 2.3 Fresh Start Flow (Recommended)
```
Generate New Identity Key Pair
├── Mark old keys as "deprecated" in Firestore
├── Generate new key bundle
├── Upload new keys to Firebase
└── Notify contacts about key change
```

#### 2.4 Local Storage Recreation
```
Clear All Local Storage
├── Delete old session database
├── Generate new registration ID
├── Store new private keys
└── Initialize fresh session store
```

#### 2.5 Contact Notification
```
Firebase Cloud Function: notifyKeyChange()
├── Input: User ID, New Identity Key
├── Update: /users/{userId}/keyChangeTimestamp
└── Trigger: Push notifications to contacts
```

---

## Scenario 3: User Sends a Message

### Step-by-Step Flow

#### 3.1 Message Composition
```
User Input → Compose Message → Select Recipient
```

#### 3.2 Session Check
```
Check Local Session Store
├── Session Exists → Use existing session
└── No Session → Initiate new session (3.3)
```

#### 3.3 New Session Initiation (X3DH)
```
Firebase Cloud Function: fetchRecipientKeys()
├── Input: Recipient User ID
├── Fetch from Firestore: /users/{recipientId}/keys/
│   ├── Identity Key
│   ├── Signed Prekey
│   └── One One-Time Key (mark as used)
└── Return: Key Bundle

Local X3DH Calculation:
├── Generate Ephemeral Key Pair
├── Perform DH calculations (DH1, DH2, DH3, DH4)
├── Derive Initial Root Key and Chain Key
└── Store Session in local SQLite
```

#### 3.4 Message Encryption
```
Double Ratchet Encryption:
├── Get current Chain Key from session
├── Derive Message Key
├── Encrypt message with AES-256-GCM
├── Generate MAC
├── Update session state
└── Create encrypted message envelope
```

#### 3.5 Message Upload
```
Firebase Cloud Function: sendMessage()
├── Input: Encrypted Message + Metadata
├── Store in Firestore: /messages/{messageId}
│   ├── senderId
│   ├── recipientId
│   ├── encryptedContent
│   ├── header (DH public key, counters)
│   ├── timestamp
│   └── messageType
├── Add to Realtime DB: /messageQueue/{recipientId}/{messageId}
└── Trigger push notification
```

#### 3.6 Local Message Storage
```
Store in Local SQLite:
├── message_id
├── recipient_id
├── encrypted_content
├── sent_timestamp
├── delivery_status: "sent"
└── session_state_snapshot
```

---

## Scenario 4: User Receives a Message

### Step-by-Step Flow

#### 4.1 Message Notification
```
Firebase Push Notification → App Wake/Foreground
├── Contains: Message ID, Sender ID
└── Triggers: Message fetch process
```

#### 4.2 Message Fetching
```
Firebase Realtime DB Listener: /messageQueue/{userId}/
├── New message detected
├── Fetch full message from Firestore
├── Remove from message queue
└── Begin decryption process
```

#### 4.3 Session Management
```
Check Local Session Store:
├── Session Exists → Load session state
└── No Session → Initialize from message header
```

#### 4.4 Message Decryption
```
Double Ratchet Decryption:
├── Extract DH public key from header
├── Check if DH ratchet step needed
│   ├── New DH key → Perform DH ratchet
│   └── Same DH key → Use existing chain
├── Derive Message Key from Chain Key
├── Decrypt message content
├── Verify MAC
├── Update session state
└── Handle out-of-order messages (if any)
```

#### 4.5 Message Processing
```
Successful Decryption:
├── Store decrypted message in local DB
├── Update UI with new message
├── Send delivery receipt to sender
└── Clean up used keys

Decryption Failure:
├── Log error details
├── Request key refresh from sender
├── Store encrypted message for retry
└── Notify user about decryption issue
```

#### 4.6 Delivery Receipt
```
Firebase Cloud Function: sendDeliveryReceipt()
├── Input: Message ID, Recipient ID
├── Update: /messages/{messageId}/deliveryStatus
└── Notify sender via push notification
```

---

## Key Storage Locations

### Firebase Firestore Structure
```
/users/{userId}/
├── keys/
│   ├── identityKey: {publicKey, timestamp}
│   ├── signedPrekey: {publicKey, signature, timestamp, keyId}
│   ├── oneTimeKeys/
│   │   └── {keyId}: {publicKey, used: boolean}
│   └── keyChangeTimestamp
├── profile/
│   ├── displayName
│   ├── phoneNumber
│   └── lastSeen
└── devices/
    └── {deviceId}: {registrationId, lastActive}

/messages/{messageId}/
├── senderId
├── recipientId
├── encryptedContent
├── header: {dhPublicKey, previousCounter, messageCounter}
├── timestamp
├── messageType: "prekey" | "message"
├── deliveryStatus: "sent" | "delivered" | "read"
└── expiresAt
```

### Firebase Realtime Database Structure
```
/messageQueue/{userId}/
└── {messageId}: {
    senderId,
    timestamp,
    priority: "high" | "normal"
}

/onlineStatus/{userId}/
├── isOnline: boolean
├── lastSeen: timestamp
└── deviceId
```

### Local Flutter Storage

#### Flutter Secure Storage
```
identity_private_key: Base64 encoded Ed25519 private key
signed_prekey_private: Base64 encoded Curve25519 private key
current_signed_prekey_id: Integer
one_time_keys_private: JSON {keyId: privateKey}
registration_id: Integer (random 14-bit number)
device_id: String (UUID)
```

#### SQLite Database Schema
```sql
-- Sessions table
CREATE TABLE sessions (
    address TEXT PRIMARY KEY,
    record BLOB NOT NULL,
    timestamp INTEGER DEFAULT CURRENT_TIMESTAMP
);

-- PreKeys table  
CREATE TABLE pre_keys (
    key_id INTEGER PRIMARY KEY,
    record BLOB NOT NULL
);

-- Signed PreKeys table
CREATE TABLE signed_pre_keys (
    key_id INTEGER PRIMARY KEY,
    record BLOB NOT NULL,
    timestamp INTEGER DEFAULT CURRENT_TIMESTAMP
);

-- Identity Keys table
CREATE TABLE identity_keys (
    address TEXT PRIMARY KEY,
    identity_key BLOB NOT NULL,
    trusted INTEGER DEFAULT 0
);

-- Messages table
CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    thread_id TEXT NOT NULL,
    sender_id TEXT NOT NULL,
    content TEXT NOT NULL,
    timestamp INTEGER NOT NULL,
    message_type TEXT NOT NULL,
    delivery_status TEXT DEFAULT 'sent'
);

-- Threads table
CREATE TABLE threads (
    id TEXT PRIMARY KEY,
    participant_id TEXT NOT NULL,
    last_message_timestamp INTEGER,
    unread_count INTEGER DEFAULT 0
);
```

---

## Security Considerations

### Key Rotation Schedule
```
Identity Keys: Never (unless device reset/compromise)
Signed Prekeys: Weekly rotation
One-Time Prekeys: Replenish when < 20 remaining
Session Keys: Every message (Double Ratchet)
```

### Error Handling
```
Key Generation Failure → Retry with backoff
Network Failure → Queue messages locally
Decryption Failure → Request key refresh
Session Corruption → Re-establish session
```

### Privacy Measures
```
Message Metadata: Minimal storage on server
Forward Secrecy: Delete old keys after use
Key Verification: Safety numbers for contacts
Sealed Sender: Hide sender identity from server
```

---

## Implementation Checklist

### Phase 1: Basic Setup
- [ ] Firebase project configuration
- [ ] libsignal_protocol_dart integration
- [ ] Flutter Secure Storage setup
- [ ] SQLite database initialization
- [ ] User registration flow

### Phase 2: Core Messaging
- [ ] Key generation and upload
- [ ] Session establishment (X3DH)
- [ ] Message encryption/decryption
- [ ] Firebase message relay
- [ ] Push notifications

### Phase 3: Advanced Features
- [ ] Key rotation automation
- [ ] Out-of-order message handling
- [ ] Multi-device support
- [ ] Message backup/restore
- [ ] Safety number verification

### Phase 4: Production Ready
- [ ] Error handling and retry logic
- [ ] Performance optimization
- [ ] Security audit
- [ ] Scalability testing
- [ ] User experience polish
