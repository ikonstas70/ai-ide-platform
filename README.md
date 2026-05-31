# Self-Hosted AI Coding Platform

**Infrastructure:** ESXi servers with GPU passthrough  
**Author:** IT Solutions USA  
**Status:** Roadmap / Design Phase

A fully private, self-hosted alternative to Replit — built on your own ESXi
infrastructure with local LLM inference, sandboxed code execution, and a
Monaco Editor browser IDE.

---

## Architecture

```
+----------------------------+
|        Browser IDE         |
|  (React + Monaco Editor)   |
|  - Syntax Highlighting     |
|  - Autocomplete            |
|  - Inline AI Suggestions   |
+------------+---------------+
             |
      HTTPS / WebSocket
             |
+------------v-------------------+
|       Backend API Server       |
|  (FastAPI)                     |
|  - User Auth & Sessions        |
|  - Project Management          |
|  - AI Request Routing          |
+------------+-------------------+
             |
      Internal API Calls
             |
+------------v-------------------+
|        AI LLM Server           |
|  - StarCoder / WizardCoder     |
|  - vLLM inference engine       |
|  - GPU (CUDA passthrough)      |
+------------+-------------------+
             |
      Local Network
             |
+------------v-------------------+
|    Code Execution Layer        |
|  (Docker / Kubernetes)         |
|  - Sandboxed containers        |
|  - Multi-language runtimes     |
|  - CPU / RAM / I/O limits      |
+------------+-------------------+
             |
      Persistent Storage
             |
+------------v-------------------+
|  Project Files & Embeddings    |
|  - ZFS / NVMe Storage          |
|  - FAISS / Milvus embeddings   |
|  - PostgreSQL / MongoDB        |
+--------------------------------+
```

---

## Stack Components

| Component | Technology |
|---|---|
| IDE Frontend | React + Monaco Editor + Tailwind |
| Backend | FastAPI (Python) |
| Code Execution | Docker / Kubernetes |
| AI Models | StarCoder / WizardCoder (7B–15B) |
| AI Serving | vLLM / Hugging Face Transformers |
| Database | PostgreSQL / MongoDB |
| Embeddings | FAISS / Milvus |
| Storage | ZFS / NVMe SSDs |
| Security | Docker sandboxing, VPN, HTTPS, firewall |

---

## ESXi VM Allocation

| VM | Role | CPU | RAM | GPU |
|---|---|---|---|---|
| VM1 | Backend API + Frontend | 8–16 cores | 32–64 GB | No |
| VM2 | Code Execution (Docker) | 16 cores | 64–128 GB | Optional |
| VM3 | AI LLM Server | 8 cores | 64 GB+ | Required (passthrough) |

---

## AI Models

| Model | Size | Notes |
|---|---|---|
| StarCoder / StarCoderBase | 7B–15B | Open-source, built for code completion |
| WizardCoder | 7B–15B | Fine-tuned on coding tasks |
| CodeGen | 6B–16B | Open-source code LLM |

**Serving:**
```python
from vllm import LLM

llm = LLM(model="starcoder-7b")

def generate(prompt):
    return llm.generate(prompt)
```

- 7B model ≈ 24 GB VRAM (RTX 3090 / A100 40GB)
- 13B+ models require multi-GPU setup

---

## Project-Aware AI (Embeddings)

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-mpnet-base-v2")
embeddings = model.encode(list_of_project_files)
# Store in FAISS or Milvus for semantic retrieval
```

At inference time, top-K relevant snippets are retrieved and added to the
LLM prompt — giving the model full project context, similar to Replit Ghostwriter.

---

## Code Execution Sandbox

```bash
docker run \
  --cpus=2 \
  --memory=2g \
  --pids-limit=100 \
  --network=none \
  python:3.11-slim
```

One container per user project. No network access, read-only base OS mounts.

---

## Hardware Recommendations

| Component | Minimum | Recommended |
|---|---|---|
| CPU | 16 cores | 32+ cores |
| RAM | 64 GB | 128 GB+ |
| GPU | 1× RTX 3090 (24 GB) | 2× A100 40 GB |
| Storage | 1 TB SSD | 4 TB+ ZFS / NVMe RAID10 |
| Network | 1 GbE | 10 GbE |

---

## Deployment Steps

1. Configure ESXi VMs for Backend, Execution, and AI nodes
2. Install Docker + Kubernetes on execution VM
3. Deploy vLLM on GPU VM with CUDA passthrough
4. Build FastAPI backend (auth, project storage, AI routing)
5. Deploy Monaco Editor frontend via Nginx
6. Integrate FAISS embeddings for project-aware completions
7. Test sandboxed execution (Python, Node.js, Java)
8. Secure with HTTPS (Nginx reverse proxy) and VPN

---

## Future Roadmap

- Real-time collaboration via WebSocket / WebRTC
- Multi-user Kubernetes scaling with GPU node pools
- Integration with the [network-automation-stack](https://github.com/ikonstas70/network-automation-stack) — AI agents calling the Flask API to manage the Cisco CSR fleet
- Fine-tuned model on network engineering tasks (IOS configs, BGP, OSPF)
