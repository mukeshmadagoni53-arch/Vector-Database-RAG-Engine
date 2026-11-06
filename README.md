# VectorDB Java

Java 11 port of the C++ VectorDB. It has the same REST API, 20 demo vectors,
three selectable indexes, and optional Ollama document/RAG features.

The indexes are independently implemented:

- **Brute Force** — exact linear K-NN baseline.
- **KD-Tree** — recursive 16-dimensional axis partitioning with backtracking.
- **HNSW** — a deterministic, multilayer small-world graph with beam search.

Run from this folder:

```powershell
javac -d out src/main/java/com/vectordb/Main.java
java -cp out com.vectordb.Main
```

Open http://localhost:8080. Documents and Ask AI require Ollama with
`nomic-embed-text` and `llama3.2` installed.

If port 8080 is already in use, choose another port (for example 8090):

```powershell
java -cp out com.vectordb.Main 8090
```
