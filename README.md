# Public Session Manager

> **Privacy-Preserving Session Monitoring, Cryptographic Audit Trails & Automated Workstation Sanitization**  
> Designed for public and shared Windows workstations in university labs, libraries, cyber cafes, and test centers.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-25%2B-orange.svg)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2.3-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![React](https://img.shields.io/badge/React-18-cyan.svg)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-5.1-purple.svg)](https://vitejs.dev/)

---

## 📌 Executive Summary

Shared workstations pose significant security and privacy dilemmas:
- **User Privacy Concerns**: Users hesitate to use public computers if intrusive monitoring (keylogging, screen capture, full file indexing) is deployed.
- **Administrative Security Risks**: Public machines are vulnerable to malware, data persistence across sessions, untracked modifications, and credential theft.
- **Regulatory Compliance**: Systems must comply with privacy mandates such as **GDPR** by design (data minimization, purpose limitation, rolling retention).

**Public Session Manager** solves this dilemma through **Zero-Content Privacy-Preserving Monitoring**, **Tamper-Evident Cryptographic Hash Chaining**, and **Automated Reversible Workstation Quarantine**.

---

## 🚀 Key Features & Architectural Novelties

### 1. Privacy-Preserving Audit Recording (Zero-Content Intrusiveness)
- **Metadata-Only Tracking**: Collects only process names, CPU/RAM utilization, file extensions, and data volume metrics.
- **Zero Content Scraping**: **No keystrokes logged, no screen captures, no document contents inspected**.
- **Cryptographic Identifier Masking**: Sensitive paths (file system paths, executable paths, IP addresses) are irreversibly hashed with **SHA-256** before leaving the workstation.

### 2. Tamper-Proof HMAC-SHA256 Hash Chain
- Every session event is cryptographically linked to its predecessor using a continuous **HMAC-SHA256 chain**:
  $$\mathcal{H}_n = \text{HMAC-SHA256}(\mathcal{H}_{n-1}, \text{Metadata}_n)$$
- Guarantees **immutable, legally admissible audit records**. Any log modification, truncation, or insertion breaks the chain and triggers immediate security alerts.

### 3. End-to-End Cryptographic Protection
- Event batches are encrypted locally with **AES-256 Fernet** prior to transit.
- Decrypted server-side only in memory by the Spring Boot backend (`SessionRecordingService`).
- Admin endpoints protected with **HMAC-SHA256 signed JWT Bearer Tokens**.
- Workstation communication secured via custom API Key headers (`X-Client-Api-Key`).

### 4. Browser Isolation & Reversible Quarantine
- **Session-Isolated Downloads**: Automatically redirects Google Chrome and Microsoft Edge download paths to:
  `Downloads\SecureSessionManager\<machine_id>\<session_id>\`
  ensuring downloads from one user are isolated from others and cleanly handled.
- **Reversible Local Quarantine**: Post-session or on lock, files are moved into a secure local quarantine store (`%LOCALAPPDATA%\SecureSessionManager\Quarantine`).
- **Admin-Managed Remediation**: Administrators can review quarantined items from the web dashboard, selectively restore legitimate files, or authorize permanent secure deletion.
- **Clipboard Sanitization**: Flushes the Windows clipboard on session termination.

### 5. Automated Session Lifecycle & Emergency Panic Lock
- **Win32 Input Detection**: Tracks idle duration using native Windows `GetLastInputInfo`.
- **Automated Termination**: Automatically logs out and sanitizes workstation state after configured idle limits.
- **Remote / Local Panic Lock**: Instantly sanitizes temporary buffers and invokes Windows `LockWorkStation` remotely via the admin console or locally via panic triggers.

### 6. GDPR Compliance & Data Retention
- Built-in database function purges audit events after **30 days** automatically.
- Dedicated dashboard panel for compliance metrics, tracking retention adherence and privacy guarantees.

---

## 🏛️ System Architecture

```text
  +-------------------------------------------------------------+
  |              Shared Windows Workstation (Client)            |
  |                                                             |
  |  +--------------------+   +-------------------------------+ |
  |  | Process, Network & |   | Win32 Session & Idle Monitor  | |
  |  | File Monitors      |   | (Auto Logout / Panic Lock)    | |
  |  +---------+----------+   +---------------+---------------+ |
  |            |                              |                 |
  |  +---------v------------------------------v---------------+ |
  |  | Session Recorder (SHA-256 Masking & HMAC Hash Chain)  | |
  |  +-------------------------+------------------------------+ |
  |                            | AES-256 Fernet Encrypted       |
  |  +-------------------------v------------------------------+ |
  |  | Quarantine Engine (Browser Downloads & Temp Isolation) | |
  |  +--------------------------------------------------------+ |
  +----------------------------+--------------------------------+
                               | HTTPS / WSS
                               v
  +-------------------------------------------------------------+
  |           Spring Boot 3 / Java 25+ Backend Server          |
  |                                                             |
  |  +--------------------------------------------------------+ |
  |  | SessionRecordingService (AES Decrypt & HMAC Verify)   | |
  |  +--------------------------------------------------------+ |
  |  | REST APIs: /api/sessions, /api/events, /api/audit      | |
  |  | WebSocket Broker: /topic/sessions, /topic/alerts       | |
  |  | Spring Security (HMAC-SHA256 JWT & Client API Filter) | |
  |  +-------------------------+------------------------------+ |
  |                            |                                |
  |            +---------------+---------------+                |
  |            v                               v                |
  |  +-------------------+           +-----------------------+  |
  |  | PostgreSQL / H2   |           | React 18 Admin Console|  |
  |  | 30-Day Retention  |           | (Live Stream & Panic) |  |
  |  +-------------------+           +-----------------------+  |
  +-------------------------------------------------------------+
```

---

## 📂 Project Structure

```text
secure_session_manager/
├── client/                     # Python Windows client agent
│   ├── monitors/               # Process, network, and file activity monitors
│   │   ├── file_monitor.py
│   │   ├── network_monitor.py
│   │   └── process_monitor.py
│   ├── api_client.py           # Encrypted REST communication with backend
│   ├── cleanup_engine.py       # Local cleanup and file sanitization
│   ├── quarantine_manager.py   # Reversible quarantine repository
│   ├── session_downloads.py    # Chrome/Edge session download router
│   ├── session_monitor.py      # Win32 session lifecycle and idle detector
│   ├── session_recorder.py     # Privacy-preserving recorder & HMAC chain
│   ├── main.py                 # Client daemon entry point
│   └── requirements.txt        # Python client dependencies
├── server/                     # Spring Boot enterprise backend
│   ├── src/main/java/com/sessionmanager/
│   │   ├── config/             # Security, CORS, and WebSocket configuration
│   │   ├── controller/         # Session, Event, Audit, and Admin endpoints
│   │   ├── model/              # Entities, DTOs, and session status enums
│   │   ├── repository/         # Spring Data JPA repositories
│   │   ├── security/           # JWT provider & API Key authentication filters
│   │   ├── service/            # SessionRecording, Alert, and Crypto services
│   │   └── util/               # Hash chain validator & cryptography helpers
│   ├── src/main/resources/     # application.yml and profile configs (dev/prod)
│   ├── pom.xml                 # Maven configuration and dependencies
│   └── server_runner.py        # Python FastAPI dev runner for quick offline testing
├── dashboard/                  # React 18 + TypeScript Admin Dashboard
│   ├── src/
│   │   ├── components/         # Active sessions, Audit viewer, Quarantine, Panic Lock
│   │   ├── services/           # REST API client and WebSocket streaming client
│   │   ├── types/              # TypeScript interface definitions
│   │   ├── App.tsx             # Root dashboard view
│   │   └── main.tsx            # Application entry point
│   ├── package.json
│   └── vite.config.ts
├── database/                   # Database scripts
│   ├── schema.sql              # PostgreSQL DDL with 30-day retention trigger
│   └── seed_data.sql           # Demo sessions and hash chain records
├── tests/                      # Python automated test suite
│   ├── test_e2e_flow.py        # Full end-to-end integration flow
│   ├── test_session_recorder.py# Hash chain integrity and privacy tests
│   ├── test_quarantine_manager.py
│   ├── test_cleanup_engine.py
│   └── ...
├── start-local.ps1             # One-click launch script for local development
└── README.md
```

---

## 🛠️ Quick Start & Local Setup

### Prerequisites
- **OS**: Windows 10/11 (client agent relies on Win32 APIs)
- **Python**: 3.10+
- **Node.js**: 18+ and npm
- **Java**: JDK 21+ / 25+ with Maven (for Spring Boot backend)

---

### Option A: One-Click Local Demo

Run the automated launcher script from PowerShell:

```powershell
.\start-local.ps1
```

This will automatically:
1. Start the local backend server on `http://localhost:8443`
2. Start the Vite React dashboard on `http://localhost:3000`
3. Pre-configure demo access with `admin` / `admin`

---

### Option B: Running Components Manually

#### 1. Start the Spring Boot Backend

Set the required `JWT_SECRET` environment variable (minimum 32 characters) and run:

```powershell
# In PowerShell:
$env:JWT_SECRET = "your-custom-secure-jwt-signing-key-32-chars-min"

cd server
mvn spring-boot:run
```

*(Alternatively, run `python server_runner.py` for rapid testing without Java).*

#### 2. Start the React Dashboard

```powershell
cd dashboard
npm install
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) and sign in using `admin` / `admin`.

#### 3. Run the Client Agent

```powershell
cd client
pip install -r requirements.txt

# Run a test session
python main.py --test-login
```

---

## 🔒 Security & Environment Variables

| Variable | Description | Default / Example |
| :--- | :--- | :--- |
| `JWT_SECRET` | Secret key for signing admin authentication tokens (min 32 chars) | *Must be set via environment* |
| `CLIENT_API_KEY` | Workstation client authentication header token | `secure-client-api-key-2026-public-lab` |
| `AES_KEY` | AES-256 encryption key for event payload intake | Base64-encoded 32-byte key |
| `HMAC_KEY` | HMAC-SHA256 integrity verification secret | Base64-encoded secret |
| `ADMIN_USERNAME` | Administrator dashboard username | `admin` |
| `ADMIN_PASSWORD` | Administrator dashboard password | `admin` |

---

## 🧪 Running Automated Tests

Run the complete Python test suite:

```powershell
pytest tests/ -v
```

Run Spring Boot unit and integration tests:

```powershell
cd server
mvn test
```

---

## 📜 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
