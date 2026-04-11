# Medium Workload MCP EC2 Instance

## Overview

This MCP server is deployed on an AWS EC2 t3.micro instance and runs the **M1–M10 sequential tool chain workloads** defined in the benchmark specification. It serves as one half of the EC2 vs Lambda performance comparison, collecting real infrastructure metrics under medium-tier workloads so both platforms can be evaluated under identical conditions.

---

## What It Is

A Python-based MCP server built with FastMCP, exposing tools that Claude (via custom connector) can call over HTTP using the MCP JSON-RPC protocol. The medium workload tier is defined as:

- **2–3 sequential external HTTP calls** per tool chain
- **5–100 KB** combined response payloads
- **Non-trivial local computation** — mean, median, mode, sort operations applied to returned data
- **Real data dependencies** between chained calls — later calls require values returned by earlier ones, making chains strictly sequential

---

## How It Works

Every Claude request follows the standard MCP JSON-RPC flow: `initialize` → `tools/list` → `tools/call`. Each medium workload executes as a sequential chain where identifiers from earlier responses feed into later calls. For example, M2 fetches a list of National Parks, extracts a `parkCode`, then uses it to fetch full park details before running statistical operations on the combined data.

All tool responses return structured JSON:

```json
{
  "request_id": "string",
  "status": "success" | "error",
  "result": {},
  "duration_ms": 0
}
```

The server also exposes `route_backend` and `compare_backends` tools to dispatch the same payload to both EC2 and Lambda and return side-by-side results.

---

## EC2 vs Lambda Comparison

M1–M10 chains are run on both this EC2 instance and a Lambda equivalent. The following metrics are captured per tool chain execution on each platform:

| Metric | Description |
|---|---|
| **Duration (ms)** | Server-side execution time per tool call |
| **Request/Response Bytes** | Payload size in and out per chain step |
| **RAM Usage** | Memory consumption during execution |
| **EC2 Cost** | Running cost of the MCP server for this instance |
| **Content Preview** | Sample of returned data to verify correctness across platforms |

---

## Deployment

### Requirements
- Python 3.13+, `mcp >= 1.2.0`, `fastmcp`, `uvicorn`, `requests`

### Run
```bash
pip install -e .
python server.py
```

Server starts at `http://0.0.0.0:8000/mcp`. Add the CloudFront URL to your Claude custom connector to connect.
