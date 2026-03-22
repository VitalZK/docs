### **VITALZK: TECHNICAL SPECIFICATION DOCUMENT**
**Version:** 1.0  
**Date:** March 22, 2026  
**Status:** Initial Draft  
**Author:** Protocol Architecture Team  

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

#### **4. REPOSITORY BREAKDOWN** * **`vitalzk-circuits`**:  
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

---

#### **5. TECHNICAL STACK** * **Layer 1:** Stellar Blockchain (Protocol 25).  
* **Execution Environment:** Soroban (WebAssembly-based Smart Contracts).  
* **Languages:** Rust (Contracts/Prover), TypeScript (SDK), Noir (Circuits), expo(mobile).  
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
