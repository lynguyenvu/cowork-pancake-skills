---
name: pancake
description: "[CK] Query Pancake POS & Chat API directly (no MCP). Use when user asks to fetch orders, conversations, warehouses, or any Pancake data. Requires PANCAKE_API_KEY env var (and optionally PANCAKE_ACCESS_TOKEN for chat)."
---

# Pancake Direct API Skill

Query Pancake POS and Chat APIs directly via inline Python — **no MCP server, no imports needed**.

## When to Use

- User asks for Pancake data: orders, warehouses, conversations, messages, geo data
- Quick data lookups and reporting from Pancake
- Any Pancake query without MCP protocol overhead

## Prerequisites

Check for API key (in order of precedence):
1. `PANCAKE_API_KEY` env var
2. `.env` file in current directory or `/cowork_mcp/.env`

```python
import os
from pathlib import Path

def load_env():
    for env_path in [Path('.env'), Path('/cowork_mcp/.env')]:
        if env_path.exists():
            for line in env_path.read_text().splitlines():
                if '=' in line and not line.startswith('#'):
                    k, v = line.split('=', 1)
                    os.environ.setdefault(k.strip(), v.strip())

load_env()
api_key = os.environ.get('PANCAKE_API_KEY', '')
chat_token = os.environ.get('PANCAKE_ACCESS_TOKEN', '') or api_key
```

## Base URLs & Auth

- **POS API:** `https://pos.pages.fm/api/v1` — auth via `?api_key=<key>` query param
- **Chat API:** `https://pages.fm/api/v1` — auth via `?access_token=<token>` query param

## Self-Contained Query Template

Always use this pattern — no external imports needed beyond `httpx`:

```python
import asyncio, os, json
from pathlib import Path
import httpx

# Load env
def load_env():
    for p in [Path('.env'), Path('/cowork_mcp/.env')]:
        if p.exists():
            for line in p.read_text().splitlines():
                if '=' in line and not line.startswith('#'):
                    k, v = line.split('=', 1)
                    os.environ.setdefault(k.strip(), v.strip())

load_env()

POS_BASE = "https://pos.pages.fm/api/v1"
CHAT_BASE = "https://pages.fm/api/v1"
API_KEY = os.environ.get('PANCAKE_API_KEY', '')
CHAT_TOKEN = os.environ.get('PANCAKE_ACCESS_TOKEN', '') or API_KEY

if not API_KEY:
    print("Error: PANCAKE_API_KEY not set")
    exit(1)

async def get(base, path, token_param, token, **params):
    async with httpx.AsyncClient(timeout=30) as c:
        r = await c.get(f"{base}{path}", params={token_param: token, **{k: v for k, v in params.items() if v is not None}})
        r.raise_for_status()
        return r.json()

async def post(base, path, token_param, token, body):
    async with httpx.AsyncClient(timeout=30) as c:
        r = await c.post(f"{base}{path}", params={token_param: token}, json=body)
        r.raise_for_status()
        return r.json()

async def main():
    # === REPLACE THIS SECTION WITH YOUR QUERY ===
    result = await get(POS_BASE, "/shops", "api_key", API_KEY)
    print(json.dumps(result, ensure_ascii=False, indent=2))

asyncio.run(main())
```

## Available Endpoints

### POS API (`https://pos.pages.fm/api/v1`, auth: `api_key`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/shops` | GET | List shops → get `shop_id` |
| `/shops/{shop_id}/bank_payments` | GET | Payment methods |
| `/geo/provinces` | GET | Vietnam provinces (`?country_code=VN`) |
| `/geo/districts` | GET | Districts (`?province_id=`) |
| `/geo/communes` | GET | Communes (`?district_id=`) |
| `/shops/{shop_id}/orders` | GET | List orders (`?status=&page=&page_size=&keyword=&from_date=&to_date=`) |
| `/shops/{shop_id}/orders/{order_id}` | GET | Single order details |
| `/shops/{shop_id}/orders` | POST | Create order |
| `/shops/{shop_id}/orders/{order_id}` | PUT | Update order |
| `/shops/{shop_id}/orders/tags` | GET | Order tags |
| `/shops/{shop_id}/order_source` | GET | Order sources |
| `/shops/{shop_id}/orders/arrange_shipment` | POST | Hand to carrier |
| `/shops/{shop_id}/orders/get_tracking_url` | POST | Tracking URL |
| `/shops/{shop_id}/orders/get_promotion_advance_active` | POST | Active promotions |
| `/shops/{shop_id}/warehouses` | GET | List warehouses |
| `/shops/{shop_id}/warehouses` | POST | Create warehouse |
| `/shops/{shop_id}/warehouses/{warehouse_id}` | PUT | Update warehouse |
| `/shops/{shop_id}/inventory_histories` | GET | Stock history |
| `/shops/{shop_id}/orders_returned` | GET | Return orders |
| `/shops/{shop_id}/orders_returned` | POST | Create return |

### Chat API (`https://pages.fm/api/v1`, auth: `access_token`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/pages/{page_id}/conversations` | GET | List conversations (`?type=INBOX&status=open&keyword=&page=`) |
| `/pages/{page_id}/conversations/{conv_id}` | GET | Conversation details |
| `/pages/{page_id}/conversations/{conv_id}` | PUT | Update (assign/tag/close) |
| `/pages/{page_id}/conversations/{conv_id}/messages` | GET | Message history |
| `/pages/{page_id}/conversations/{conv_id}/messages` | POST | Send message |

## Common Query Examples

### List new orders today
```python
from datetime import date
result = await get(POS_BASE, f"/shops/{shop_id}/orders", "api_key", API_KEY,
    status="new", from_date=str(date.today()), page=1, page_size=20)
```

### Get a shop's ID first
```python
shops = await get(POS_BASE, "/shops", "api_key", API_KEY)
shop_id = shops['data'][0]['id']
```

### List open conversations
```python
result = await get(CHAT_BASE, f"/pages/{page_id}/conversations", "access_token", CHAT_TOKEN,
    status="open", type="INBOX", page_size=20)
```

### Send a message
```python
result = await post(CHAT_BASE, f"/pages/{page_id}/conversations/{conv_id}/messages",
    "access_token", CHAT_TOKEN, {"message": "Xin chào!"})
```

### Create an order
```python
result = await post(POS_BASE, f"/shops/{shop_id}/orders", "api_key", API_KEY, {
    "customer_name": "Nguyễn Văn A",
    "customer_phone": "0901234567",
    "items": [{"product_id": "abc123", "quantity": 2}]
})
```

## Implementation Steps

When user asks for Pancake data:

1. **Load env** — check `PANCAKE_API_KEY` from env or `.env` file
2. **Use template above** — fill in the endpoint and params for the specific query
3. **Run via Bash tool** — `python3 -c "..."` or heredoc `python3 - <<'EOF' ... EOF`
4. **Parse result** — extract relevant fields from JSON and present clearly
5. **Error handling** — `httpx.HTTPStatusError` for API errors, check status code

## Error Codes

| Status | Meaning |
|--------|---------|
| 401 | Invalid API key |
| 403 | No permission |
| 404 | Resource not found |
| 422 | Invalid request body |
| 429 | Rate limit exceeded — wait and retry |
