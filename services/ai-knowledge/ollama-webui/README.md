# Ollama + Open WebUI (Local AI Chat Suite)

Self-hosted, CPU-optimized large language model inference engine with a full-featured, ChatGPT-style chat interface, RAG pipeline, and model manager.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Ollama API** | `http://<SERVER_IP>:11434` |
| **Open WebUI Frontend** | `http://<SERVER_IP>:3001` |
| **Compute Target** | AMD Ryzen 7 5800X (CPU Only) |

---

## CPU-Optimized Model Recommendations

For smooth inference on 8 CPU cores and 16 GB of system RAM without memory pressure:

| Model | Parameters | Command to Download |
| :--- | :--- | :--- |
| **Llama 3.2** | `3B` | `docker exec -it ollama ollama pull llama3.2:3b` |
| **Phi-4 Mini** | `3.8B` | `docker exec -it ollama ollama pull phi4-mini` |
| **Qwen 2.5 Coder** | `3B` / `7B` | `docker exec -it ollama ollama pull qwen2.5-coder:3b` |
| **Gemma 2** | `2B` | `docker exec -it ollama ollama pull gemma2:2b` |
| **Nomic Embed Text** | `137M` *(Embeddings)* | `docker exec -it ollama ollama pull nomic-embed-text` |

---

## Deployment

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Start the AI stack:
   ```bash
   docker compose up -d
   ```
3. Access `http://<SERVER_IP>:3001` to create your initial admin account.
