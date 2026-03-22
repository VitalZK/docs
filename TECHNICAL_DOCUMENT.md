### **VITALZK: TECHNICAL SPECIFICATION DOCUMENT**
**Version:** 1.0  
**Date:** March 22, 2026  
**Status:** Initial Draft  
**Author:** Protocol Architect, Marvellous Ogunode  

---

#### **1. EXECUTIVE SUMMARY** VitalZK is a decentralized medical credentialing protocol designed to bridge the gap between healthcare data privacy and institutional verification. Utilizing Zero-Knowledge Proofs (ZKP) on the Stellar network, the protocol allows users to prove the validity of their medical records without exposing the underlying sensitive data. 

#### **2. PROJECT SCOPE AND OBJECTIVES** The primary objective of VitalZK is to provide a "Privacy-First" verification layer.  
* **Integrity:** Ensure medical records are signed by authorized entities.  
* **Confidentiality:** Zero sensitive health data is stored on-chain.  
* **Interoperability:** Use Stellar’s Soroban smart contracts for cross-border verification.  
* **Speed:** Leverage Stellar’s high-throughput consensus for near-instant verification.

---

#### **3. ARCHITECTURAL OVERVIEW** **3.1. The Data Model** All medical records follow a standardized schema. A record is a set of key-value pairs (e.g., `test_type: "COVID-19"`, `result: "Negative"`, `timestamp: 1711112400`) hashed using the **Poseidon** hashing algorithm.

**3.2. The Zero-Knowledge Circuit (Noir/Rust)** The circuit performs three main logic checks:  
1.  **Signature Verification:** Validates that the hash of the record was signed by a registered Hospital Public Key.  
2.  **State Verification:** Confirms the "Expiry Date" field is greater than the current network block time.  
3.  **Output Generation:** Produces a Boolean "True" and a Nullifier to prevent proof-replay.

---

#### **4. REPOSITORY BREAKDOWN** 
* **`vitalzk-circuits`**:  
    * *Purpose*: Houses the Noir ZK logic.  
    * *Deliverable*: Compiled `.json` artifacts containing the verification keys.
* **`vitalzk-contracts`**:  
    * *Purpose*: Soroban smart contracts.  
    * *Functions*: `add_issuer()`, `remove_issuer()`, `verify_proof(proof_data, public_inputs)`.
* **`vitalzk-sdk-ts`**:  
    * *Purpose*: The interface for web and mobile.  
    * *Functions*: Methods to fetch hospital keys and format proofs for Stellar transactions.
* **`vitalzk-mobile-vault`**:  
    * *Purpose*: Local-first Rust storage for users to sign and generate proofs on-device.
* **`vitalzk-docs`**:  
    * *Purpose*: documentation.
---

#### **5. TECHNICAL STACK** * **Layer 1:** Stellar Blockchain (Protocol 25).  
* **Execution Environment:** Soroban (WebAssembly-based Smart Contracts).  
* **Languages:** Rust (Contracts/Prover), TypeScript (SDK), Noir (Circuits), Expo(mobile).  
* **Crypto Primitives:** BN254 Curve, Poseidon Hash, Ed25519 Signatures.

---

#### **6. IMPLEMENTATION ROADMAP (5-DAY SPRINT)** **Day 1: Protocol Foundations** Initialize the GitHub Organization. Deploy the `IssuerRegistry` contract on Stellar Testnet to manage authorized medical providers.  

**Day 2: Logic and Circuits** Develop the ZK circuit in Noir. Implement range proofs (e.g., age or date checks) and hash-path verification.  

**Day 3: Contract Verification** Export the Noir verification key. Integrate the `bn254_pairing_check` into the Soroban smart contract to allow on-chain proof validation.  

**Day 4: The Mobile Vault** Set up the React Native/Rust bridge. Ensure the mobile device can generate a proof in under 5 seconds using local hardware.  

**Day 5: Integration & Launch** Connect the SDK to the contracts. Run a full simulation: Hospital signs data → User generates proof → Third-party verifies on Stellar.

---

#### **7. GLOSSARY OF TERMS** * **Soroban:** Stellar’s smart contract platform.  
* **Nullifier:** A unique value that ensures a specific proof cannot be used more than once.  
* **WASM:** WebAssembly, the runtime used by Soroban for high-performance execution.  

---

**8.1. The "IssuerRegistry" Smart Contract (Rust/Soroban)**
The `IssuerRegistry` acts as the source of truth for identity. It maintains a persistent map of authorized medical institutions.
* **Data Structure:** `Map<Address, IssuerMetadata>`
* **IssuerMetadata:** Includes the institution's public key, legal identifier, and "Trust Score" (used for tiered verification).
* **Access Control:** Implements a multi-signature requirement for adding new issuers, preventing a single compromised admin key from authorized fraudulent medical providers.

**8.2. The ZK-Circuit Logic (Noir/Rust)**
The circuit is the heart of the privacy layer. It is designed to be "Constraint-Optimized" to run efficiently on mobile hardware.
* **Private Inputs:** Raw Medical Record (JSON), Issuer Signature, User Private Key (to prove ownership).
* **Public Inputs:** Merkle Root of Authorized Issuers, Current Timestamp, Required Fact (e.g., "Result == Negative").
* **Constraint Breakdown:** * **EdDSA Signature Check:** Ensures the record hasn't been tampered with since the hospital signed it.
    * **Merkle Membership Proof:** Proves the signing hospital is currently in the `IssuerRegistry` without revealing which specific hospital issued it.
    * **Timestamp Logic:** Ensures the credential hasn't expired relative to the Stellar Ledger's `close_time`.

---

#### **9. THE "VITAL-VAULT" MOBILE INTEGRATION**

To maintain a pure section architecture, the mobile vault acts as a **Local-First Identity Provider**.
* **Secure Enclave Storage:** The user's private keys and raw health data are stored in the device's hardware-backed Secure Enclave (Keychain for iOS/Keystore for Android).
* **UniFFI Bridge:** Since the ZK-Prover is written in Rust, we use **UniFFI** to generate high-performance bindings for React Native. This allows the mobile app to call `generate_proof()` natively at C-speeds rather than through a slow JavaScript bridge.
* **Offline Capability:** Proofs are generated locally. The user only needs a data connection to broadcast the final small (approx. 300-byte) proof to the Stellar network.

---

#### **10. STELLAR ECOSYSTEM STANDARDS (SEP COMPLIANCE)**

VitalZK is designed to be compatible with existing Stellar standards to ensure it can be used by other wallets (like LOBSTR or Freighter).
* **SEP-10 (Stellar Web Authentication):** Used for the initial handshake between the Hospital portal and the User’s mobile vault.
* **SEP-30 (Recoverable Identities):** VitalZK will implement a social recovery mechanism. If a user loses their phone, they can recover their medical identity through a set of "Guardians" (trusted contacts) without needing a centralized backup of their health data.

---

#### **11. DETAILED 5-DAY TECHNICAL TASKS**

| Day | Module | Specific Task Detail |
| :--- | :--- | :--- |
| **Day 1** | **Registry** | Implement `soroban-sdk` contract; write unit tests for `add_issuer` and `revoke_issuer` using the `soroban-test` crate. |
| **Day 2** | **Circuit** | Write Noir circuit for `Poseidon` hashing of JSON-LD fields; implement the signature verification circuit. |
| **Day 3** | **Verification** | Generate the Solidity/WASM verifier from Noir; port logic to `vitalzk-verifier` Soroban contract; test with mock proofs. |
| **Day 4** | **SDK/Mobile** | Create `vitalzk-sdk` for proof serialization; set up `cargo-mobile` for cross-compiling the Rust prover to ARM64 (iOS/Android). |
| **Day 5** | **E2E Demo** | Deploy all contracts to **Futurenet/Testnet**; create a simple "Verification Portal" web app that accepts a QR code containing the ZK-Proof. |

---

#### **12. RISK MITIGATION AND AUDIT PATHWAY**

* **Constraint Leakage:** We ensure that the ZK-circuit does not accidentally leak the "Hospital ID" through public inputs.
* **Trusted Setup:** Since we are using **PlonK/UltraHonk**, we utilize the universal SRS (Structured Reference String), removing the need for a project-specific trusted setup ceremony.
* **Performance Bottlenecks:** If mobile proof generation exceeds 10 seconds, we will implement **Recursion**, breaking the proof into smaller chunks to be verified in parallel.

---
