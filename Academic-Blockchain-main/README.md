# Academic Blockchain PBL — 3-Organization Architecture

A tamper-evident academic certificate issuance, verification, and revocation system built on **Hyperledger Fabric 2.5 (LTS)** and **IPFS**, containerized with **Docker** for complete cross-platform portability between **macOS** (Apple Silicon / Intel) and **Ubuntu Linux**.

---

## 🏛️ Architecture Overview

The network is governed by a **3-Organization Consortium** operating on a shared channel (`academicchannel`) with **Raft** crash fault-tolerant ordering:

```
+-----------------------------------------------------------------------------------+
|                           ACADEMIC BLOCKCHAIN CONSORTIUM                          |
|                                                                                   |
|  +------------------------+  +------------------------+  +---------------------+  |
|  |         Org 1          |  |         Org 2          |  |        Org 3        |  |
|  |  University / Faculty  |  |   Examination Board    |  |  Admin & Verifier   |  |
|  +------------------------+  +------------------------+  +---------------------+  |
|  | • peer0.org1:7051      |  | • peer0.org2:8051      |  | • peer0.org3:9051   |  |
|  | • CouchDB0: 5984       |  | • CouchDB1: 6984       |  | • CouchDB2: 7984    |  |
|  | • MSP: Org1MSP         |  | • MSP: Org2MSP         |  | • MSP: Org3MSP      |  |
|  | • Faculty/HOD/DAC      |  | • Exam Grade Lockdown  |  | • Dean/Admin/Issue  |  |
|  +------------------------+  +------------------------+  +---------------------+  |
|               \                          |                         /              |
|                \                         |                        /               |
|                 +------------------------------------------------+                |
|                 |     Channel: academicchannel (Fabric 2.5)      |                |
|                 +------------------------------------------------+                |
|                                          |                                        |
|                 +------------------------------------------------+                |
|                 | Raft Consensus: orderer.academic.edu:7050      |                |
|                 | Decentralized Storage: ipfs-node:5001 / 8080   |                |
|                 +------------------------------------------------+                |
+-----------------------------------------------------------------------------------+
```

### The 4 Modules (Preserving the Baseline)

| Module | Component | Description & Responsibilities |
|---|---|---|
| **Module 1** | **Registration & Requests** | Student onboarding (`RegisterStudent`), request submission (`RequestCertificate`), profile validation. |
| **Module 2** | **7-Stage Approval Workflow** | State machine enforced by smart contract across organizations: `SUBMITTED` → `FACULTY_APPROVED` → `HOD_APPROVED` → `DAC_APPROVED` (Org 1) → `EXAM_LOCKED` (Org 2) → `DEAN_APPROVED` → `ADMIN_FINALIZED` (Org 3). |
| **Module 3** | **PDF & IPFS Storage** | Generates official certificate PDF, uploads to IPFS Kubo node, retrieves IPFS CID, and computes SHA-256 fingerprint. |
| **Module 4** | **Distributed Fabric Network** | Anchors CID and SHA-256 onto Fabric ledger. Powers public verification (`VERIFIED` / `REVOKED` / `INVALID`) and immutable revocation. |

---

## 🚀 Quick Start (macOS & Ubuntu)

### Prerequisites
- **Docker** (v24.0+ or Docker Desktop)
- **Docker Compose** (v2.20+)
- **Git**

No host Fabric binaries (`cryptogen`, `configtxgen`) or Go compiler are required on your host machine — all cryptographic tools and chaincode packaging run inside official Docker containers.

### 1. Clone & Start the Project
```bash
git clone <repo-url> "Academic Blockchain"
cd "Academic Blockchain"

# One-command startup (Generates crypto, boots 3-Org network, channel, chaincode, API & UI)
./start.sh
```

### 2. Access the Applications
- **Web Verification & Management Portal**: [http://localhost:3000](http://localhost:3000)
- **Backend REST API**: [http://localhost:4000/api/health](http://localhost:4000/api/health)
- **Local IPFS Gateway**: [http://localhost:8080](http://localhost:8080)

---

## 🧪 Automated Testing

Execute the complete end-to-end integration test exercising all 4 modules (Student onboarding → 7-stage approval workflow → PDF generation → IPFS CID upload → Fabric commit → Verification → Revocation):

```bash
./test-flow.sh
```

---

## 💻 Running on Ubuntu Linux

Because this folder is shared with or cloned to an Ubuntu machine, the project is pre-configured with:
1. `.gitattributes` enforcing Unix LF line endings across all shell scripts and YAML files.
2. Official multi-arch Docker images (`linux/amd64` and `linux/arm64`).
3. Zero host binary dependencies (all crypto and channel tools run in Docker).

To run on Ubuntu:
```bash
# Ensure Docker service is running and user is in docker group
sudo usermod -aG docker $USER
newgrp docker

# Run the master start script
./start.sh
```

---

## 🛠️ Individual Network Operations

| Command | Action |
|---|---|
| `./start.sh` | Bring up entire 3-org stack, channel, chaincode, API, and UI |
| `./stop.sh` | Tear down all containers, volumes, and networks cleanly |
| `./test-flow.sh` | Run automated end-to-end test suite |
| `./network/network.sh status` | View container health across all 3 organizations |
| `./network/network.sh down` | Stop only the Fabric network and IPFS infrastructure |
| `./network/network.sh up` | Start only the Fabric network and deploy chaincode |

---

## 📁 Repository Structure

```
Academic Blockchain/
├── .gitattributes              # Guarantees LF line endings on Mac & Linux
├── start.sh                    # One-command master runner
├── stop.sh                     # Clean shutdown script
├── test-flow.sh                # End-to-end automated test runner
├── README.md                   # Documentation and runbook
├── docker/
│   ├── docker-compose-net.yaml # Orderer, Org1, Org2, Org3, CouchDBs, IPFS, CLI
│   ├── docker-compose-app.yaml # Express Backend API & Frontend Portal
│   └── configtx/
│       └── configtx.yaml       # Channel, consortium & Raft ordering spec
├── network/
│   ├── crypto-config.yaml      # 3-Org cryptographic identities
│   ├── network.sh              # Network manager (up, down, status)
│   └── scripts/
│       ├── envVar.sh           # Organization CLI environment loader
│       ├── generate-crypto.sh  # Dockerized cryptogen & genesis block builder
│       ├── create-channel.sh   # Channel participation join script
│       └── deploy-chaincode.sh # Fabric 2.5 chaincode lifecycle script
├── chaincode/
│   └── academic-contract/      # Smart contract implementing 7-stage lifecycle
│       ├── package.json
│       └── index.js
├── backend/                    # Express REST API
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── fabric.js           # Hyperledger Fabric ledger client
│       ├── ipfs.js             # IPFS Kubo client & SHA-256 calculation
│       ├── pdf.js              # Academic Certificate PDF generator
│       ├── intentEngine.js     # Contributor 5.2: Conversational NLP & Intent Engine
│       ├── test_intent_engine.js # Automated Chatbot & Intent Engine test suite
│       └── server.js           # REST endpoints (including POST /api/chat)
└── frontend/                   # Web Verification & Management Portal
    ├── Dockerfile
    ├── server.js
    └── public/
        ├── index.html          # Interactive UI
        ├── app.js              # Workflow stepper, IPFS & Verification logic
        └── styles.css          # Responsive dashboard styles
```

---

## 🤖 Group 5: Conversational AI Assistant & Intent Engine (Contributor 5.2)

Contributor 5.2 implements the **Conversational NLP & Intent Engine**, mapping free-text user questions into structured blockchain queries and generating natural-language replies.

### 1. Supported Core Intents & Example Queries

| Intent | Purpose | Example Queries |
|---|---|---|
| `STATUS_CHECK` | Reports current stage from 7 canonical stages + genuine `txId`. | • "What's the status of CERT-101?"<br>• "Where is certificate CERT-102 currently at?"<br>• "What stage is CERT-103 in right now?" |
| `HISTORY_LOOKUP` | Returns complete lifecycle history in chronological order. | • "Show me the history of CERT-101"<br>• "Give me the full audit trail for CERT-102"<br>• "What stages did CERT-101 pass through?" |
| `WHO_APPROVED` | Identifies approver, org, and timestamp for lifecycle milestones. | • "Who approved CERT-101?"<br>• "Who signed off on CERT-102?"<br>• "Which actor authorized CERT-103?" |
| `GENERAL_HELP` | Explains assistant capabilities and supported queries. | • "What can you do?"<br>• "Help"<br>• "List supported commands" |

### 2. REST API Endpoint (`POST /api/chat`)

**Request (JSON):**
```json
{
  "message": "What is the status of CERT-101?"
}
```

**Response (Contract B):**
```json
{
  "reply": "CERT-101 is currently at HOD_APPROVED stage, confirmed on-chain (tx: a1b2c3d4e5f67890...).",
  "intent": "STATUS_CHECK",
  "data": {
    "certId": "CERT-101",
    "status": "HOD_APPROVED",
    "org": "Org1MSP",
    "action": "ApproveHOD",
    "txId": "a1b2c3d4e5f67890123456789abcdef0123456789abcdef0123456789abcdef0",
    "timestamp": "2026-09-27T10:00:00Z",
    "actor": "hod.org1",
    "history": [ ... ]
  }
}
```

### 3. Contract A Dependency Fields

The engine consumes the following fields provided by Contributor 5.1's `getCertHistory(certId)`:
- `certId` *(string)*: Unique certificate identifier (e.g., `"CERT-101"`).
- `status` *(string)*: Current canonical stage (e.g., `"HOD_APPROVED"`).
- `org` *(string)*: Hyperledger Fabric MSP ID (e.g., `"Org1MSP"`, `"Org2MSP"`, `"Org3MSP"`).
- `action` *(string)*: Chaincode function invoked (e.g., `"ApproveHOD"`).
- `txId` *(string)*: Real Fabric transaction ID (quoted to the user for tamper-proof credibility).
- `timestamp` *(ISO string)*: Transaction commit timestamp.
- `actor` *(string)*: Authenticated identity that submitted the transaction.
- `history` *(array)*: Chronological array of all past state transitions.

### 4. Single-Line Swap Point for Contributor 5.1 Integration

In [backend/src/intentEngine.js](file:///Users/anmolpainuly/Downloads/Academic-Blockchain-main/backend/src/intentEngine.js#L78-L92):
```javascript
// To swap from the Contract A STUB to 5.1's live ledger function:
const { getCertHistory } = require('./fabric');

async function fetchCertHistory(certId) {
  const normalizedId = (certId || '').trim().toUpperCase();
  return await getCertHistory(normalizedId);
}
```

### 5. Running the Intent Engine Test Suite
```bash
node backend/src/test_intent_engine.js
```

