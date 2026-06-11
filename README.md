# ⚡ VectorDB — Vector Database Engine in C++

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue?logo=cplusplus&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Ollama](https://img.shields.io/badge/Ollama-Powered-orange?logo=ollama)
![Platform](https://img.shields.io/badge/Platform-Windows-blue?logo=windows)

A **Vector Database** built from scratch in C++ with a modern web-based dashboard.  
Features **HNSW**, **KD-Tree**, and **Brute Force** nearest-neighbor search, plus a **RAG (Retrieval-Augmented Generation) pipeline** using a local LLM via Ollama.

> 💡 I built this project to deeply understand how production vector databases like Pinecone, Weaviate, and Chroma work under the hood — by implementing the core algorithms myself in C++.

---

## Features

| Feature | Description |
|---|---|
| 🔍 **3 Search Algorithms** | HNSW (production-grade), KD-Tree, Brute Force — run all three and compare speed |
| 📏 **3 Distance Metrics** | Cosine similarity, Euclidean distance, Manhattan distance |
| 📊 **16D Demo Vectors** | 20 pre-loaded semantic vectors across 4 categories (CS, Math, Food, Sports) |
| 🗺️ **2D PCA Scatter Plot** | Live visualization of semantic space — watch clusters form in real time |
| 🧠 **Real Document Embedding** | Paste any text → Ollama embeds it with `nomic-embed-text` (768D) |
| 🤖 **RAG Pipeline** | Ask questions about your documents → HNSW retrieves context → local LLM generates answers |
| 🌐 **Full REST API** | CRUD endpoints: insert, delete, search, benchmark, hnsw-info |

---

## How It Works

```
Your Text
    │
    ▼
Ollama (nomic-embed-text)          ← converts text to a 768-dimensional vector
    │
    ▼
HNSW Index (C++)                   ← indexes the vector in a multilayer graph
    │
    ▼
Semantic Search                    ← finds nearest neighbors in vector space
    │
    ▼
Ollama (llama3.2)                  ← reads retrieved chunks, generates an answer
    │
    ▼
Answer
```

**HNSW (Hierarchical Navigable Small World)** is the same algorithm used by Pinecone, Weaviate, Chroma, and Milvus. It builds a multilayer graph where each layer is progressively sparser — searches start at the top layer and zoom in, achieving O(log N) complexity instead of O(N) for brute force.

---

## What I Learned

- Implementing **HNSW from scratch** — multilayer graph construction, greedy search, beam search with ef parameter tuning
- How **KD-Trees degrade** in high dimensions (curse of dimensionality) vs HNSW's graph-based approach
- Building a **REST API in C++** using a single-header HTTP library (cpp-httplib)
- **PCA projection** — reducing 16D vectors to 2D for visualization using power iteration
- **Text chunking with overlap** for document ingestion in RAG systems
- Integrating with **Ollama's embedding and generation APIs** for real 768D embeddings
- Thread-safe data structures with `std::mutex` for concurrent HTTP requests

---

## Prerequisites

You need **3 things** installed:

1. **MSYS2** (provides the g++ compiler)
2. **Git**
3. **Ollama** (runs the local AI models)

---

## Setup (Windows)

### 1. Install MSYS2 (C++ Compiler)

1. Download from **https://www.msys2.org** and install (keep default path `C:\msys64`)
2. Open **MSYS2 UCRT64** from Start Menu
3. Run:

```bash
pacman -Syu
# Close and reopen if prompted
pacman -S mingw-w64-ucrt-x86_64-gcc
```

4. Add to Windows PATH:
   - `Win + R` → `sysdm.cpl` → **Advanced** → **Environment Variables**
   - Edit **Path** → Add `C:\msys64\ucrt64\bin`
   - Verify in a new PowerShell: `g++ --version`

### 2. Install Git

Download from **https://git-scm.com/download/win**, install with defaults.

### 3. Install Ollama

1. Download from **https://ollama.com** → install
2. Pull the required models:

```powershell
ollama pull nomic-embed-text    # ~274 MB — embedding model
ollama pull llama3.2            # ~2 GB — language model
```

3. Verify: `ollama list` should show both models.

> **Minimum specs:** 8GB RAM recommended. Models use ~3GB total.

---

## Build & Run

### Clone and compile

```powershell
git clone https://github.com/YOUR_USERNAME/VectorDB_RAG_Engine.git
cd VectorDB_RAG_Engine
g++ -std=c++17 -O2 main.cpp -o db -lws2_32
```

> Compilation takes ~10–20 seconds. If `g++` isn't found, check that MSYS2 is in your PATH.

### Start the server

```powershell
# Terminal 1 — Start Ollama (skip if already in system tray)
ollama serve

# Terminal 2 — Start VectorDB
./db
```

You should see:
```
=== VectorDB Engine ===
http://localhost:8080
20 demo vectors | 16 dims | HNSW+KD-Tree+BruteForce
Ollama: ONLINE
  embed model: nomic-embed-text  gen model: llama3.2
```

Open **http://localhost:8080** in your browser.

---

## Usage

### Search (Demo Vectors)
- Type a concept: `binary tree`, `sushi`, `basketball`, `calculus`
- Pick an algorithm (HNSW / KD-Tree / Brute Force) and distance metric
- Hit **⚡ SEARCH** — results appear with distances and the scatter plot highlights matches
- Run **▶ COMPARE ALL ALGOS** to benchmark all three side-by-side

### Documents (Real Embeddings)
- Add a title and paste any text (lecture notes, articles, etc.)
- Click **⚡ EMBED & INSERT** — long texts are auto-chunked into overlapping 250-word segments
- Each chunk gets a real 768D embedding via Ollama

### Ask AI (RAG)
1. Insert some documents first
2. Ask a question → the system embeds your question, runs HNSW search to find the most relevant chunks, then sends them as context to the local LLM
3. The AI answers based only on your documents

---

## REST API

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/search?v=f1,f2,...&k=5&metric=cosine&algo=hnsw` | K-NN search |
| `POST` | `/insert` | Insert a demo vector |
| `DELETE` | `/delete/:id` | Delete by ID |
| `GET` | `/items` | List all demo vectors |
| `GET` | `/benchmark?v=...&k=5&metric=cosine` | Compare all 3 algorithms |
| `GET` | `/hnsw-info` | HNSW graph layer stats |
| `POST` | `/doc/insert` | Embed and store document (`{title, text}`) |
| `GET` | `/doc/list` | List stored documents |
| `DELETE` | `/doc/delete/:id` | Delete document chunk |
| `POST` | `/doc/ask` | RAG query (`{question, k}`) |
| `GET` | `/status` | Ollama status and model info |

---

## Project Structure

```
VectorDB_RAG_Engine/
├── main.cpp        ← C++ backend (HNSW, KD-Tree, BruteForce, REST API, RAG)
├── httplib.h       ← Single-header HTTP server library (cpp-httplib)
├── index.html      ← Frontend dashboard (PCA visualization, chat UI, benchmark)
└── README.md       ← This file
```

### Architecture

| Component | Complexity | Description |
|---|---|---|
| `BruteForce` | O(N·d) | Exact search, baseline |
| `KDTree` | O(log N) | Exact, axis-aligned partitioning |
| `HNSW` | O(log N) | Approximate, multilayer small-world graph |
| `VectorDB` | — | Unified interface over all 3 (16D demo vectors) |
| `DocumentDB` | — | HNSW-only index for Ollama embeddings (768D) |
| `OllamaClient` | — | HTTP client for `/api/embeddings` + `/api/generate` |

---

## Algorithm Notes

### HNSW
- Nodes inserted into a multilayer graph with random level assignment
- Insert: greedy descent from top layer → beam search at each layer → bidirectional connections
- Search: greedy descent → expand at layer 0 with priority queue (ef parameter)
- Upper layers act as "highways" for fast neighborhood localization

### KD-Tree
- Binary space partitioning along cycling dimensions
- Prunes subtrees when closest possible point can't beat current best
- **Degrades badly in high dimensions** — at 768D it's essentially brute force

### Why HNSW Wins
KD-Tree pruning relies on axis-aligned bounds. In high dimensions, almost all space is near the hypersphere boundary — nothing gets pruned. HNSW's graph-based approach doesn't suffer from this.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Ollama: OFFLINE` in header | Run `ollama serve` in a terminal |
| Embedding takes forever | Ollama downloads the model on first use — wait ~2 min |
| `g++: command not found` | Add `C:\msys64\ucrt64\bin` to Windows PATH |
| Port 8080 in use | `netstat -ano \| findstr 8080` then `taskkill /PID <pid> /F` |
| LLM answer is slow | Normal on CPU (~10–30s). Use `llama3.2:1b` for faster responses |

### Using a faster LLM

```powershell
ollama pull llama3.2:1b
```

Then change `genModel` in `main.cpp` and recompile:
```cpp
std::string genModel = "llama3.2:1b";
```

---

## License

MIT
