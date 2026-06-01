# 📡 NetScan Pro

**Cybersecurity Assessment Tool per reti locali.**

NetScan Pro scansiona la rete, identifica ogni dispositivo, rileva vulnerabilità CVE note, calcola un Security Score e genera report PDF professionali pronti per il cliente. Pensato per consulenti IT e PMI che vogliono una panoramica immediata della propria esposizione al rischio.

---

## ✨ Funzionalità

### 🔍 Scansione e Discovery
- Scansione subnet IPv4 in notazione CIDR
- Supporto **multi-subnet** (es. `192.168.1.0/24, 10.0.0.0/24`)
- Discovery dispositivi via **ARP** — Scapy se disponibile, fallback `ping + arp -a` nativo
- Rilevamento MAC Address e Vendor (lookup API)
- Scansione porte TCP configurabile (1–65535)
- **Banner grabbing** per identificazione servizi e versioni
- Risultati in **tempo reale** via Server-Sent Events
- Compatibile **Windows e Linux** senza modifiche

### 🧠 Fingerprinting e Risk Engine
- Identificazione ruolo dispositivo con **score di confidenza**:
  `router · windows · linux · iot · phone · printer · unknown`
- Segnali combinati: OUI/MAC, vendor, porte aperte, TTL ping, banner HTTP/SSH
- **Risk Engine multifattoriale**: base risk × moltiplicatore ruolo × contesto di rete
- Contesti: `Home LAN · Corporate · Unknown`
- **CVE Database integrato**: 20+ vulnerabilità note (EternalBlue, BlueKeep, vsftpd backdoor, OpenSSH RCE, Apache path traversal…)
- Matching automatico CVE per versione/banner
- **Remediation steps** per ogni servizio a rischio (IT/EN)
- **Fix priority** 1–4 per ogni vulnerabilità

### 📊 Interfaccia Web — 6 Tab
| Tab | Contenuto |
|-----|-----------|
| 🔍 Scanner | Scansione live, log, device cards con CVE e remediation |
| 📋 Inventario | Tabella ordinabile con alias inline editabili, export CSV |
| 🗺️ Mappa | Topologia SVG interattiva: router → dispositivi, dot rosso per CVE |
| ⚠️ Priorità | Piano di remediation ordinato P1→P4 con passi concreti |
| 📜 Storico | Diff automatico tra scansioni: nuovo/sparito/porte cambiate |
| ⚙️ Impostazioni | Scheduler, email, password, nome azienda |

### 🔐 Autenticazione
- **Token di setup monouso** generato in console al primo avvio
- Nessun utente esterno può creare account senza accesso fisico al server
- Hashing password con **PBKDF2-SHA256** (300.000 iterazioni)
- Token di sessione a 7 giorni con revoca su logout
- Cambio password dall'interfaccia

### 📅 Scheduling e Alert Email
- Scansioni automatiche con **cron expression** configurabile
- Email HTML con score, rischi, diff dalla scansione precedente
- **PDF allegato** automaticamente
- Alert configurabili: nuovo dispositivo, rischio HIGH, score calato
- Test email dall'interfaccia
- SMTP configurabile (Gmail, Outlook, server aziendale)

### 📌 Alias e Baseline
- **Alias inline** per ogni dispositivo (click → digita → Enter)
- Pulsante **📌 Baseline**: salva lo stato attuale come riferimento
- Confronto automatico ad ogni scansione: `🆕 Nuovo · 🟡 Cambiato · ✓ OK`
- Badge visibili in device list e inventario

### 📄 Report e Export
- **PDF professionale** generato lato backend (ReportLab)
- **Report HTML** apribile nel browser, pronto per la stampa
- **Export CSV** inventario completo
- **Salvataggio log** in `.txt`
- Report bilingue 🇮🇹 / 🇬🇧

---

## 📦 Licenze delle dipendenze

Tutte le dipendenze sono **libere per uso commerciale** senza costi aggiuntivi.

| Libreria | Licenza | Note |
|----------|---------|------|
| FastAPI | MIT | ✅ Libero |
| Uvicorn | BSD | ✅ Libero |
| Pydantic | MIT | ✅ Libero |
| Requests | Apache 2.0 | ✅ Libero |
| ReportLab | BSD | ✅ Libero |
| APScheduler | MIT | ✅ Libero |
| PyInstaller | GPL v2 + eccezione bootloader | ✅ Libero per app commerciali |
| Inno Setup | Freeware | ✅ Libero anche commercialmente |
| Scapy | GPL v2 | ✅ **Opzionale** — se non installato usa fallback nativo |
| Npcap | Custom | ✅ **Non bundlato** — utente installa gratis da npcap.com |

### Scapy — perché è opzionale
Scapy ha licenza GPL v2. Per evitare vincoli di redistribuzione, è stato reso **opzionale**:
- Se Scapy è installato → usa ARP scan via Scapy (più accurato)
- Se Scapy non è installato → usa fallback `ping + arp -a` nativo (zero dipendenze)

Il fallback funziona su Windows e Linux senza richiedere Npcap.

### Npcap — perché non è bundlato
Bundlare Npcap nel proprio installer richiede una licenza OEM (~3.500€/anno).
NetScan Pro **non bundla Npcap**: se l'utente non lo ha, l'installer apre automaticamente
la pagina di download gratuito su npcap.com.

---

## 🏗️ Architettura

```
NetScan Pro
│
├── Main.py                  ← FastAPI backend, tutti gli endpoint
├── scan/
│   └── scan.py              ← ARP, port scan, banner, CVE, risk engine
├── auth.py                  ← Autenticazione PBKDF2 + token sessione
├── history.py               ← Storico scansioni (scan_history.json)
├── assets.py                ← Alias dispositivi + baseline
├── pdf_report.py            ← Generatore PDF con ReportLab
├── scheduler.py             ← APScheduler + alert email SMTP
├── PRESENTATION/
│   └── scanner.html         ← Frontend completo (single file, no dipendenze JS)
├── build.py                 ← Script PyInstaller
├── installer.iss            ← Script Inno Setup
└── requirements.txt
```

### Endpoint API

| Metodo | Endpoint | Auth | Descrizione |
|--------|----------|------|-------------|
| GET | `/` | No | Serve il frontend HTML |
| GET | `/api/setup/status` | No | Setup completato? |
| POST | `/api/setup` | Token console | Prima configurazione |
| POST | `/api/login` | No | Login → token sessione |
| POST | `/api/logout` | Sì | Revoca token |
| GET | `/api/me` | Sì | Info utente corrente |
| POST | `/api/change-password` | Sì | Cambio password |
| POST | `/scan` | Sì | Avvia scansione (SSE stream) |
| GET | `/history` | Sì | Storico scansioni |
| GET/POST | `/assets/{ip}` | Sì | Alias e note dispositivo |
| GET/POST | `/baseline` | Sì | Salva/leggi baseline |
| POST | `/report/pdf` | Sì | Genera PDF |
| GET/POST | `/scheduler/config` | Sì | Config scheduler |
| POST | `/scheduler/test-email` | Sì | Test email |

---

## 🚀 Installazione e Avvio

### Prerequisiti

- Python 3.11+
- Privilegi di **Amministratore** (Windows) o **root/sudo** (Linux)
- [Npcap](https://npcap.com) — solo se si vuole usare Scapy per ARP (opzionale)

### Installazione dipendenze

```bash
pip install -r requirements.txt
```

Per installare **senza Scapy** (nessun vincolo GPL, nessun Npcap):

```bash
pip install fastapi uvicorn pydantic requests reportlab apscheduler
```

### Avvio

```bash
# Windows — esegui come Amministratore
python Main.py

# Linux
sudo python3 Main.py
```

### Primo avvio — setup account

Al primo avvio il server stampa in console un **token monouso**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  NetScan Pro — PRIMO AVVIO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TOKEN DI SETUP (copia questo codice):

      a3f9c2e1d7b84f05...

  Inseriscilo nel browser per creare la password.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

1. Apri `http://localhost:8000`
2. Incolla il token nel form di setup
3. Scegli password e nome azienda
4. Accedi

### Reset account

```bash
# Windows
del auth_config.json

# Linux
rm auth_config.json
```

---

## 🖥️ Dove installarlo in azienda

### PC / Mini PC dedicato (PMI tipica)
```
Mini PC sempre acceso (es. Intel NUC, ~150€)
collegato alla rete via cavo
Windows 10/11 Pro + Npcap

Accesso da tutta la rete: http://192.168.1.X:8000
```

### Server Linux esistente (consigliato)
```
Installa come servizio systemd — parte automaticamente al riavvio
Nessun Npcap necessario
Più stabile di Windows per uso 24/7
```

Configurazione servizio systemd:

```ini
# /etc/systemd/system/netscan.service
[Unit]
Description=NetScan Pro
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/netscan
ExecStart=/usr/bin/python3 Main.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable netscan
sudo systemctl start netscan
```

---

## 📦 Build Installer Windows

### Step 1 — Eseguibile

```bash
pip install pyinstaller
python build.py
# Output: dist\NetScanPro\
```

### Step 2 — Installer .exe

1. Installa [Inno Setup 6](https://jrsoftware.org/isdl.php)
2. Apri `installer.iss` → premi **F9**
3. Output: `Output\NetScanPro_Setup.exe`

> **Npcap:** l'installer apre automaticamente npcap.com se non è già installato.
> Non viene bundlato per evitare la licenza OEM da ~3.500€/anno.

---

## ⚙️ Configurazione Scanner

| Parametro | Default | Descrizione |
|-----------|---------|-------------|
| Subnet | `192.168.1.0/24` | CIDR da scansionare (più subnet separate da virgola) |
| Timeout | `2s` | Timeout per ARP e banner grab |
| Porte max | `1024` | Range porte TCP (1–65535) |
| Network context | `unknown` | `home · corporate · unknown` |
| Backend URL | `http://localhost:8000` | Endpoint del backend |

---

## 📋 Dipendenze

```
fastapi          ← Web framework (MIT)
uvicorn          ← ASGI server (BSD)
pydantic         ← Validazione dati (MIT)
requests         ← Vendor lookup API (Apache 2.0)
reportlab        ← Generazione PDF (BSD)
apscheduler      ← Scansioni pianificate (MIT)
scapy            ← ARP scan avanzato — OPZIONALE (GPL v2)
```

---

## ⚠️ Disclaimer Legale

Questo software è progettato **esclusivamente** per l'analisi di reti di propria proprietà
o per cui si dispone di esplicita autorizzazione scritta.

L'utilizzo su reti o sistemi di terzi senza autorizzazione può costituire reato ai sensi
del D.Lgs. 231/2001 e dell'art. 615-ter del Codice Penale italiano, nonché violare
normative internazionali (CFAA negli USA, Computer Misuse Act nel Regno Unito).

**L'autore declina ogni responsabilità per usi non autorizzati.**
