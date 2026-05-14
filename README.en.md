# Otto — Personal AI Assistant

<!-- GIF: Demostración del bot respondiendo por Telegram (texto y voz) -->

<p align="center">
  <a href="README.md"><img src="https://flagcdn.com/w40/es.png" width="20"> Español</a> | 
  <img src="https://flagcdn.com/w40/gb.png" width="20"> <strong>English</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/n8n-workflow-orange" />
  <img src="https://img.shields.io/badge/Groq-LLaMA%203.3%2070B-blue" />
  <img src="https://img.shields.io/badge/Obsidian-vault-purple" />
  <img src="https://img.shields.io/badge/Telegram-bot-2CA5E0" />
  <img src="https://img.shields.io/badge/license-MIT-green" />
</p>

---

## <img src="https://flagcdn.com/w40/gb.png" width="20"> English <a name="-english"></a>

Personal AI assistant running 24/7 on a Raspberry Pi, accessible via Telegram. Capable of saving, editing and querying notes in Obsidian, remembering past conversations and responding in both text and voice.

### 🏗️ Infrastructure

<!-- IMAGE: Photo or diagram of the Raspberry Pi with SSD connected -->

| Component | Detail |
|---|---|
| Hardware | Raspberry Pi 4 (4GB RAM) |
| Operating system | DietPi |
| Storage | 256GB SSD (mounted at `/mnt/ssd`) |
| Containers | Docker + Portainer |
| Remote access | Tailscale (private VPN) |
| Sync | Syncthing (Windows PC + Android) |

### 🛠️ Technologies

| Technology | Purpose |
|---|---|
| [n8n](https://n8n.io/) | Workflow orchestration |
| [Groq](https://groq.com/) | LLaMA 3.3 70B (text) + Whisper (speech-to-text) |
| [Telegram Bot API](https://core.telegram.org/bots/api) | Messaging interface |
| [Obsidian](https://obsidian.md/) | Knowledge base (markdown vault) |
| [Syncthing](https://syncthing.net/) | Vault sync across devices |
| Google Translate TTS | Text-to-speech (unofficial endpoint, free) |

### 🗂️ Vault structure

<!-- IMAGE: Obsidian Graph View screenshot showing connected notes -->

```
Otto Vault/
├── Home.md                  ← Main dashboard
├── Inbox/                   ← Quick unclassified notes
├── Proyectos/               ← Projects with a start and end
├── Diario/                  ← Daily personal entries
├── Contactos/               ← People information
├── Finanzas/                ← Expenses, income, budgets
├── Aprendizaje/             ← Books, courses, resources
├── Sistema_Otto/            ← Otto config and memory
├── Historiales/             ← Archived weekly history
└── Papelera/                ← Deleted notes
```

Each folder has its own auto-updated `Index.md`. `Home.md` links all indexes, creating a navigable knowledge graph in Obsidian.

### 📋 Requirements

- Raspberry Pi 4 (4GB RAM recommended) or your homelab.
- Docker and Portainer installed
- [Groq](https://console.groq.com/) account (free)
- Telegram bot created with [@BotFather](https://t.me/BotFather)
- Obsidian installed on PC and/or Android
- Syncthing on all devices

### 🔧 Quick setup

1. Clone the repository
2. Import the workflow JSON into n8n
3. Set up credentials (Groq API key + Telegram Bot Token)
4. Adjust environment variables
5. Activate the workflow

## 🚨 NOTICE

It's very likely that you'll need to change the paths of the constants or variables in the "Code" and "Agent AI" nodes. Additionally, enabling certain features in the .yaml file may prevent some functionality from working.

---

## 📦 Versions

### V1.0 — Initial functional structure

**Main features:**
- LLM-based folder classification (AI Agent 1)
- Reading relevant notes from Obsidian vault
- Basic note creation and editing in markdown
- Text-only responses via Telegram

**Flow:**
```
Telegram Trigger
  → AI Agent 1 (choose folders)
  → Code (read notes)
  → AI Agent 2 (generate response)
  → Code parse JSON
  → If is_note
      ├── True  → Write note → Telegram
      └── False → Telegram
```

<!-- IMAGE: V1.0 workflow screenshot in n8n -->

---

### V1.1 — Context and note lifecycle

**New features:**
- Persistent conversation history in `.historial.json`
- Full support for editing and deleting notes
- Improved `is_note` branching logic
- Assistant remembers past interactions

**Added flow:**
```
→ Code Read History (before AI Agent 2)
→ Code Write History True/False (after responding)
```

**Known weakness:** deletion was permanent, no trash bin.

<!-- IMAGE: V1.1 workflow screenshot in n8n -->

---

### V1.2 — Trash bin and indexes

**New features:**
- **Trash bin** — deleted notes are moved to `Papelera/` instead of being permanently removed ⚠️
- **Auto indexes** — each folder has an `Index.md` that updates automatically on create/delete
- **Home.md** — central dashboard linking all indexes
- **Graph View** — notes appear visually connected in Obsidian

**⚠️ Known issue:** trash bin functionality is currently not working. We will attempt to fix this in future versions.

<!-- IMAGE: Obsidian Graph View with folder groups -->

---

### V1.3 — Cleanup and robustness

**New features:**
- **Broken link removal** — on note deletion, all `[[note]]` references are cleaned across the vault via `removeLinksFromVault()`
- **Improved JSON parsing** — robust extraction even when the LLM includes extra text around the JSON
- **Automatic history rotation** — every Sunday at 3am, `historial.json` is archived to `Historiales/YYYY-MM-DD.json` (Configurable to the day and time of your choice)
- **Token optimization** — AI Agent 1 now detects intent (`list`, `read`, `create`, `edit`, `delete`, `chat`) and Code only sends the necessary context.

**Technical details:**
- `Code Títulos` added before AI Agent 1 to provide full vault listing
- Structured Output Parser with flexible JSON Schema supporting multiple `note_title` values separated by comma
- Trash bin works correctly from this version onwards

<!-- IMAGE: Full V1.3 workflow screenshot in n8n -->

---

### V1.4 — Multimodal experience (voice)

**New features:**
- **Voice input** — Telegram voice notes transcribed with Groq Whisper
- **Voice output** — reply converted to audio via Google TTS and sent as voice note
- **Separate audio flow** — Switch node detects text vs audio messages
- **`isAudio` flag** — controls response format throughout the flow
- **Special character cleanup** — prevents Telegram Markdown parsing errors

**Audio subflow:**
```
Switch (text/audio)
  └── Audio → Get File → HTTP Download → HTTP Groq Whisper
            → Code Transcription (injects text into main flow)
            → [main flow]
            → If isAudio
                ├── True  → HTTP Google TTS → Send Audio
                └── False → Send Text
```

<!-- GIF: Demo sending a voice note and receiving a voice response -->

---

## 🚀 Planned improvements (V1.5 / V2.0)

| Improvement | Description |
|---|---|
| Modularization | Sub-workflows for History Manager, Vault Reader and TTS Service |
| Error handling | Error Trigger for failures in Groq, Telegram or file I/O |
| Environment variables | Externalize hardcoded paths like `OBSIDIAN_VAULT_PATH` |
| `confidence` field | AI Agent 1 asks for confirmation before destructive actions if confidence is low |
| Few-shot prompts | Examples in AI Agent 1 prompt to improve `note_title` accuracy |
| Semantic search | Embeddings with ChromaDB to find notes without exact title match |
| Daily summary | Schedule Trigger sending pending tasks every morning |
| Slash commands | `/stats`, `/search`, `/export` via `botCommand` in Telegram |
| PDF support | Upload PDFs for Otto to use as context when answering questions |
| Database | Migration to PostgreSQL for history and metadata when volume requires it |

---

## 📄 License

MIT — feel free to use, adapt and learn from this project.
