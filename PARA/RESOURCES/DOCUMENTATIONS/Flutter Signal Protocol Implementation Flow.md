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
```dart
// Install time key generation
final identityKeyPair = generateIdentityKeyPair();
final registrationId = generateRegistrationId(false);
final preKeys = generatePreKeys(0, 110);  // Generate 110 one-time prekeys
final signedPreKey = generateSignedPreKey(identityKeyPair, 0);

Storage Plan:
├── identityKeyPair → Store in Flutter Secure Storage
├── registrationId → Store in Flutter Secure Storage  
├── preKeys → Store in InMemoryPreKeyStore + Secure Storage backup
└── signedPreKey → Store in InMemorySignedPreKeyStore + Secure Storage backup
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
```dart
// Initialize Signal Protocol Stores
final sessionStore = InMemorySessionStore();
final preKeyStore = InMemoryPreKeyStore();
final signedPreKeyStore = InMemorySignedPreKeyStore();
final identityStore = InMemoryIdentityKeyStore(identityKeyPair, registrationId);

// Store all prekeys
for (var p in preKeys) {
  await preKeyStore.storePreKey(p.id, p);
}
await signedPreKeyStore.storeSignedPreKey(signedPreKey.id, signedPreKey);

Flutter Secure Storage Backup:
├── identity_key_pair → Serialized identityKeyPair
├── registration_id → registrationId
├── pre_keys_backup → Serialized preKeys array
└── signed_pre_key_backup → Serialized signedPreKey
```

#### 1.5 Session Store Initialization
```dart
// No SQLite needed - Using In-Memory Stores
// But for persistence across app restarts, implement:
// - PersistentSessionStore
// - PersistentPreKeyStore  
// - PersistentSignedPreKeyStore
// - PersistentIdentityKeyStore

// These would extend the In-Memory stores and add SQLite persistence
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
```dart
// Clear and reinitialize stores
final newIdentityKeyPair = generateIdentityKeyPair();
final newRegistrationId = generateRegistrationId(false);
final newPreKeys = generatePreKeys(0, 110);
final newSignedPreKey = generateSignedPreKey(newIdentityKeyPair, 0);

// Create fresh stores
final sessionStore = InMemorySessionStore();
final preKeyStore = InMemoryPreKeyStore();  
final signedPreKeyStore = InMemorySignedPreKeyStore();
final identityStore = InMemoryIdentityKeyStore(newIdentityKeyPair, newRegistrationId);

// Populate stores
for (var p in newPreKeys) {
  await preKeyStore.storePreKey(p.id, p);
}
await signedPreKeyStore.storeSignedPreKey(newSignedPreKey.id, newSignedPreKey);
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
```dart
// Check if session exists with recipient
final remoteAddress = SignalProtocolAddress(recipientUserId, deviceId);
final hasSession = await sessionStore.containsSession(remoteAddress);

if (hasSession) {
  // Use existing session
  final sessionCipher = SessionCipher(sessionStore, preKeyStore, 
      signedPreKeyStore, identityStore, remoteAddress);
} else {
  // Need to build new session (go to 3.3)
}
```

#### 3.3 New Session Initiation (Building Session)
```dart
// Fetch recipient's key bundle from Firebase
final retrievedPreKey = await fetchRecipientKeyBundle(recipientUserId);

// Build session using SessionBuilder
final remoteAddress = SignalProtocolAddress(recipientUserId, deviceId);
final sessionBuilder = SessionBuilder(sessionStore, preKeyStore,
    signedPreKeyStore, identityStore, remoteAddress);

// Process the retrieved prekey bundle (performs X3DH internally)
await sessionBuilder.processPreKeyBundle(retrievedPreKey);

// Session is now established and stored in sessionStore
```

#### 3.4 Message Encryption
```dart
// Create SessionCipher for encryption
final sessionCipher = SessionCipher(sessionStore, preKeyStore,
    signedPreKeyStore, identityStore, remoteAddress);

// Encrypt the message
final messageBytes = utf8.encode(messageText);
final ciphertext = await sessionCipher.encrypt(messageBytes);

// ciphertext contains the encrypted message ready for transmission
// The Double Ratchet algorithm is handled internally by SessionCipher
```

#### 3.5 Message Upload
```dart
// Prepare message for Firebase upload
final encryptedMessage = {
  'senderId': currentUserId,
  'recipientId': recipientUserId,
  'ciphertext': ciphertext.serialize(), // Serialized CiphertextMessage
  'type': ciphertext.getType(), // MESSAGE_TYPE or PREKEY_TYPE
  'timestamp': DateTime.now().millisecondsSinceEpoch,
  'messageId': generateMessageId(),
};

// Upload via Firebase Cloud Function
await sendMessageToFirebase(encryptedMessage);
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
```dart
// Load existing session
final remoteAddress = SignalProtocolAddress(senderId, deviceId);
final hasSession = await sessionStore.containsSession(remoteAddress);

if (!hasSession) {
  // This should not happen for normal messages
  // PreKeyMessage should have been sent first
  throw Exception('No session found for sender');
}
```

#### 4.4 Message Decryption
```dart
// Create SessionCipher for decryption
final sessionCipher = SessionCipher(sessionStore, preKeyStore,
    signedPreKeyStore, identityStore, remoteAddress);

// Deserialize the received ciphertext
final ciphertext = CiphertextMessage.fromSerialized(encryptedMessage['ciphertext']);

// Decrypt the message
final decryptedBytes = await sessionCipher.decrypt(ciphertext);
final decryptedMessage = utf8.decode(decryptedBytes);

// Double Ratchet algorithm is handled internally by SessionCipher
// Session state is automatically updated
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

### Local Flutter Storage

#### Flutter Secure Storage
```dart
// Key-value pairs stored securely
identity_key_pair: String // Serialized IdentityKeyPair
registration_id: int // Generated registration ID
signed_pre_key_backup: String // Serialized SignedPreKey for recovery
pre_keys_backup: String // JSON array of serialized PreKeys
device_id: String // UUID for this device installation
```

#### In-Memory Stores (Runtime)
```dart
// These stores hold the active keys during app runtime
InMemorySessionStore sessionStore;
InMemoryPreKeyStore preKeyStore;
InMemorySignedPreKeyStore signedPreKeyStore; 
InMemoryIdentityKeyStore identityStore;

// For persistence across app restarts, implement:
// PersistentSessionStore, PersistentPreKeyStore, etc.
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

---

## Scenario 5: Message Delivery During App Deletion (Critical Edge Case)

### The Problem: User B Deletes App While Messages Are In-Transit

#### Timeline of Events:
```
1. User A and B have active Signal session
2. User B deletes app (loses all session state)
3. User A sends Message 1 (using old session)
4. User A sends Message 2 (using old session)
5. User B reinstalls app
6. User B chooses "Restore Backup" or "Fresh Start"
```

### Analysis: Can B Decrypt A's Messages?

#### Case 1: B Chooses "Fresh Start" (New Keys)
```
Result: ❌ CANNOT DECRYPT
Reason: 
├── A's messages encrypted with old session state
├── B generates completely new identity keys
├── No way to recover old session state
└── Forward secrecy prevents decryption

Action Required:
├── A detects B's key change (safety number change)
├── A must re-establish session with B's new keys
├── A should resend the 2 messages (user decision)
└── Future messages will work normally
```

#### Case 2: B Chooses "Restore Backup" (Same Keys)
```
Result: ❌ STILL CANNOT DECRYPT
Reason:
├── Session state is stored locally (not in backup)
├── Double Ratchet session state is lost
├── A's messages reference chain keys B no longer has
├── Session desynchronization occurred

What's Lost:
├── Root Key state
├── Chain Key counters  
├── Sending/Receiving key chains
├── Message counter positions
└── Skipped message keys

What's Restored:
├── Identity Key Pair ✓
├── Signed PreKey ✓  
├── One-Time PreKeys ✓
└── Registration ID ✓
```

### Technical Deep Dive: Why Messages Are Lost

#### Double Ratchet State Dependency
```dart
// Session state that's lost when app is deleted:
RatchetState {
  DHs,         // Current sending DH key pair
  DHr,         // Current receiving DH public key
  RK,          // Root key (derives chain keys)
  CKs,         // Chain key for sending
  CKr,         // Chain key for receiving  
  Ns,          // Message number for sending
  Nr,          // Message number for receiving
  PN,          // Previous chain length
  MKSKIPPED    // Skipped message keys for out-of-order
}

// When B deletes app, ALL of this state is lost forever
// A's messages depend on this exact state to decrypt
```

### Solution Patterns

#### Option 1: Session Re-establishment Detection
```dart
// In your Firebase message handling
if (decryptionFailed && ciphertext.getType() == MESSAGE_TYPE) {
  // Regular message failed - session might be broken
  await requestSessionReset(senderId);
  // Store failed message for later retry
  await storeFailedMessage(messageId, encryptedData);
}

// Sender (A) should detect and handle:
if (receivedSessionResetRequest) {
  // Re-establish session with fresh X3DH
  await rebuildSessionWith(recipientId);
  // Optionally resend recent failed messages
  await resendFailedMessages(recipientId);
}
```

#### Option 2: Message Recovery Flow
```dart
// Store messages that can't be decrypted
class FailedMessage {
  final String messageId;
  final String senderId;
  final Uint8List encryptedData;
  final DateTime timestamp;
  final int retryCount;
}

// Retry decryption after session re-establishment
async retryFailedMessages(String senderId) {
  final failedMessages = await getFailedMessagesFor(senderId);
  for (final msg in failedMessages) {
    try {
      final decrypted = await sessionCipher.decrypt(msg.encryptedData);
      await processDecryptedMessage(msg.messageId, decrypted);
      await deleteFailedMessage(msg.messageId);
    } catch (e) {
      if (msg.retryCount > 3) {
        await markMessageAsUnrecoverable(msg.messageId);
      }
    }
  }
}
```

### User Experience Recommendations

#### What Users Should Expect
```
❌ "Messages sent while you were offline will be lost"
✅ "Some recent messages may need to be resent"

❌ Technical error messages
✅ "Secure connection re-established with [Contact]"
```

#### Recommended UX Flow
```
1. B reinstalls and chooses restore/fresh start
2. A sends messages that fail to decrypt
3. App automatically detects failure and requests session reset
4. Show B: "Secure connection with A has been re-established"  
5. Show A: "B's security keys changed. Recent messages may need to be resent"
6. Both users can continue messaging normally
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

---

## libsignal_protocol_dart Key Classes & Methods

### Key Generation Functions
```dart
// Core key generation methods
IdentityKeyPair generateIdentityKeyPair()
int generateRegistrationId(bool extendedRange)
List<PreKeyRecord> generatePreKeys(int start, int count)
SignedPreKeyRecord generateSignedPreKey(IdentityKeyPair identityKey, int signedPreKeyId)
```

### Store Interfaces Implementation
```dart
// In-Memory implementations (provided by library)
InMemorySessionStore() // Implements SessionStore
InMemoryPreKeyStore() // Implements PreKeyStore  
InMemorySignedPreKeyStore() // Implements SignedPreKeyStore
InMemoryIdentityKeyStore(identityKeyPair, registrationId) // Implements IdentityKeyStore

// Key storage methods
await preKeyStore.storePreKey(preKey.id, preKey)
await signedPreKeyStore.storeSignedPreKey(signedPreKey.id, signedPreKey)
```

### Session Management Classes
```dart
// Address for identifying remote party
SignalProtocolAddress(userId, deviceId)

// Building sessions
SessionBuilder(sessionStore, preKeyStore, signedPreKeyStore, identityStore, remoteAddress)
await sessionBuilder.processPreKeyBundle(retrievedPreKey)

// Encrypting/Decrypting messages  
SessionCipher(sessionStore, preKeyStore, signedPreKeyStore, identityStore, remoteAddress)
CiphertextMessage ciphertext = await sessionCipher.encrypt(messageBytes)
Uint8List plaintext = await sessionCipher.decrypt(ciphertext)
```

### Message Types
```dart
// Different types of encrypted messages
CiphertextMessage.serialize() // Convert to bytes for transmission
CiphertextMessage.fromSerialized(bytes) // Reconstruct from bytes
ciphertext.getType() // Returns MESSAGE_TYPE or PREKEY_TYPE
```

### Key Bundle Structure (for Firebase)
```dart
// Structure to store in Firebase Firestore
class PreKeyBundle {
  final int registrationId;
  final int deviceId;
  final int preKeyId;
  final ECPublicKey preKeyPublic;
  final int signedPreKeyId;
  final ECPublicKey signedPreKeyPublic;
  final Uint8List signedPreKeySignature;
  final IdentityKey identityKey;
}
```

### Session State Management
```dart
// Check if session exists
bool hasSession = await sessionStore.containsSession(remoteAddress);

// Session persistence (for app restart survival)
// You'll need to implement persistent versions of the stores
// that save/load session state to/from local database
```
