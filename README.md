# VoteChain: A Verifiable Encrypted E-Voting Framework with Candidate-Blind Biometric Authorization

VoteChain is an experimental electronic voting framework that integrates **candidate-blind biometric authorization**, **zero-knowledge validated encrypted ballot submission**, and **threshold-verifiable encrypted tally recovery** within a blockchain-supported voting workflow.

The system separates identity-aware biometric authorization from candidate selection and private ballot preparation. After successful biometric authorization, protected election-specific credential material is released to the voter-facing client. Candidate selection, encrypted ballot construction, and zero-knowledge proof generation are then performed without requiring the biometric authorization service to receive the selected candidate.

This repository contains the implementation, experimental artifacts, reproducibility material, smart contracts, zero-knowledge circuits, cryptographic components, backend services, frontend application, biometric modules, trustee tooling, and evaluation data used in the current VoteChain research project.

---

## Research Objective

Electronic voting systems must address several requirements simultaneously:

- reliable voter authorization
- ballot confidentiality
- eligibility verification
- duplicate-vote prevention
- verifiable ballot admissibility
- auditable tally recovery

VoteChain investigates how these requirements can be integrated while maintaining a clear separation between the identity-aware authorization layer and the private ballot-processing layer.

The current framework focuses on three main components:

1. **Candidate-blind biometric authorization**
2. **Zero-knowledge validated encrypted ballot submission**
3. **Threshold-verifiable encrypted tally recovery**

---

## System Overview

The current VoteChain workflow can be summarized as:

```text
Voter registration
        ↓
Election-specific eligibility commitment
        ↓
Biometric authorization
        ↓
Protected election-specific credential material released
        ↓
Candidate selection remains inside the voter-facing client
        ↓
Encrypted ballot construction
        ↓
Groth16 proof generation
        ↓
On-chain proof and duplicate-vote validation
        ↓
Encrypted ballot acceptance
        ↓
Homomorphic accumulation of accepted encrypted contributions
        ↓
Voting closes
        ↓
Three-of-five trustee tally-recovery procedure
        ↓
Independent verification of trustee evidence
        ↓
Supported on-chain tally finalization
        ↓
Final counts and transcript commitment recorded
```

The selected candidate is not required by the biometric authorization request.

---

## Core Security Design

### Candidate-Blind Biometric Authorization

Biometric verification remains identity-aware because the authorization service must determine whether an eligible voter has successfully authenticated.

However, the authorization boundary is separated from candidate selection.

After successful authorization:

- protected election-specific credential material is released
- candidate selection remains in the voter-facing client
- private ballot preparation occurs after authorization
- the biometric authorization request does not require the selected candidate

This design therefore limits what the identity-aware authorization service needs to learn about the voting decision.

---

### Election-Specific Eligibility

Eligible voters are represented through election-specific cryptographic commitments.

The implemented voting relation includes:

- private eligibility membership
- an election-specific commitment root
- election-context binding
- election-scoped nullifier derivation
- duplicate-vote protection

The nullifier permits the system to reject reuse within the same election without placing the plaintext candidate choice into the duplicate-vote mechanism.

---

### Zero-Knowledge Ballot Validation

VoteChain uses a custom **Circom 2 + Groth16** voting relation.

The proof relation combines checks for:

- eligibility membership
- election-scoped nullifier correctness
- election-context consistency
- valid single-choice voting
- consistency between the private vote and its encrypted ballot representation

The smart contract verifies the supported Groth16 relation before accepting the encrypted ballot.

The plaintext candidate selection is not supplied to the voting contract as the ballot value.

---

### Encrypted Ballot Representation

The implemented ballot design uses a fixed **16-slot BabyJub exponential-ElGamal representation**.

The fixed-width representation supports elections with up to 16 active candidate slots while maintaining a common ballot structure across the evaluated candidate-count configurations.

Accepted encrypted candidate contributions are accumulated homomorphically.

The system does not require intermediate plaintext candidate totals to be published while voting remains active.

---

### Threshold-Verifiable Tally Recovery

After voting ends, the encrypted aggregate tally is recovered through an implemented **three-of-five trustee procedure**.

The tally-recovery workflow supports:

- encrypted aggregate recovery
- partial-decryption contributions
- independently checkable trustee evidence
- final tally validation
- commitment to the finalization evidence

Trustee evidence is checked through the supported external verification workflow.

The system does **not** claim that every trustee-evidence verification step is executed directly by the smart contract.

The final supported tally values and a commitment to the verification transcript are recorded on-chain.

---

## Trust and Privacy Boundaries

The implementation is intended to make its trust assumptions explicit.

VoteChain does **not** claim:

- universal voter anonymity
- authority-blind unlinkability under every operating condition
- receipt-freeness
- coercion resistance
- security against arbitrary trustee collusion
- that all trustee verification is performed on-chain

The biometric authorization layer remains identity-aware.

The privacy objective is instead to separate that identity-aware authorization step from the candidate-selection and encrypted-ballot-processing path.

The current evaluation is performed in a **local Anvil environment**. It should not be interpreted as a public Ethereum deployment or public-network performance measurement.

---

## Architecture

The repository combines the following functional layers.

### Voter-Facing Client

The React/TypeScript frontend supports:

- election interaction
- biometric authorization workflow
- candidate selection
- private ballot preparation
- proof-generation integration
- encrypted ballot submission
- election-state and result viewing

Primary path:

```text
votechain-frontend-ts/
```

---

### Backend and Election Services

The Node.js/Express backend contains services for:

- election lifecycle management
- candidate and voter registration workflows
- election-specific membership handling
- biometric authorization coordination
- credential handling
- blockchain interaction
- ZK-related application services
- trustee and finalization workflows
- administrative authorization controls

Primary path:

```text
votechain-central-server/
```

---

### Smart Contracts

The current encrypted-ballot contract implementation and associated verifier integration are located under:

```text
contracts/wp4-foundry/
```

Important sources include:

```text
contracts/wp4-foundry/src/VoteChainWP4.sol
contracts/wp4-foundry/src/WP4Profile.sol
contracts/wp4-foundry/src/verifiers/
```

The contract layer supports the voting lifecycle, Groth16 ballot validation, encrypted contribution handling, election-state controls, and supported finalization conditions.

---

### Zero-Knowledge Circuits

Current circuit and proof-related material is maintained under:

```text
circuits/
crypto/
```

The final research architecture uses the custom Circom/Groth16 voting relation rather than treating the earlier Semaphore-oriented prototype as the final voting construction.

---

### Trustee Tooling

Threshold tally-recovery material is maintained under:

```text
trustee-tools/
```

Associated election finalization and trustee services also appear in the backend implementation.

---

### Biometric Components

The inherited biometric implementation includes face and fingerprint components:

```text
votechain-face-recognition/
votechain-fingerprint/
```

These modules support the identity-aware authorization stage.

Candidate selection and private ballot construction are intentionally separated from this stage in the current architecture.

---

### Experimental and Research Artifacts

Experimental records and publication-oriented evaluation artifacts are maintained under:

```text
experiments/
research/
snapshots/
diagnostics/
```

These directories contain benchmark runs, evidence records, validation outputs, configuration material, result synthesis, provenance information, and reproducibility artifacts.

---

## Repository Structure

```text
.
├── circuits/                    # Zero-knowledge circuit material
├── contracts/
│   └── wp4-foundry/             # Current Foundry contract implementation
│       ├── src/
│       ├── test/
│       └── gas/
├── crypto/                      # Cryptographic implementation/support material
├── diagnostics/                 # Diagnostic evidence
├── experiments/                 # Experimental campaigns and measured results
├── images/                      # Project images
├── lib/                         # Vendored development dependencies
├── research/                    # Research, ZKP and reproducibility records
├── script/                      # Foundry deployment scripts
├── scripts/                     # Local setup and utility scripts
├── snapshots/                   # Deployment/runtime snapshots
├── src/                         # Earlier/legacy Solidity implementation paths
├── test/                        # Earlier/legacy Foundry tests
├── trustee-tools/               # Trustee/tally-recovery tooling
├── votechain-central-server/    # Node.js/Express backend
├── votechain-face-recognition/  # Face-recognition module
├── votechain-fingerprint/       # Fingerprint module
├── votechain-frontend-ts/       # React/TypeScript frontend
├── package.json
└── README.md
```

---

## Legacy Implementation Material

The repository retains earlier engineering paths for traceability and development provenance.

In particular, some files under:

```text
src/
test/
research/zkp/
```

contain earlier Semaphore-oriented implementation material.

These files should not automatically be interpreted as the final cryptographic architecture described by the current VoteChain study.

The current research design uses the custom Circom 2 + Groth16 relation, election-specific eligibility construction, encrypted ballot representation, and threshold tally-recovery workflow represented by the later implementation and experimental artifacts.

Legacy files are retained because they document the engineering progression of the project.

---

## Experimental Evaluation

The current experimental campaign evaluated:

- **2,000 accepted encrypted-ballot transactions**
- **25 elections**
- eligible-set sizes of **10, 100, 1,000, and 10,000 commitments**
- candidate configurations of **2, 4, 8, and 16 active candidates**

The evaluation was conducted using a local Anvil blockchain environment.

### Ballot Submission

Mean encrypted-ballot submission cost:

```text
1,314,307.123 gas
```

Mean local submission-to-receipt latency:

```text
760.640 ms
```

### Eligible-Set Scaling

The evaluated eligible-set configurations retained a fixed depth-17 membership relation.

Mean Groth16 proving time ranged from:

```text
5.292 s to 5.423 s
```

across eligible-set sizes from 10 to 10,000 commitments.

### Candidate-Count Scaling

Candidate-count experiments covered:

```text
2, 4, 8, and 16 active candidates
```

Transaction gas remained nearly unchanged across these configurations because the implemented encrypted ballot uses a fixed 16-slot representation.

Detailed experimental evidence is retained under the `experiments/` hierarchy.

---

## Technologies

### Blockchain and Smart Contracts

- Solidity
- Foundry
- Forge
- Anvil
- Ethers.js

### Zero-Knowledge and Cryptography

- Circom 2
- Groth16
- snarkjs
- Poseidon-based commitment/Merkle components
- BabyJub cryptographic operations
- exponential-ElGamal ballot representation

### Frontend

- React
- TypeScript
- Vite
- Tailwind CSS
- Ethers.js
- wagmi / viem
- snarkjs

### Backend

- Node.js
- Express.js
- MongoDB / Mongoose
- Ethers.js
- Circom-compatible cryptographic utilities

### Biometric Components

- Python
- face-recognition pipeline
- OpenCV/associated face-processing components
- ESP32/R307-oriented fingerprint integration

---

## Prerequisites

The complete research environment uses several toolchains.

Typical prerequisites include:

- Node.js
- npm
- Foundry
- Forge
- Anvil
- Circom 2
- snarkjs
- Python
- MongoDB
- a browser wallet where required by the frontend workflow

Biometric hardware and its associated software environment are required only when exercising the corresponding biometric path.

---

## Install JavaScript Dependencies

From the repository root:

```bash
npm install
```

Backend:

```bash
cd votechain-central-server
npm install
```

Frontend:

```bash
cd votechain-frontend-ts
npm install
```

---

## Build and Test the Current Contract Path

From the repository root:

```bash
cd contracts/wp4-foundry
forge build
forge test
```

The current paper-facing contract path is under `contracts/wp4-foundry/`.

The root-level `src/` directory contains earlier implementation material and should not be confused with the final contract path.

---

## Backend Tests

```bash
cd votechain-central-server
npm test
```

For development execution:

```bash
npm run dev
```

or:

```bash
npm start
```

Runtime configuration must be supplied for the intended local election environment.

---

## Frontend Build

```bash
cd votechain-frontend-ts
npm install
npm run build
```

For development:

```bash
npm run dev
```

---

## ZK Utility Scripts

The repository root exposes several ZK-related helper commands:

```bash
npm run setup:zk
npm run sync:zk-abi
npm run zk:poc
npm run zk:e2e
```

Some of these paths originate from earlier development phases and may require the corresponding historical dependencies or runtime environment.

For the current architecture, consult the later circuit, contract, backend, and experiment artifacts before treating a legacy helper script as authoritative.

---

## Local Blockchain Utilities

Local-chain helper scripts are available under:

```text
scripts/
```

including Anvil-oriented startup and cleanup utilities.

The reported research measurements were collected from controlled local experiments. Public-chain deployment characteristics were not evaluated as part of the reported performance results.

---

## Reproducibility Material

The repository preserves extensive experimental evidence rather than only summarized manuscript values.

Relevant locations include:

```text
experiments/wp8/
research/
diagnostics/
snapshots/
```

The WP8 experiment hierarchy contains publication-oriented result synthesis, validation records, evidence manifests, provenance records, and reproducibility-package material associated with the reported evaluation.

This material is intended to support independent inspection of the reported measurements and their provenance.

---

## Development and Repository Provenance

This standalone repository was extracted from the VoteChain research workspace at:

```text
UGC_Project/VC-W06/VoteChain-ZK
```

The standalone repository places the VoteChain-ZK project contents directly at the repository root.

Earlier VoteChain development history is also available at:

```text
https://github.com/rifat87/VoteChain
```

The retained `VC-W06_*` reports and similarly named artifacts document earlier work-package stages and are preserved for research provenance.

---

## Research Context

The current VoteChain framework extends an earlier blockchain-integrated biometric electronic-voting architecture.

The important distinction is:

```text
Earlier baseline
    Identity-aware biometric / IoT voting architecture

Current VoteChain framework
    Candidate-blind authorization boundary
    +
    zero-knowledge validated encrypted ballots
    +
    threshold-verifiable encrypted tally recovery
```

The current work therefore should not be interpreted as merely replacing the voting interface of the earlier architecture. Its primary research focus is the privacy and verifiability boundary between voter authorization, private ballot construction, cryptographic ballot admission, and encrypted tally recovery.

---

## Current Research Title

**VoteChain: A Verifiable Encrypted E-Voting Framework with Candidate-Blind Biometric Authorization**

---

## Important Research Disclaimer

This repository is a research prototype and experimental implementation.

The reported measurements are based on controlled local experiments and do not establish production readiness for governmental or public-election deployment.

Real-world election deployment would require additional independent security review, operational controls, deployment-specific threat analysis, governance procedures, infrastructure hardening, key-management procedures, usability evaluation, and jurisdiction-specific election compliance.
