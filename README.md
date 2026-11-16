# 🚜 GACS-Farm — GenieACS Multi-Instance Orchestrator

CLI manager untuk deploy, monitor, dan manage **multi-instance GenieACS (TR-069 ACS)** pada satu VPS, dengan integrasi L2TP VPN untuk konektivitas ONU lokal.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-Ubuntu%2022.04-orange.svg)
![GenieACS](https://img.shields.io/badge/GenieACS-v1.2%20%7C%20v1.3-green.svg)

---

## ✨ Fitur

| Fitur | Deskripsi |
|---|---|
| **Multi-Instance** | Deploy banyak GenieACS instance di satu server, masing-masing terisolasi |
| **Auto Port Allocation** | Port CWMP/NBI/FS/UI dialokasikan otomatis tanpa bentrok |
| **Nginx Reverse Proxy** | Subdomain otomatis per instance (`acs-<nama>.domain.id`) |
| **Wildcard SSL/HTTPS** | SSL via Let's Encrypt + Cloudflare DNS-01 challenge |
| **L2TP VPN Integration** | Otomatis buat L2TP user per instance untuk koneksi MikroTik |
| **ONU Route Management** | Auto routing subnet ONU agar ACS bisa summon/push perangkat |
| **Parameter Restore** | Restore provisions, virtual params, presets, UI config dari preset |
| **SkyGenPanel Integration** | Auto-deploy panel operator & portal pelanggan per instance (`panel-<nama>` & `portal-<nama>`) |
| **Version Support** | GenieACS Stable (v1.2) dan Latest (v1.3-dev) |
| **Pause/Unpause** | Freeze instance tanpa menghentikan container |
| **Activity Log** | Riwayat semua aksi dengan filter dan search |

---

## 🏗️ Arsitektur

```
┌─────────────────────────────────────────────────────────┐
│                        VPS (Cloud)                       │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │ Instance1 │  │ Instance2 │  │ Instance3 │  ...         │
│  │ GenieACS  │  │ GenieACS  │  │ GenieACS  │              │
│  │ +MongoDB  │  │ +MongoDB  │  │ +MongoDB  │              │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘               │
│       │              │              │                     │
│  ┌────┴──────────────┴──────────────┴────┐               │
│  │          Nginx Reverse Proxy          │               │
│  │       (SSL/HTTPS + Subdomains)        │               │
│  └───────────────────────────────────────┘               │
│                       │                                   │
│  ┌────────────────────┴──────────────────┐               │
│  │           L2TP VPN Server             │               │
│  │    172.16.101.1 (Server Gateway)      │               │
│  └────────┬───────────┬─────────────────┘               │
│           │           │                                   │
└───────────┼───────────┼───────────────────────────────────┘
            │           │
     L2TP Tunnel   L2TP Tunnel
            │           │
   ┌────────┴──┐  ┌─────┴──────┐
   │ MikroTik1 │  │ MikroTik2  │
   │172.16.101.│  │172.16.101. │
   │   10      │  │   11       │
   └─────┬─────┘  └─────┬─────┘
         │              │
    ┌────┴────┐    ┌────┴────┐
    │ONU/ONT  │    │ONU/ONT  │
    │10.50.x.x│    │192.168.x│
    └─────────┘    └─────────┘
```

---

## 📦 Instalasi

### Prasyarat
- **OS**: Ubuntu 22.04+ (atau Debian-based)
- **Docker & Docker Compose**: [Install Docker](https://docs.docker.com/engine/install/ubuntu/)
- **Git & Curl**: `apt install git curl`
- **Domain** dengan Wildcard DNS Record (`*.domain.id → IP VPS`)
- **Cloudflare API Token** (untuk SSL)

> Script otomatis mengecek root permission dan semua dependency saat dijalankan.

### Quick Start

```bash
# 1. Clone repository (path bebas)
git clone https://github.com/nasrulmufid/gacs-farm-plus-panel.git /home/docker/genieacs
cd /home/docker/genieacs/manager

# 2. Jalankan manager (harus root)
chmod +x mostech-gacs.sh
sudo ./mostech-gacs.sh
```

> Path clone bebas, script auto-detect lokasi. Contoh: `/opt/genieacs`, `/root/gacs`, dll.

### 🚀 Setup Pertama Kali (Fresh VPS)

Ikuti urutan ini di dalam manager:

```
┌─ 3. Services & Settings
│   ├─ 4. Setup Source           ← Clone GenieACS (stable/latest) & SkyGenPanel (+ build image)
│   ├─ 2. Install Services       ← Install L2TP, Nginx, Certbot
│   └─ 1. Setup Domain & SSL     ← Konfigurasi domain + SSL
│
└─ 1. Manage Instance
    ├─ 1. Install New Instance   ← Deploy GenieACS + SkyGenPanel pertama
    └─ 5. SkyGenPanel            ← Tambah/update panel ke instance yang sudah ada
```

**Langkah detail:**
1. Pilih `3` → Services & Settings → `4` Setup Source → Clone Stable/Latest & SkyGenPanel
2. Pilih `3` → Services & Settings → `2` Install Services → Install All
3. Pilih `3` → Services & Settings → `1` Setup Domain & SSL
4. Pilih `1` → Manage Instance → `1` Install New Instance (atau `5` SkyGenPanel untuk instance yang sudah ada)

---

## 🎮 Penggunaan

```bash
cd /home/docker/genieacs/manager
sudo ./mostech-gacs.sh
```

### Main Menu
```
╔══════════════════════════════════════════╗
║       MOSTECH GACS MANAGER v1.2          ║
║    GenieACS Multi-Instance Orchestrator  ║
╚══════════════════════════════════════════╝

  Instances: 1  │  Domain: domain.id  │  SSL: Active  │  L2TP: Active  │  Docker: Active
──────────────────────────────────────────
  1. Manage Instance
  2. View Activity Log
  3. Services & Settings
  0. Exit
```

### [1] Manage Instance
```
  1. Install New Instance
  2. Monitor Resources
  3. Pause / Unpause
  4. Uninstall Instance
  5. SkyGenPanel (Add / Update / Remove)
  0. Back
```

### [3] Services & Settings
```
  L2TP: Active  │  Nginx: Active  │  Certbot: Ready
  Domain: domain.id  │  SSL: Active
  Source Stable: Ready  │  Source Latest: Ready
  Panel Source: Ready  │  Panel Image: Built
──────────────────────────────────────────
  1. Setup Domain & SSL
  2. Install Services
  3. Uninstall Services
  4. Setup Source (GenieACS & SkyGenPanel)
  0. Back
```

### Install Instance — Auto Flow
Script akan otomatis:
- Alokasi port unik (CWMP/NBI/FS/UI + Panel/Portal)
- Build & start Docker containers
- Generate Nginx proxy config
- Buat L2TP VPN user + password
- Prompt subnet ONU → auto route di VPS
- Prompt restore parameter preset
- Prompt deploy SkyGenPanel (otomatis terhubung ke `http://genieacs-nbi:7557`)
- Tampilkan info koneksi lengkap + panduan MikroTik

---

## 🔌 Konektivitas ONU via L2TP

### Konfigurasi MikroTik

```routeros
# 1. Buat L2TP Client
/interface l2tp-client add name=l2tp-out1 connect-to=<IP_VPS> \
  user=<username> password=<password> disabled=no

# 2. Firewall: Allow L2TP forward (POSISI PALING ATAS!)
/ip firewall filter add chain=forward in-interface=l2tp-out1 \
  action=accept comment="Allow L2TP to LAN" place-before=0
/ip firewall filter add chain=forward out-interface=l2tp-out1 \
  action=accept comment="Allow LAN to L2TP" place-before=1

# 3. JANGAN pakai masquerade di L2TP interface!
```

> **⚠️ Penting:** Rule L2TP harus di posisi **paling atas** di firewall forward chain, sebelum hotspot atau drop rules.

### ACS URL di OLT
```
http://172.16.101.1:<PORT_CWMP>
```

---

## 📁 Struktur Direktori

```
/home/docker/genieacs/
├── README.md
├── .gitignore
├── manager/
│   ├── mostech-gacs.sh          # Script CLI utama
│   ├── config.conf              # Config per-VPS (runtime)
│   ├── log.txt                  # Activity log (runtime)
│   └── nginx/                   # Nginx configs (runtime)
├── instances/                   # Instance data (runtime)
│   └── <instance>/
│       ├── docker-compose.yml
│       ├── docker-compose.override.yml  # SkyGenPanel service (auto-merged)
│       ├── .panel.env                   # SkyGenPanel JWT secrets
│       └── .onu_subnet                  # ONU subnet info
└── source/
    ├── deploy/
    │   ├── stable/Dockerfile
    │   └── latest/Dockerfile
    ├── GACS-Ubuntu-22.04/
    │   └── parameter/           # Preset BSON files
    ├── stable/                  # GenieACS v1.2 (clone via menu)
    ├── latest/                  # GenieACS v1.3 (clone via menu)
    └── panel/                   # SkyGenPanel source (clone via menu)
```

---

## 🔐 Subdomain Pattern

| Service | Subdomain | Protocol |
|---|---|---|
| SkyGenPanel (Operator) | `panel-<nama>.domain.id` | HTTPS |
| SkyGenPanel (Customer Portal) | `portal-<nama>.domain.id` | HTTPS |
| GenieACS Web UI | `acs-<nama>.domain.id` | HTTPS |
| CWMP | `cwmp-<nama>.domain.id` | HTTP |
| NBI | `nbi-<nama>.domain.id` | HTTP |
| FS | `fs-<nama>.domain.id` | HTTP |

---

## 📝 Catatan

- **Root Required**: Script harus dijalankan sebagai root (`sudo`).
- **Dependency Auto-Check**: Script otomatis cek Docker, Git, Curl saat startup.
- **Parameter Restore**: Otomatis mendeteksi versi. Stable restore 4 collection (termasuk UI config), Latest hanya 3 (skip config karena UI v1.3 berbeda).
- **Route Persistence**: ONU routes disimpan di `/etc/l2tp-onu-routes.conf` dan otomatis di-restore saat VPS reboot via cron.
- **Periodic Inform**: Set interval 60 detik di OLT profile untuk near-realtime management.
- **MikroTik Firewall**: Rule L2TP harus di posisi 0-1 (paling atas) di forward chain.

---

## 📜 License

MIT License — by [Mostech/Safrin Network](https://github.com/safrinnetwork)
