# repomix-mix-vfs — Expanded prototypes

This repository contains prototypes for:
- Repomix packer (TypeScript) with multi-language parsing (TypeScript, Python, Go).
- VFS server (TypeScript) using tree-sitter parsing, persistent SQLite DB, and semantic search.
- MCP-style JSON-RPC adapter with policy enforcement and API key auth.

Goals:
- Demonstrate a production-ish flow: repomix → vfs → LLM (via MCP) → editor (VS Code).
- Provide a clear, extendable foundation to iterate to production.

Prereqs
- Node 18+
- npm or yarn
- Rust/build tools (some tree-sitter grammars require native buildchain)
- SQLite (bundled as a binary dependency via Node packages, no external DB required)
- Optional: OPENAI_API_KEY (for real embeddings)

Top-level layout
- repomix-packer/ — TypeScript packer CLI
- vfs-server/ — Node HTTP server (indexer, API, MCP adapter)
- vscode-extension/ — (previous prototype) integrate with HTTP VFS API

Quickstart (local dev)

1) Install dependencies and build repomix packer
cd repomix-packer
npm install
npm run build

2) Use repomix packer (example)
# pack the repo's src + pkg directories, compress file contents
node dist/index.js --paths src,lib --output ../repo.rmx.json.gz --compress --token-budget 150000

3) Install & run vfs-server
cd ../vfs-server
npm install
npm run build

# optional: set API key and OpenAI key for real embeddings
export VFS_API_KEY="devkey123"
export OPENAI_API_KEY="sk-..."

# start indexer & server for your repo
node dist/server.js --dir ../path-to-your-repo --port 8080

4) Example HTTP / MCP usage
# search (text)
curl -H "x-api-key: devkey123" "http://localhost:8080/search?q=req.session&limit=20"

# search (semantic)
curl -H "x-api-key: devkey123" "http://localhost:8080/search?q=authenticate%20user&semantic=1&limit=10"

# read snippet
curl -H "x-api-key: devkey123" "http://localhost:8080/read?path=src/middleware/auth.ts&start=1&end=200"

# MCP JSON-RPC example
curl -H "Content-Type: application/json" -H "x-api-key: devkey123" -d '{
  "jsonrpc":"2.0","id":"1","method":"vfs.search","params":{"q":"req.session","limit":10}
}' http://localhost:8080/mcp

Notes & next steps
- This prototype stores vectors (JSON arrays) in SQLite and performs brute-force kNN; replace with a vector DB for scale.
- Add secret redaction / exclusions in packer (recommended before packing private repos).
- Add authentication & TLS if exposing VFS to cloud or external LLM processes.
- For high-quality parsing across languages, integrate optional language-specific extractors or LSIF.

If you want, I can:
- Add CI to generate repomix artifacts per push and store as GitHub artifacts.
- Replace brute-force vector search with a small HNSW index and a persistent vector DB.
- Harden the MCP adapter (rate-limiting, logging, RBAC).