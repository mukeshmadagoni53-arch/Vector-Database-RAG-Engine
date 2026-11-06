# VectorDB — Build a Vector Database from Scratch in Java

A working **Vector Database** built in Java 11 with a browser dashboard. It implements **HNSW**, **KD-Tree**, and **Brute Force** nearest-neighbour search side-by-side, and includes an Ollama-powered document/RAG workflow.

> An educational project for understanding how vector databases such as Pinecone, Weaviate, Chroma, and Milvus work internally.

---

## What This Project Does

| Feature | Description |
|---|---|
| **3 Search Algorithms** | HNSW, KD-Tree, and Brute Force—run each one and compare elapsed time |
| **3 Distance Metrics** | Cosine, Euclidean, and Manhattan distance |
| **16D Demo Vectors** | 20 semantic vectors in Computer Science, Math, Food, and Sports categories |
| **2D PCA Scatter Plot** | Interactive projection of the vector space in the supplied dashboard |
| **Real Document Embeddings** | Ollama converts pasted text into `nomic-embed-text` embeddings |
| **RAG Pipeline** | Embed a question, retrieve relevant chunks, then generate an answer with Ollama |
| **REST API** | Insert, delete, search, benchmark, document, RAG, and HNSW graph endpoints |

---

## How It Works

```text
Your text
    │
    ▼
Ollama (nomic-embed-text)       → creates an embedding
    │
    ▼
Java VectorDB                    → indexes/searches vectors
    │
    ▼
Relevant document chunks
    │
    ▼
Ollama (llama3.2)               → generates the answer
```

HNSW (Hierarchical Navigable Small World) builds a multilayer graph. Searches start in sparse upper layers and descend toward the closest neighbourhood at layer zero, avoiding a full scan for larger datasets.

---

## Prerequisites

Install the following on Windows:

1. **Java 11 or newer** (JDK—not only a JRE)
2. **Git**
3. **Ollama**—only required for the Documents and Ask AI features

Verify Java and Git:

```powershell
java -version
javac -version
git --version
```

---

## Setup on Windows

### 1. Clone the repository

```powershell
git clone https://github.com/mukeshmadagoni53-arch/Vector-Database-RAG-Engine.git
cd Vector-Database-RAG-Engine
```

### 2. Install Ollama and models (optional for vector demo search)

Install Ollama from [ollama.com/download/windows](https://ollama.com/download/windows), then open a new PowerShell window:

```powershell
ollama pull nomic-embed-text
ollama pull llama3.2
ollama list
```

`nomic-embed-text` creates the embeddings; `llama3.2` writes answers. Ollama normally runs in the background after installation. If it is not running, use:

```powershell
ollama serve
```

### 3. Compile the Java server

From the project directory:

```powershell
javac -d out src\main\java\com\vectordb\Main.java
```

No Maven, Gradle, or third-party Java libraries are required. The application uses the built-in JDK HTTP server.

### 4. Run the server

```powershell
java -cp out com.vectordb.Main
```

Expected startup output:

```text
=== VectorDB Java Engine ===
http://localhost:8080
20 demo vectors | 16 dims | HNSW+KD-Tree+BruteForce
Ollama: ONLINE
```

Open [http://localhost:8080](http://localhost:8080).

If port 8080 is already used, run it on a different port:

```powershell
java -cp out com.vectordb.Main 8090
```

Then open [http://localhost:8090](http://localhost:8090).

---

## Using the Application

### Search demo vectors

- Enter a concept such as `binary tree`, `sushi`, `basketball`, or `calculus`.
- Select **HNSW**, **KD-Tree**, or **Brute Force**.
- Select Cosine, Euclidean, or Manhattan distance.
- Click **Search** to see nearest neighbours and distances.
- Click **Compare all algorithms** to run the same query through all three indexes.

The dashboard projects the 20 demo vectors into two dimensions with PCA, showing the semantic clusters visually.

### Documents and RAG

1. Supply a document title and text.
2. Click **Embed & Insert**.
3. Long text is split into 250-word chunks with a 30-word overlap.
4. Ask a question in the RAG panel.

The server embeds the question, retrieves similar document chunks by cosine distance, and sends those chunks to `llama3.2` with the question.

---

## REST API

The API is served at `http://localhost:8080` (or the selected port).

### Demo vector endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/search?v=f1,f2,...&k=5&metric=cosine&algo=hnsw` | Run K-NN search |
| `POST` | `/insert` | Insert a 16D vector |
| `DELETE` | `/delete/:id` | Remove a demo vector |
| `GET` | `/items` | List demo vectors |
| `GET` | `/benchmark?v=...&k=5&metric=cosine` | Compare the three algorithms |
| `GET` | `/hnsw-info` | Get HNSW layers, nodes, and edges |
| `GET` | `/status` | Get Ollama and document status |

### Document and RAG endpoints

| Method | Endpoint | Body | Description |
|---|---|---|---|
| `POST` | `/doc/insert` | `{"title":"...","text":"..."}` | Embed and store chunks |
| `GET` | `/doc/list` | — | List stored chunks |
| `DELETE` | `/doc/delete/:id` | — | Delete a chunk |
| `POST` | `/doc/search` | `{"question":"...","k":3}` | Retrieve relevant chunks |
| `POST` | `/doc/ask` | `{"question":"...","k":3}` | Run the complete RAG pipeline |

### Example request

```powershell
curl "http://localhost:8080/search?v=0.9,0.8,0.7,0.6,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1&k=3&metric=cosine&algo=hnsw"
```

---

## Project Structure

```text
java-vectordb/
├── index.html                         ← browser dashboard (PCA, chat, benchmark)
├── src/main/java/com/vectordb/
│   └── Main.java                      ← Java backend, indexes, REST API, RAG
├── out/                               ← compiled classes (created locally)
└── README.md
```

## Architecture

```text
BruteForce      O(N × d)      Exact baseline
KDTree          Typically fast at low dimensions; exact axis partitioning
Hnsw            Approximate multilayer small-world graph

DB              Keeps the three 16D demo indexes in sync
Docs            Stores Ollama-generated document embeddings
Ollama client   Calls /api/embeddings and /api/generate on localhost:11434
```

## Algorithm Notes

### Brute Force

Calculates the selected distance from the query to every vector, sorts all candidates, and returns the closest `k`. It is exact and provides the comparison baseline.

### KD-Tree

Recursively splits the 16-dimensional space by cycling the split axis. Search visits the nearer branch first and visits the other branch only when it could still contain a better result.

### HNSW

Each inserted vector receives a random maximum level and connects to nearby vectors at each level. Search greedily descends the upper layers, then uses a beam search at level zero. The implementation uses deterministic random seeding (`42`) to keep demonstrations reproducible.

## Common Issues

| Problem | Fix |
|---|---|
| `ollama` is not recognized | Install Ollama, close PowerShell, then open a new terminal |
| Dashboard reports Ollama offline | Start it with `ollama serve` and check `ollama list` |
| Port 8080 is in use | Run `java -cp out com.vectordb.Main 8090` |
| `javac` is not recognized | Install a Java JDK and add its `bin` directory to PATH |
| RAG answer is slow | `llama3.2` can take time on CPU; use a smaller installed model if needed |

## License

MIT — use, study, and adapt it freely.
