<p align="center">
  <img src="https://cloakclaw.com/cloakclaw-logo.jpg" alt="CloakClaw" width="120" />
</p>

<h1 align="center">🦀 CloakClaw</h1>

<p align="center">
  <strong>Local AI privacy proxy</strong> — redact sensitive data before sending to cloud LLMs, restore originals in the response.
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/cloakclaw"><img src="https://img.shields.io/npm/v/cloakclaw?color=00d4ff&label=npm" alt="npm version"></a>
  <a href="https://github.com/canonflip/cloakclaw/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-green" alt="MIT License"></a>
  <a href="https://www.npmjs.com/package/cloakclaw"><img src="https://img.shields.io/npm/dm/cloakclaw?color=orange" alt="npm downloads"></a>
  <a href="https://x.com/cloakclaw"><img src="https://img.shields.io/badge/follow-@cloakclaw-1DA1F2?logo=x&logoColor=white" alt="Follow @cloakclaw"></a>
</p>

<p align="center">
  <a href="https://cloakclaw.com">Website</a> · 
  <a href="#install">Install</a> · 
  <a href="#quick-start">Quick Start</a> · 
  <a href="#web-ui">Web UI</a> · 
  <a href="https://www.npmjs.com/package/cloakclaw">npm</a>
</p>

---

Your data never leaves your machine. Zero cloud dependency for cloaking. Zero telemetry.

## How It Works

```
📄 Your Document               🔒 Cloaked Version              ☁️ Cloud LLM
─────────────────────────────────────────────────────────────────────────────
"Sarah Chen, CEO of            "James Park, CEO of             Sees only
 Acme Corp, owes               Northstar Solutions, owes       fake data —
 $4.2M on invoice               $3.8M on invoice               analyzes it
 #INV-2024-0847"                #INV-2024-0293"                normally

☁️ LLM Response                🔓 Restored Response
─────────────────────────────────────────────────────────────────────────────
"James Park should             "Sarah Chen should
 restructure the $3.8M          restructure the $4.2M
 obligation..."                  obligation..."
```

**Two-pass detection** catches what regex alone misses:
1. ⚡ **Regex pass** — fast pattern matching (SSNs, emails, IPs, dollar amounts, etc.)
2. 🧠 **LLM pass** — local Ollama model finds names, companies, and ambiguous entities in context

Works without Ollama in regex-only mode. LLM pass catches significantly more.

## Install

```bash
npm install -g cloakclaw
```

**Optional but recommended:**
```bash
brew install poppler          # Better PDF text extraction (macOS)
apt install poppler-utils     # Linux equivalent
ollama pull qwen2.5:7b        # AI-powered name/company detection
```

**Verify:** `cloakclaw --version`

## Quick Start

```bash
# Cloak a document
cloakclaw cloak contract.pdf --profile legal

# See the entity mapping
cloakclaw diff -s <session-id>

# After getting LLM response, restore originals
cloakclaw decloak -s <session-id> -f response.txt
```

## Web UI

```bash
cloakclaw serve
# → http://localhost:3900
```

Drag-and-drop files, paste text, toggle entity types, real-time streaming progress, session history, and a built-in decloaker.

<!-- TODO: Add screenshot/demo GIF here -->
<!-- ![CloakClaw Web UI](https://cloakclaw.com/demo.gif) -->

## 24 Entity Types

| Category | Types |
|----------|-------|
| 👤 **Identity** | People, Companies, Passports, Drivers License |
| 📞 **Contact** | Emails, Phones, Addresses |
| 💰 **Financial** | Dollars, Percentages, Accounts, Banks, SSNs |
| ⚖️ **Legal** | Case Numbers, Jurisdictions |
| 💻 **Tech** | IP Addresses, MAC Addresses, Passwords, API Keys, URLs |
| 🔧 **Other** | Crypto Wallets, GPS Coordinates, VINs, Medical IDs, Dates |

## 6 Document Profiles

| Profile | Use Case | Types |
|---------|----------|-------|
| 🛡️ **General** | Catch-all | All 24 |
| 📜 **Legal** | Contracts, NDAs, filings | 10 |
| 💰 **Financial** | Bank statements, P&L, investor docs | 11 |
| ✉️ **Email** | Business correspondence | 10 |
| 💻 **Code** | .env files, configs, infra docs | 9 |
| 🏥 **Medical** | HIPAA-adjacent records | 11 |

## Security

- 🔐 **AES-256-GCM** encrypted mapping database
- 🔑 **Optional password protection** (scrypt-derived key)
- ♻️ **Auto-expiry** — sessions purged after 7 days
- 📏 **50MB upload limit**
- 🚫 **Zero telemetry** — nothing phones home, ever
- 🏠 **100% local** — cloaking never touches the internet

## CLI Reference

```
cloakclaw cloak [file]              Cloak a file or stdin
  -p, --profile <name>              general|legal|financial|email|code|medical
  -i, --interactive                  Approve each entity
  --no-llm                           Skip Ollama LLM pass
  -o, --output <file>                Write to file
  -c, --copy                         Copy to clipboard

cloakclaw decloak                    Restore originals in LLM response
  -s, --session <id>                 Session ID from cloak
  -f, --file <file>                  Read from file (or stdin)

cloakclaw diff -s <id>               Show entity mapping table
cloakclaw sessions                   List recent sessions
cloakclaw session <id>               Session details + mappings

cloakclaw password set               Set database password
cloakclaw password remove            Remove password

cloakclaw config show                Show config
cloakclaw config set <key> <val>     Set config value
cloakclaw serve                      Start web UI (:3900)
cloakclaw doctor                     Check dependencies
```

## Configuration

Config: `~/.cloakclaw/config.yaml`

```yaml
ollama:
  url: http://localhost:11434
  model: qwen2.5:7b
```

### Recommended Models by RAM

| RAM | Model | Detection Quality |
|-----|-------|-------------------|
| 8GB | `qwen2.5:3b` | Basic (regex carries most weight) |
| 16GB | `qwen2.5:7b` | Good |
| 32GB+ | `qwen2.5:32b` | Very good |
| 64GB+ | `qwen2.5:72b` | Excellent |

## OpenClaw Integration

CloakClaw works as an [OpenClaw](https://github.com/openclaw/openclaw) skill — documents sent to your agent are automatically cloaked before reaching cloud LLMs:

```bash
clawhub install cloakclaw
```

## How It Works Under the Hood

1. **Extract** — PDF via poppler (`pdftotext`), with pdfjs-dist fallback. Supports PDF, TXT, MD, CSV, JSON, DOCX, and more.
2. **Detect** — Two-pass NER: regex patterns → Ollama LLM contextual analysis
3. **Replace** — Generate realistic fake data (consistent within session). Dollars scale proportionally, dates shift consistently, names get plausible alternatives.
4. **Store** — Encrypted SQLite mapping (original ↔ replacement)
5. **Decloak** — Reverse substitution using session mappings

## Built With

- [Ollama](https://ollama.com/) — local LLM inference
- [pdfjs-dist](https://github.com/nicolo-ribaudo/pdfjs-dist) — PDF extraction fallback
- [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) — encrypted mapping store
- Node.js 22+ — native Web API multipart parsing

## Contributing

Issues and PRs welcome. See [SPEC.md](SPEC.md) for architecture details.

## ⚠️ Disclaimer

**CloakClaw is NOT HIPAA, GDPR, SOC 2, PCI-DSS, or CCPA compliant.** It is a best-effort privacy tool that may miss entities or produce false positives. You are solely responsible for reviewing cloaked output before sharing with any third party. See [cloakclaw.com](https://cloakclaw.com) for full terms.

## License

[MIT](LICENSE) — © 2026 [Canonflip](https://canonflip.com)
