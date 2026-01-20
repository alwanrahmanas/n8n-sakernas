# 🤖 SAKERNAS RAG - WhatsApp AI Assistant

> **Intelligent RAG (Retrieval Augmented Generation) System for BPS SAKERNAS February 2026**

WhatsApp chatbot yang menjawab pertanyaan seputar Survei Angkatan Kerja Nasional (SAKERNAS) menggunakan teknologi AI dan vector search.

![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-green)
![Supabase](https://img.shields.io/badge/Supabase-Vector%20Store-blue)
![WAHA](https://img.shields.io/badge/WAHA-WhatsApp%20API-25D366)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Project Structure](#-project-structure)
- [Documentation](#-documentation)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

SAKERNAS RAG adalah sistem AI assistant yang:
- 📱 Terintegrasi dengan WhatsApp via WAHA
- 🔍 Menggunakan vector search untuk menemukan dokumen relevan
- 🤖 Menjawab pertanyaan dengan konteks dari knowledge base
- ⚡ Mendukung adaptive routing (single-path & multi-path retrieval)
- 💬 Filter greeting otomatis untuk respon instan

### Use Cases:
- ✅ Menjawab pertanyaan definisi ketenagakerjaan
- ✅ Lookup kode KBLI dan KBJI
- ✅ Menjelaskan prosedur dan pedoman SAKERNAS
- ✅ Klarifikasi klasifikasi status pekerjaan
- ✅ Query kompleks dengan multi-hop reasoning

---

## 🏗 Architecture

```
┌─────────────────┐
│   WhatsApp      │
│   (WAHA API)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Greeting       │────▶│  Send Greeting  │
│  Filter         │     │  Response       │
└────────┬────────┘     └─────────────────┘
         │ (Not Greeting)
         ▼
┌─────────────────┐
│  Intent         │
│  Classifier     │
│  (GPT-4o-mini)  │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│Single │ │Multi  │
│Path   │ │Path   │
│Retriev│ │Retriev│
└───┬───┘ └───┬───┘
    │         │
    ▼         ▼
┌─────────────────┐
│  Supabase       │
│  Vector Store   │
│  (pgvector)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Hybrid         │
│  Reranker       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Synthesizer    │
│  (GPT-4.1-nano) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  WhatsApp       │
│  Response       │
└─────────────────┘
```

---

## ✨ Features

### 🎯 Adaptive Query Routing
- **Simple queries** → Single-path retrieval (faster, cheaper)
- **Complex queries** → Multi-path with 3 parallel searches

### 🔍 Hybrid Reranking
- Semantic similarity (70%)
- Keyword matching (30%)
- Metadata boost for document type

### 💬 Greeting Detection
- Instant response for greetings ("halo", "test", "cek")
- Skip RAG pipeline = save cost & time

### 🧠 Memory Support
- Conversation context per user
- Configurable session management

### 📊 Comprehensive Logging
- Debug logs at each node
- Error tracking and handling

---

## 🚀 Installation

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- [Supabase](https://supabase.com/) account
- [OpenAI](https://openai.com/) API key
- [WAHA](https://github.com/devlikeapro/waha) WhatsApp API

### Step 1: Setup Supabase

```sql
-- Run the SQL setup script
-- Located at: setup_supabase_simple.sql
```

### Step 2: Import n8n Workflow

1. Open n8n
2. Import `SAKERNAS RAG - Hybrid Adaptive FIXED V5.json`
3. Configure credentials:
   - OpenAI API
   - Supabase API
   - WAHA API

### Step 3: Push Documents

```bash
# Install dependencies
pip install -r requirements.txt

# Configure .env
cp .env.example .env
# Edit .env with your credentials

# Push documents to Supabase
python push_rag_vectors.py
```

### Step 4: Activate Workflow

1. Test with simple query
2. Verify responses
3. Activate workflow

---

## ⚙️ Configuration

### Environment Variables

```env
# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-anon-key

# OpenAI
OPENAI_API_KEY=sk-your-key

# WAHA
WAHA_URL=http://localhost:3000
WAHA_API_KEY=your-waha-key
```

### n8n Credentials

| Credential | Node |
|------------|------|
| OpenAi account | LLM nodes |
| Supabase account | Vector Store nodes |
| WAHA account | WhatsApp nodes |

---

## 📖 Usage

### Send Query via WhatsApp

```
User: Apa itu SAKERNAS?
Bot:  SAKERNAS (Survei Angkatan Kerja Nasional) adalah survei rutin 
      BPS untuk mengumpulkan data ketenagakerjaan...
```

### Query Types Supported

| Type | Example | Routing |
|------|---------|---------|
| Definitional | "Apa itu pengangguran?" | Single-path |
| Code Lookup | "Kode KBLI untuk ojek?" | Single-path |
| Procedural | "Bagaimana cara mengisi formulir?" | Single-path |
| Classification | "Apakah mahasiswa termasuk angkatan kerja?" | Multi-path |
| Comparison | "Bedanya pekerja formal dan informal?" | Multi-path |

---

## 📁 Project Structure

```
sakernas-rag/
├── 📄 SAKERNAS RAG - Hybrid Adaptive FIXED V5.json  # Main workflow
├── 📁 data/                    # PDF documents
├── 📁 src/                     # Python source code
├── 📁 configs/                 # Configuration files
│
├── 🔧 JavaScript Nodes
│   ├── greeting_filter_conservative.js
│   ├── normalize_data_single_ultra_fixed.js
│   ├── hybrid_reranker_fixed.js
│   ├── format_context_fixed.js
│   └── parse_intent_output.js
│
├── 🔧 System Messages
│   ├── intent_classifier_system_message_v3.txt
│   ├── single_path_retriever_system_message.txt
│   ├── multi_path_retriever_system_message.txt
│   └── node4_synthesizer_system_message.txt
│
├── 🗄️ Database
│   ├── setup_supabase_simple.sql
│   └── push_rag_vectors.py
│
├── 📚 Documentation
│   ├── QUICK_START.md
│   ├── HYBRID_ADAPTIVE_WORKFLOW.md
│   ├── PERFORMANCE_OPTIMIZATION_GUIDE.md
│   ├── TROUBLESHOOTING_RETRIEVAL.md
│   └── ... (other guides)
│
└── 📋 Config
    ├── .env
    ├── .gitignore
    └── requirements.txt
```

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [QUICK_START.md](QUICK_START.md) | Get started in 10 minutes |
| [HYBRID_ADAPTIVE_WORKFLOW.md](HYBRID_ADAPTIVE_WORKFLOW.md) | Detailed workflow explanation |
| [PERFORMANCE_OPTIMIZATION_GUIDE.md](PERFORMANCE_OPTIMIZATION_GUIDE.md) | Speed & cost optimization |
| [TROUBLESHOOTING_RETRIEVAL.md](TROUBLESHOOTING_RETRIEVAL.md) | Debug common issues |
| [SUPABASE_DEBUG_GUIDE.md](SUPABASE_DEBUG_GUIDE.md) | Vector store debugging |
| [GREETING_FILTER_GUIDE.md](GREETING_FILTER_GUIDE.md) | Greeting detection setup |

---

## 📊 Performance

| Metric | Value |
|--------|-------|
| Avg Response Time | 15-25 seconds |
| Cost per Query | ~$0.003-0.005 |
| Greeting Response | <1 second |
| Accuracy | ~90% (based on test cases) |

---

## 🛠 Tech Stack

- **Workflow:** n8n
- **LLM:** OpenAI GPT-4o-mini, GPT-3.5-turbo
- **Vector Store:** Supabase (pgvector)
- **Embeddings:** text-embedding-3-small
- **WhatsApp:** WAHA (WhatsApp HTTP API)
- **Backend:** Python (for data processing)

---

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

---

## 📄 License

This project is for internal BPS use. Contact the maintainer for licensing information.

---

## 👤 Author

**BPS - Badan Pusat Statistik**

SAKERNAS February 2026 Team

---

## 🙏 Acknowledgments

- [n8n](https://n8n.io/) - Workflow automation
- [OpenAI](https://openai.com/) - Language models
- [Supabase](https://supabase.com/) - Vector database
- [WAHA](https://github.com/devlikeapro/waha) - WhatsApp API

---

<p align="center">
  Made with ❤️ for SAKERNAS 2026
</p>
