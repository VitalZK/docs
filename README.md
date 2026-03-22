**VITALZK: DECENTRALIZED MEDICAL CREDENTIALING PROTOCOL**

---

### **1. PROJECT OVERVIEW**
**VitalZK** is a privacy-first medical credentialing protocol built on the **Stellar Blockchain** using **Soroban smart contracts**. The protocol allows individuals to prove medical assertions (vaccination status, test results, or health certifications) using **Zero-Knowledge Proofs (ZKP)**. By leveraging the **BN254 elliptic curve** and **Poseidon hashing** natively supported on Stellar, VitalZK ensures that sensitive health data remains on the user’s device while providing mathematically verifiable proof to third parties.

---

### **2. SYSTEM ARCHITECTURE**
The architecture is divided into three primary layers to ensure modularity and security:

#### **A. The Credential Layer (Off-Chain)**
* **Issuer (Hospital/Lab):** Digitally signs a data structure (JSON-LD) containing medical facts using their Stellar secret key.
* **Holder (User):** Stores the raw medical data in a local, encrypted vault.

#### **B. The Proving Layer (Client-Side)**
* **Prover:** A Rust-based engine (using **Noir**) that takes the raw data and the Issuer’s signature as private inputs. It generates a ZK-SNARK proof that the signature is valid and the data meets the required criteria (e.g., "Date < Expiry").

#### **C. The Verification Layer (On-Chain)**
* **Soroban Verifier:** A smart contract on Stellar that holds the verification key. It executes the `pairing_check` to validate the proof submitted by the user.

---

### **3. REPOSITORY STRUCTURE (GITHUB ORGANIZATION)**

1.  **`vitalzk-circuits`**: Contains the Noir/Rust ZK circuits. This is the core logic defining the "rules" of the medical proof.
2.  **`vitalzk-contracts`**: The Soroban smart contracts written in Rust. Includes the `IssuerRegistry` and the `ProofVerifier`.
3.  **`vitalzk-sdk`**: A TypeScript/Rust wrapper that simplifies proof generation and Stellar transaction submission for third-party integrators.
4.  **`vitalzk-mobile-vault`**: A React Native application housing the local Rust prover via UniFFI, acting as the user's "Medical ID."
5.  **`vitalzk-docs`**: Technical specs, API references, and security audit logs.

---

### **4. TECHNICAL STACK**
* **Blockchain:** Stellar (Protocol 25+).
* **Smart Contracts:** Soroban SDK (Rust).
* **ZK Framework:** Noir (Aztec Network) for circuit writing; Barretenberg for backend proving.
* **Cryptography:** BN254 Curve, Poseidon Hashing.
* **Mobile:** React Native with Expo.
* **Interface:** Stellar CLI and Freight Wallet for transaction signing.

---

### **5. IMPLEMENTATION TIMELINE (5-DAY ACCELERATED PLAN)**

| Day | Focus | Deliverables |
| :--- | :--- | :--- |
| **Day 1** | **Protocol Design & Registry** | Setup Github Org; Write `IssuerRegistry` contract in Soroban; Deploy to Testnet. |
| **Day 2** | **ZK Circuit Development** | Define medical data schema; Write Noir circuits for Signature Verification and Range Proofs. |
| **Day 3** | **On-Chain Verifier** | Integrate the generated Noir verification key into a Soroban `Verifier` contract. |
| **Day 4** | **Client-Side Integration** | Build the Rust-to-JS bridge (SDK); Implement proof generation in a local environment. |
| **Day 5** | **End-to-End Testing** | Perform a full "Issue -> Prove -> Verify" flow; Document API endpoints; Finalize READMEs. |

---

### **6. SECURITY CONSIDERATIONS**
* **Nullifiers:** To prevent "double-spending" or replaying the same proof, the protocol implements nullifiers derived from the unique credential hash.
* **Trusted Issuers:** Only hospitals with active, multisig-governed Stellar accounts in the `IssuerRegistry` can have their signatures recognized by the circuit.
* **Data Minimization:** No PII (Personally Identifiable Information) is ever hashed or stored on-chain; only the cryptographic commitment is utilized.

---
