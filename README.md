# GenAI

A hands-on generative AI learning lab covering tokenization, embeddings, chat completions, agent loops, retrieval-augmented generation, LangGraph workflows, human-in-the-loop execution, memory, background queues, MCP tooling, and observability.

The repository is organized as small, focused examples. Each folder demonstrates one concept and keeps the code close to the underlying APIs so it is easy to study, modify, and extend.

## What This Repo Covers

```mermaid
flowchart LR
    A["Prompting and Chat"] --> B["Embeddings"]
    B --> C["Vector Search"]
    C --> D["RAG"]
    A --> E["Tools and Agents"]
    E --> F["LangGraph Workflows"]
    F --> G["Human Review"]
    F --> H["Memory"]
    D --> I["Async Queue"]
    E --> J["MCP Server"]
    F --> K["Vibe Coding Assistant"]
    A --> L["Langfuse Observability"]
```

## Repository Map

| Path | Purpose |
| --- | --- |
| `01-tokenization/` | Tokenization demo using `tiktoken` for GPT-style models. |
| `02-vector-embeddings/` | OpenAI embedding generation with `text-embedding-3-small`. |
| `03-hello-world/` | Chat completion examples for zero-shot, few-shot, persona, and structured prompting. |
| `04-agent/` | Custom agent loop with plan, action, observation, and output steps. |
| `05-rag-1/` | PDF ingestion, chunking, Qdrant vector storage, and RAG chat over `nodejs.pdf`. |
| `06_langraph/` | Basic LangGraph chatbot plus a routed coding-question graph. |
| `08_tool/` | LangGraph tool-calling example with todos, math, and weather tools. |
| `09_human_in_loop/` | LangGraph interrupt flow for human assistance and resume. |
| `chat_graph/` | MongoDB-checkpointed chat graph with Langfuse callback tracing. |
| `memory/` | Memory-aware assistant using OpenAI, Qdrant, Neo4j, and Mem0. |
| `rag_queue/` | FastAPI and RQ-based async query processing for RAG-style responses. |
| `langfuse/` | Docker Compose stack for local Langfuse observability. |
| `mcp.js` | Minimal Model Context Protocol server with an `addTwoNumbers` tool. |
| `vibe_coder/` | Voice/code assistant experiment using LangGraph tools and OpenAI TTS. |

## Prerequisites

- Python 3.11+
- Node.js 18+
- Docker and Docker Compose for Qdrant, Valkey/Redis, Langfuse, and related services
- An OpenAI API key

## Quick Start

```bash
git clone https://github.com/devthedevil/GenAI.git
cd GenAI

python -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
pip install openai python-dotenv tiktoken langchain-openai langchain-community langchain-qdrant langgraph langgraph-checkpoint-mongodb langfuse redis rq fastapi uvicorn mem0ai qdrant-client

npm install
export OPENAI_API_KEY="your_openai_api_key"
```

Some examples require additional services or environment variables. Keep secrets in a local `.env` file and do not commit them.

```bash
OPENAI_API_KEY=your_openai_api_key
LANGCHAIN_TRACING_V2=false
LANGCHAIN_API_KEY=
LANGFUSE_PUBLIC_KEY=
LANGFUSE_SECRET_KEY=
LANGFUSE_HOST=http://localhost:3000
```

## Run the Examples

### Tokenization

```bash
python 01-tokenization/main.py
```

Shows how text is converted into model tokens and decoded back into text.

### Embeddings

```bash
python 02-vector-embeddings/main.py
```

Creates an embedding vector for sample text using OpenAI embeddings.

### Chat Prompting

```bash
python 03-hello-world/chat.py
python 03-hello-world/chat-02.py
python 03-hello-world/chat-03.py
python 03-hello-world/chat-cot-03.py
```

Demonstrates chat completion patterns such as system instructions, examples, persona prompting, and structured JSON responses.

### Custom Agent Loop

```bash
python 04-agent/main.py
```

Runs an interactive agent that plans, selects a tool, observes the result, and returns a final answer.

Available tools:

- `get_weather`: fetches weather from `wttr.in`
- `run_command`: executes a shell command locally

Use `run_command` only in trusted local experiments. Do not expose it to untrusted users.

### RAG With Qdrant

Start Qdrant:

```bash
docker compose -f 05-rag-1/docker-compose.yml up -d
```

Index the sample PDF and ask questions:

```bash
python 05-rag-1/indexing.py
python 05-rag-1/chat.py
```

The RAG demo loads `05-rag-1/nodejs.pdf`, chunks it, stores embeddings in Qdrant, retrieves relevant chunks, and answers with source page context.

Note: `chat.py` points to `http://localhost:6333`, while `indexing.py` currently points to `http://vector-db:6333`. If you run `indexing.py` from your host machine, update that URL to `http://localhost:6333` or run the script from a container attached to the Compose network.

### LangGraph Basics

```bash
python 06_langraph/graph.py
python 06_langraph/code_graph.py
```

`graph.py` is a minimal one-node graph. `code_graph.py` classifies whether a query is coding-related, routes it to the correct node, and validates coding answers.

### LangGraph Tools

```bash
python 08_tool/graph.py
```

Runs an interactive LangGraph assistant with callable tools for todos, addition, and weather lookup.

### Human In The Loop

```bash
python 09_human_in_loop/graph.py
```

Demonstrates LangGraph interrupts, checkpointing, and resume commands for human assistance flows.

This example expects MongoDB at:

```text
mongodb://admin:admin@mongodb:27017
```

Update the URI if you are running MongoDB locally or with a different Docker service name.

### Chat Graph With Checkpointing

```bash
python chat_graph/graph.py
```

Runs a MongoDB-checkpointed LangGraph chat flow with Langfuse callbacks.

### Memory-Aware Assistant

```bash
python memory/main.py
```

Runs a Mem0-based assistant that searches prior memories, injects relevant context into the system prompt, and stores new interactions.

This example expects Qdrant and Neo4j services. Update the service hostnames if you are running them outside Docker.

### Async RAG Queue

Start Valkey:

```bash
docker compose -f rag_queue/docker-compose.yml up -d
```

Run the API:

```bash
python -m rag_queue.main
```

Start a worker in an environment that can reach Valkey and Qdrant:

```bash
rq worker --with-scheduler --url redis://localhost:6379
```

Submit a query:

```bash
curl -X POST "http://localhost:8000/chat?query=What%20is%20Node.js"
```

Fetch a result:

```bash
curl "http://localhost:8000/result/<job_id>"
```

### Langfuse

```bash
docker compose -f langfuse/docker-compose.yml up -d
```

Open Langfuse at:

```text
http://localhost:3000
```

The Compose file includes development defaults. Change secrets and passwords before using it outside local experiments.

### MCP Server

```bash
node mcp.js
```

The MCP server exposes one tool:

- `addTwoNumbers`: accepts `a` and `b`, then returns their sum.

Example MCP client config:

```json
{
  "mcpServers": {
    "chaicode-server": {
      "command": "node",
      "args": ["mcp.js"]
    }
  }
}
```

### Vibe Coder

```bash
cd vibe_coder
pip install -r requirements.txt
python -m app.main
```

This experiment combines speech recognition, OpenAI TTS, LangGraph tool calls, and command execution for a local coding-assistant workflow.

## Development Notes

- Keep API keys and service credentials in `.env`.
- Several folders are workshop-style experiments and may require small hostname changes depending on whether you run scripts locally or inside Docker.
- `run_command` tools execute commands on the local machine. Keep them private and trusted.
- The root `requirements.txt` does not include every package used by every experiment, so the quick-start command installs the common optional dependencies used across the examples.

## Suggested Next Improvements

- Add `.env.example` with required variables for each module.
- Split dependencies into per-folder requirements files.
- Add Docker Compose services for MongoDB, Neo4j, Qdrant, and the queue worker.
- Add screenshots or short recordings for the LangGraph and Vibe Coder demos.
- Add tests for reusable helpers and API endpoints.

## License

No license file is currently included.
