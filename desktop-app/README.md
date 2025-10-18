# SkyForge — Build and Delivery Specification

**Purpose:**  
SkyForge is a Windows-first, GPU-ready, zero-knowledge desktop wrapper for WebODM.  
This document defines *what* must be built, *how* it should behave, and *what constitutes a complete deliverable*.  
Audience: Human (Supervisor) + Claude-Code (Builder) operating inside Cursor IDE.

---

## 1. Product Definition

**Goal:**  
Deliver a signed Windows installer that installs and configures the entire WebODM stack with minimal user interaction.

**Core Outcomes**
- Single MSI installer (WiX-based) that sets up SkyForge under `C:\ProgramData\SkyForge`
- Electron-based Control App that hides all Docker/WSL2 complexity
- Secure, local-only default runtime with optional GPU acceleration
- Compliance with AGPL-3.0 redistribution rules

**User Experience Priorities**
1. Zero-knowledge setup (no Docker or CLI exposure)
2. Clear start/stop controls and live status feedback
3. Persistent data and simple backup/restore
4. Optional autostart and LAN exposure, both opt-in
5. Professional polish (signed binaries, Windows integration)

---

## 2. System Architecture

### High-Level Components

| Component | Purpose | Location / Notes |
|------------|----------|------------------|
| **Installer (MSI/WiX)** | Deploy runtime files and Control App | `C:\ProgramData\SkyForge` |
| **Control App (Electron)** | User-facing GUI managing Docker stack | Uses `docker` CLI + PowerShell scripts |
| **Runtime (Docker Desktop + WSL2)** | Runs WebODM, NodeODM, and ODM containers | Compose manifests under `docker/` |
| **GPU Path (Optional)** | Enables GPU processing via NVIDIA + WSL2 | Controlled in Settings |
| **Security Plugin (Lockdown)** | Optional WebODM plugin restricting admin features | API callable from Control App |

### Runtime Flow
1. User launches Control App  
2. Control App verifies Docker/WSL2 health  
3. `docker compose up` launches WebODM/NodeODM stack  
4. Control App polls REST endpoints for health/status  
5. User accesses WebODM UI via default browser  
6. Data persists under host folders for transparency  

### File Structure (Target State)
C:\ProgramData\SkyForge
├── docker
│ ├── docker-compose.yml
│ ├── config
│ ├── media
│ └── db
├── ControlApp
├── logs
└── backups\


---

## 3. Installation and Runtime Specification

### Installer Responsibilities
- Drop runtime and manifest files to `C:\ProgramData\SkyForge`
- Create data folders with secure ACLs
- Add Start Menu + desktop shortcuts
- Register firewall rule (localhost only)
- Offer autostart options:
  - Launch Control App at logon  
  - Register Windows service (enterprise mode)
- Trigger PowerShell post-install script:
  - Validate WSL2 + Docker
  - Create directories
  - Optionally start stack

### Prerequisite Checks
- WSL2 enabled  
- Docker Desktop installed and running  
- (Optional) NVIDIA GPU driver and WSL integration  

### Runtime Defaults
- Bind WebODM to `127.0.0.1:8000`
- Data and logs stored in `C:\ProgramData\SkyForge`
- Autostart disabled unless chosen
- No telemetry  
- Updates: opt-in only

---

## 4. Control App Functional Specification

### Core Responsibilities
1. **System Detection:** Verify WSL2, Docker, GPU status.  
2. **Stack Control:** Start/stop WebODM containers via PowerShell scripts.  
3. **Dashboard:** Display health indicators and logs.  
4. **Access Shortcuts:**  
   - Open WebODM (`http://localhost:8000`)  
   - Open Data Folder  
5. **Backup / Restore:** Manual and optional scheduled backups.  
6. **Settings Panel:**  
   - Port configuration  
   - Autostart toggles  
   - Lockdown plugin control  
   - GPU enablement  
   - Update check  
7. **Notifications:** Toasts for job completion/failure.  
8. **Security:** Store credentials via Windows Credential Manager (keytar).  

### Integration with PowerShell
- `start-webodm.ps1` → start containers and return JSON status  
- `stop-webodm.ps1` → stop containers gracefully  
- Health polling → REST + `docker ps`  

### UI Rules
- No CLI exposure  
- Explicit Start/Stop actions only  
- Clear visual feedback (status: Running / Stopped / Error)  

---

## 5. Compliance and Licensing Requirements

SkyForge redistributes WebODM/NodeODM/ODM (AGPL-3.0).  
Mandatory obligations:

1. Include full AGPL-3.0 license text in installer and About dialog.  
2. Provide access to corresponding source (URL or bundled archive).  
3. Maintain attribution and copyright notices.  
4. Keep all redistributed code unmodified or clearly documented.  
5. Ensure all Control App logic communicates with WebODM via REST API only.

License boundary diagram must be documented in `/docs/compliance.md`.

---

## 6. Operational Policy

| Category | Default | Notes |
|-----------|----------|------|
| **Network** | Localhost only | LAN exposure requires opt-in and warning |
| **Data Persistence** | Always preserved | Never auto-deleted on uninstall |
| **Backups** | Manual + optional scheduled | Stored in `backups/` |
| **GPU Mode** | Disabled by default | User enables via Settings |
| **Security** | Least privilege | No exposed Docker TCP socket |
| **Updates** | Manual check | Auto-update optional for Control App only |

---

## 7. Build Automation and Deliverables

Claude-Code must produce:

1. **Installer Build**
   - WiX MSI signed with code-signing certificate  
   - Embedded Control App payload  
   - Post-install validation script  

2. **Control App Build**
   - Built with `electron-builder`  
   - Signed executable  
   - MSI or EXE payload included in installer  

3. **Runtime Assets**
   - Docker Compose manifest (pinned image tags)  
   - Default config templates  
   - PowerShell scripts for stack control  

4. **Validation Pipeline**
   - Smoke test VM: verify installation, startup, persistence, GPU check (`nvidia-smi` visible).  
   - Output:  
     - `SkyForge-Setup-<version>.msi`  
     - `SkyForge-Source-<version>.zip` (AGPL compliance archive)  

---

## 8. Roadmap and Milestones

| Stage | Goal | Deliverable |
|--------|------|-------------|
| **Phase 1** | Functional MSI installer + working Control App | SkyForge v0.9 Beta |
| **Phase 2** | GPU runtime verification + Lockdown plugin integration | v1.0 Stable |
| **Phase 3** | Backup scheduler + auto-update | v1.1 |
| **Phase 4** | Multi-node and plugin extensibility | v1.2 |
| **Phase 5** | Cloud sync / enterprise deployment | v2.0 |

---

## 9. Definition of Done

A SkyForge release is *complete* when:
- Installer builds successfully and installs cleanly on a fresh Windows 11 system  
- Control App starts, controls, and monitors WebODM stack  
- Data persists across uninstall/reinstall  
- GPU processing passes validation  
- Compliance artifacts included and verified  

---

**End of Specification**
