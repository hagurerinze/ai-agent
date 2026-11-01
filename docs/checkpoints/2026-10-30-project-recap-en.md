# AI Agent — Project Checkpoint & Development Status

**Foundation:** Partially completed  
**WhatsApp:** Not connected  
**Development:** Paused

## 1. Project Goals

This project is not just a WhatsApp chatbot. The goal is to build a personal AI agent that can:

- Maintain one consistent persona and memory.
- Use multiple AI providers with fallback support.
- Learn from conversations, documents, research, and exploration.
- Archive WhatsApp conversations.
- Use WhatsApp and the terminal as its interfaces.
- Gradually initiate actions through a permission and approval system.
- Use an internet connection for cloud AI while still managing local storage when offline.

Established architecture decisions:

| Component | Decision |
|---|---|
| Runtime | Node.js |
| Language | JavaScript ES Modules |
| WhatsApp | Baileys |
| Database | SQLite |
| AI | Cloud AI API |
| Storage | Local HDD |
| Local AI | Not planned |
| Interface | WhatsApp + terminal |
| Autonomy | Gradual, permission-based |
| Online/offline | Manual toggle |

## 2. Storage and Memory Policies

Several important decisions have already been established.

### WhatsApp Archive

Archive scope:

- Personal messages.
- Incoming and outgoing messages.
- Bot conversations.
- Group chats.
- Replies and quoted messages.
- Sender, group, and timestamp metadata.

### Media Policy C

- All text and metadata can be archived.
- Media will not be downloaded in bulk.
- Selected media can be retained when relevant to the learning process.
- Images, documents, and audio will be processed according to the defined policy.

### Data Separation

Raw archives are not the same as long-term memory.

The planned workflow:

```text
WhatsApp / Documents / Research
              |
              v
        Raw Archive
              |
              v
       Learning Process
              |
              v
       Memory Extraction
              |
              v
        Long-term Memory
```

Each memory should have provenance, confidence, and status information. Facts, assumptions, and unverified information must remain distinguishable.

Group ingestion must not be enabled without informing and obtaining consent from the relevant participants.

## 3. Existing Project Structure

Project location:

```text
/home/hagure/ai-agent/
```

Structure already created or previously verified:

```text
ai-agent/
│
├── core/
│   ├── index.js
│   ├── init-database.js
│   └── whatsapp-test.js
│
├── providers/
│
├── memory/
│
├── learning/
│
├── autonomy/
│
├── config/
│
├── logs/
│
├── interfaces/
│   ├── whatsapp/
│   │   └── connection.js
│   │
│   └── terminal/
│
├── storage/
│   ├── database/
│   │   └── agent.db
│   │
│   ├── knowledge/
│   │
│   ├── media/
│   │
│   ├── backups/
│   │
│   └── whatsapp-auth/
│       └── creds.json
│
├── node_modules/
│
├── .git/
├── .gitignore
├── package.json
└── package-lock.json
```

**Note:** This structure is based on the recorded development history. It has not been verified against the latest filesystem listing. Empty directories and additional files still need to be checked.

## 4. Important Files and Their Functions

| File | Function |
|---|---|
| `core/index.js` | Main agent entry point; currently a placeholder |
| `core/init-database.js` | Database initialization |
| `core/whatsapp-test.js` | WhatsApp connection test |
| `interfaces/whatsapp/connection.js` | Baileys connection and pairing |
| `package.json` | npm configuration and scripts |
| `package-lock.json` | Dependency version lockfile |
| `.gitignore` | Excludes sensitive files and runtime data |

Scripts in `package.json`:

```json
{
  "start": "node core/index.js",
  "dev": "node --watch core/index.js",
  "db:init": "node core/init-database.js",
  "wa:test": "node core/whatsapp-test.js"
}
```

`core/index.js` does not connect to any AI provider or WhatsApp service yet. It is currently just a placeholder.

## 5. Database Status

Database location:

```text
storage/database/agent.db
```

Previously verified tables:

```text
conversations
knowledge
memories
messages
tasks
events
media_assets
memory_sources
schema_meta
```

SQLite configuration:

- WAL mode enabled.
- Foreign keys enabled per connection.
- Busy timeout: 5000 ms.
- Synchronous mode: NORMAL.
- Database integrity check returns `ok`.

The latest schema change to `memory_sources` uses `ON DELETE CASCADE` for relationships with `memories`, `messages`, and `knowledge`.

Previous integrity check:

```text
integrity_check = ok
```

The database has been successfully initialized and tested. AI integration has not started yet.

## 6. Installed Software

Versions below are based on previous terminal checks, not a fresh system scan.

| Software / Package | Version | Status |
|---|---|---|
| Debian | 13 Trixie | Installed |
| Node.js | 20.19.2 | Installed |
| npm | 9.2.0 | Installed |
| SQLite CLI | 3.46.1 | Installed |
| node-sqlite3 | 5.1.5+ds1-1+b5 | Installed via APT |
| Baileys | 6.7.23 | Installed |
| Python | 3.13.5 | Installed |
| Git | 2.47.3 | Installed |

SQLite module location:

```text
/usr/lib/x86_64-linux-gnu/nodejs/sqlite3/lib/sqlite3.js
```

One technical issue has already been resolved: Debian's `node-sqlite3` package could not be imported using a regular ESM import.

The current workaround:

```js
import { createRequire } from 'node:module';

const require = createRequire(import.meta.url);
const sqlite3 = require('sqlite3');
```

Baileys is installed as a local npm dependency inside the project.

`qrcode-terminal` was installed during the initial experiment and later uninstalled because pairing code was selected instead of QR pairing.

Not yet installed or configured:

- AI provider/API integration.
- Memory extraction system.
- Learning pipeline.
- Autonomy engine.
- Terminal interface.
- WhatsApp event archive integration.
- Scheduler and systemd service.

## 7. WhatsApp Troubleshooting History

This is the last area under investigation.

| Stage | Result |
|---|---|
| Baileys RC14 | Login failed with 401 |
| Pairing API | `requestPairingCode` available on the socket instance |
| Baileys 6.7.23 | WebSocket connection established, login failed with 405 |
| Default WA Web version | Outdated |
| `fetchLatestWaWebVersion()` | Successfully retrieved the latest version |
| Explicit WA Web version | Still failed with 401 |
| Ubuntu/Chrome browser | Failed with 401 |
| Mac OS/Chrome browser | Failed with 401 |

Latest known state:

```text
Baileys: 6.7.23
WhatsApp Web: latest
Browser: Mac OS / Chrome
WebSocket: connected
Login: failed
Disconnect: 401
Pairing code: not successfully requested
```

The latest investigation identified this file:

```text
node_modules/@whiskeysockets/baileys/lib/Utils/validate-connection.js
```

At line 16:

```js
platform: proto.ClientPayload.UserAgent.Platform.WEB,
```

This means changing `Browsers.macOS()` only changes the browser fingerprint. The protocol platform value remains `WEB`.

The investigation was paused while checking the `ClientPayload.UserAgent.Platform` enum in `WAProto.proto` to determine whether `MACOS` is available before deciding whether a patch is necessary.

No changes have been made to the Baileys library.

## 8. Actual Laptop Inventory

There is an important limitation: the project structure and some installed software are known, but a complete filesystem and APT/npm package inventory has not been collected from the laptop.

Therefore, the entire laptop environment should not be considered fully documented yet.

When development resumes, run the following commands to generate an inventory that can be saved as a checkpoint.

### A. Project Files and Directories

```bash
cd ~/ai-agent

find . \
  -path './node_modules' -prune -o \
  -path './.git' -prune -o \
  -printf '%y %p\n' | sort
```

### B. npm Dependencies

```bash
cd ~/ai-agent

npm ls --depth=0
```

### C. Relevant APT Packages

```bash
dpkg-query -W \
  -f='${binary:Package}\t${Version}\n' \
  nodejs npm sqlite3 libsqlite3-0 \
  node-sqlite3 git python3 2>/dev/null
```

### D. Runtime Versions

```bash
node --version
npm --version
sqlite3 --version
python3 --version
git --version
```

### E. Database Status

```bash
cd ~/ai-agent

sqlite3 storage/database/agent.db <<'EOF'
.tables
PRAGMA integrity_check;
SELECT name, sql
FROM sqlite_master
WHERE type = 'table'
ORDER BY name;
EOF
```

These commands do not modify the project. Their output can be used to create a final system inventory.

## 9. Checkpoint Status and Next Development Stages

### Project Foundation

**Partially completed**

The project skeleton, npm configuration, Git repository, and storage structure are available.

### Database Foundation

**Completed for the initial stage**

SQLite, schema, WAL, foreign keys, and integrity checks have been verified.

### WhatsApp Connectivity

**Not completed**

WebSocket connection succeeds, but authentication is rejected with status 401.

### AI Provider and Memory

**Not started**

Waiting for the next foundation stages.

### Learning and Autonomy

**Not started**

No implementation exists yet.

### Recommended Stop Point

Record the following checkpoint:

```text
CHECKPOINT: WhatsApp Connectivity Investigation

Date: 2026-10-30
Project: ~/ai-agent
OS: Debian 13 Trixie
Runtime: Node.js 20.19.2
Baileys: 6.7.23

Completed:
- Project skeleton
- SQLite initialization
- Database schema
- WAL and integrity verification
- Baileys installation
- Pairing API verification
- Latest WhatsApp Web version retrieval
- Browser fingerprint experiment
- ClientPayload platform source inspection

Unresolved:
- WhatsApp login returns 401
- Pairing code not triggered
- ClientPayload Platform remains WEB
- WAProto enum investigation incomplete

Do not:
- Delete WhatsApp auth state without diagnosis
- Patch node_modules without verifying the enum
- Proceed to AI integration yet

Status: PAUSED
```

---

**Current development status:** The initial project and database foundations are in place. WhatsApp connectivity remains unresolved, and all subsequent AI, learning, and autonomy development is paused until the connectivity investigation is resumed.
