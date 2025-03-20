# Project Overview

This project focuses on implementing post-quantum cryptography (PQC) protocols for secure messaging, key exchange, and digital signature verification. The system uses quantum-resistant algorithms to ensure secure communication and data integrity in a future-proof way. Below is a detailed description of the various components of the project.

---

## App Folder

### `app/__init__.py`

This file is the main entry point for the application package, located in the `app` folder. It imports key classes from the `messaging` and `logging` modules, making them available for use throughout the application.

#### Key Components:
- **`SecureMessaging`**: Class handling post-quantum secure messaging.
- **`Message`**: Class representing a message within the application.
- **`MessageStore`**: Class for storing and managing messages.
- **`SecureLogger`**: Logger class for secure logging purposes.

The `__init__.py` file ensures that only the specified classes and functions are accessible when importing this package, maintaining a clean and organized API for the application.

### `app/logging.py`

This file provides the functionality for secure logging in cryptographic operations, including key exchanges, message transfers, and security-related events. The logging is enhanced with encryption and file locking mechanisms for reliability.

#### Key Components:
- **`SecureLogger`**: The main class for secure logging in the application. It provides methods for logging events securely and reading event data from encrypted log files.
  
- **File Locking Mechanism**: Platform-specific file locking to ensure thread safety and avoid file corruption. On Windows, it uses `msvcrt` for file locking, and on Unix-like systems, it uses `fcntl`.

#### Logging Features:
- **Log Encryption**: Logs are encrypted using an AES-256-GCM cipher, ensuring that sensitive data in the logs is protected.
- **Event Logging**: Logs events with associated metadata like timestamps, event types, and additional information passed as keyword arguments.
- **Error Handling**: Includes mechanisms to avoid recursive error logging and attempts to recover from corrupted log files.
- **Log Event Retrieval**: Provides methods to retrieve and filter log events based on time ranges, event types, or other criteria.

#### Functions and Methods:
- **`__init__(self, log_path: Optional[str] = None, encryption_key: Optional[bytes] = None)`**: Initializes a secure logger, setting up the log path and encryption key.
- **`_load_or_generate_key(self) -> bytes`**: Loads or generates an encryption key for log encryption.
- **`log_event(self, event_type: str, **kwargs) -> None`**: Logs a security-related event, including encryption of log data.
- **`get_events(self, start_time: Optional[float] = None, end_time: Optional[float] = None, event_type: Optional[str] = None, limit: Optional[int] = None) -> List[Dict[str, Any]]`**: Retrieves events from the logs, filtering by time range, event type, or limit.
- **`get_event_summary(self, start_time: Optional[float] = None, end_time: Optional[float] = None) -> Dict[str, int]`**: Retrieves a summary of events by type within a given time range.
- **`get_security_metrics(self) -> Dict[str, Any]`**: Calculates and returns security metrics based on the logs.
- **`clear_logs(self) -> None`**: Clears all log files, deleting any entries in the log directory.

### `app/secure_messaging.py`

This file contains the core logic for the **secure messaging** system in the application. It handles peer-to-peer communication using **post-quantum cryptography** and includes functionalities like message encryption, key exchange, and signature verification.

#### Key Components:
- **`Message`**: Represents a message in the secure messaging system, encapsulating content, sender information, encryption algorithms, and more.
- **`KeyExchangeState`**: Defines the different states of a key exchange process (e.g., initiated, confirmed, established).
- **`SecureMessaging`**: The main class responsible for secure message exchange. It manages key exchange, symmetric encryption, and digital signatures.
- **`KeyExchangeAlgorithm`**: Abstract base class for key exchange algorithms.
- **`KyberKeyExchange`**: Post-quantum key exchange algorithm using Kyber.
- **`AES256GCM` & `ChaCha20Poly1305`**: Symmetric encryption algorithms for securely encrypting messages.
- **`DilithiumSignature` & `SPHINCSSignature`**: Signature algorithms used to ensure message authenticity.

The module leverages advanced cryptographic algorithms to securely exchange messages between peers and provide quantum-resistant communication.

---

## Crypto Folder

### `crypto/__init__.py`

This file initializes the cryptographic layer for the application. It provides implementations of post-quantum cryptographic algorithms, symmetric encryption methods, digital signatures, and secure key storage. Additionally, it includes backward compatibility for existing code that uses previous cryptographic standards.

#### Key Components:
- **Key Exchange Algorithms**:
  - **`KeyExchangeAlgorithm`**: Base class for key exchange algorithms.
  - **`MLKEMKeyExchange`**, **`HQCKeyExchange`**, **`FrodoKEMKeyExchange`**, **`NTRUKeyExchange`**: Specific implementations of post-quantum key exchange algorithms.
  - **`KyberKeyExchange`** (deprecated): Alias for **`MLKEMKeyExchange`** for backward compatibility.
  
- **Symmetric Encryption Algorithms**:
  - **`SymmetricAlgorithm`**: Base class for symmetric encryption algorithms.
  - **`AES256GCM`**, **`ChaCha20Poly1305`**: Implementations of widely-used symmetric encryption algorithms.

- **Digital Signatures**:
  - **`SignatureAlgorithm`**: Base class for signature algorithms.
  - **`MLDSASignature`**, **`SPHINCSSignature`**: Post-quantum digital signature algorithms.
  - **`DilithiumSignature`** (deprecated): Alias for **`MLDSASignature`** for backward compatibility.

- **Key Storage**:
  - **`KeyStorage`**: A class that handles storing and retrieving cryptographic keys.

- **Crypto Algorithm Base**:
  - **`CryptoAlgorithm`**: Base class for all cryptographic algorithms.

- **LIBOQS (Optional)**:
  - The file attempts to import the **`oqs`** library, which provides post-quantum cryptographic algorithms. If available, it sets the **`LIBOQS_AVAILABLE`** flag and retrieves the **`LIBOQS_VERSION`**.

### `crypto/algorithm_base.py`

This file contains the base classes for cryptographic algorithms used within the application. It provides an abstract class `CryptoAlgorithm`, which serves as the foundation for different cryptographic algorithm implementations, both post-quantum and traditional.

#### Key Components:
- **`CryptoAlgorithm`**: An abstract base class for all cryptographic algorithms, defining essential properties and methods that every algorithm implementation must provide. 
    - **`name`**: The internal name of the algorithm (including implementation details).
    - **`display_name`**: A user-friendly name for the algorithm, typically used in the UI (removes `[Mock]` suffix if present).
    - **`description`**: A description of the algorithm, explaining its functionality or features.
    - **`is_using_mock`**: A boolean flag to check if the algorithm is using a mock implementation (helpful for testing purposes).
    - **`actual_variant`**: Returns the actual variant of the algorithm if applicable (e.g., a specific OQS variant).
    - **`get_security_info()`**: Provides detailed security information about the algorithm, including security level, description, and whether it's a mock implementation.

#### Purpose:
The `CryptoAlgorithm` class establishes a template for cryptographic algorithms that is extended by more specific algorithms (like key exchange, encryption, etc.). It provides standardized access to metadata about the algorithm and ensures that all derived algorithms implement essential properties and methods for consistency across the system.

---

### `crypto/key_exchange.py`

This file contains the implementation of post-quantum key exchange algorithms, with the `KeyExchangeAlgorithm` serving as an abstract base class, and specific implementations like `MLKEMKeyExchange` providing the actual key exchange functionality.

#### Key Components:
- **`KeyExchangeAlgorithm`**: 
    - An abstract base class for all key exchange algorithms, defining essential methods like `generate_keypair`, `encapsulate`, and `decapsulate`.
    - **Methods**:
        - **`generate_keypair()`**: Generates a new public/private keypair.
        - **`encapsulate(public_key: bytes)`**: Encapsulates a shared secret using the recipient's public key.
        - **`decapsulate(private_key: bytes, ciphertext: bytes)`**: Decapsulates the shared secret using the recipient's private key.

- **`MLKEMKeyExchange`**: 
    - This class implements the ML-KEM (Module-Lattice-based Key Encapsulation Mechanism) key exchange algorithm, which is a post-quantum algorithm based on the hardness of the Learning With Errors (LWE) problem over module lattices.
    - **Constructor**:
        - **`__init__(security_level: int)`**: Initializes the ML-KEM key exchange algorithm with a specified security level (1, 3, or 5).
    - **Properties**:
        - **`name`**: Returns the internal name of the algorithm.
        - **`display_name`**: Provides a user-friendly name for the algorithm.
        - **`description`**: A description of the algorithm, including whether it's using a mock implementation.
        - **`is_using_mock`**: Indicates whether a mock implementation is being used due to the unavailability of the OQS library.
    - **Methods**:
        - **`generate_keypair()`**: Generates a public/private keypair for ML-KEM, either using a mock implementation or the actual OQS library.
        - **`encapsulate(public_key: bytes)`**: Encapsulates a shared secret using the recipient's public key.
        - **`decapsulate(private_key: bytes, ciphertext: bytes)`**: Decapsulates the shared secret using the recipient's private key.

#### Purpose:
This script defines the cryptographic algorithms responsible for securely exchanging keys between parties, focusing on post-quantum algorithms like ML-KEM. The implementation allows for fallback to deterministic mock algorithms when the Open Quantum Safe (OQS) library is unavailable, ensuring that the system can still function in testing environments or when OQS is not installed.

---

## Networking Folder

### `networking/discovery.py`

This script implements a node discovery service for a peer-to-peer (P2P) network using UDP broadcast messages. It allows nodes to discover each other on the local network automatically and supports manual peer addition.

#### Key Features of the Script:
- **NodeDiscovery Class**: The `NodeDiscovery` class is responsible for discovering and managing nodes in the network. It supports both automatic node discovery via broadcast messages and manual peer addition.
- **Automatic Node Discovery**: Nodes periodically announce their presence to the local network via UDP broadcasts.
- **Manual Node Addition**: Nodes can be manually added using the `add_known_node()` method.
- **Periodic Announcement**: The node periodically announces its presence every minute to ensure that other nodes are aware of its availability.
- **Node Expiry Cleanup**: Nodes that haven't been seen in the last 5 minutes are periodically removed from the discovery list.

### `networking/p2p_node.py`

This script implements a **P2P Node** (peer-to-peer node) for quantum-resistant communication between nodes. It uses the `asyncio` library for managing asynchronous connections and implements the message routing, relay, and secure peer communication functionalities.

#### Key Features:
- **P2PNode Class**: The `P2PNode` class is responsible for handling inbound and outbound connections, message routing, and secure communication.
- **Peer-to-Peer Communication**: Nodes communicate with each other over secure channels, with the encryption of messages ensured by the `secure_messaging` module.
- **Message Routing**: The `P2PNode` class ensures messages are routed to the correct peers using their public keys and cryptographic protocols.
