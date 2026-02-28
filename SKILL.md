---
name: pancake
description: "[CK] Query Pancake POS & Chat API directly (no MCP). Use when user asks to fetch orders, conversations, warehouses, or any Pancake data. Requires PANCAKE_API_KEY env var (and optionally PANCAKE_ACCESS_TOKEN for chat)."
---

# Pancake Direct API Skill

Query Pancake POS and Chat APIs directly via Python scripts — no MCP server needed.

## When to Use

- User asks for Pancake data: orders, warehouses, conversations, messages, geo data
- User wants to query Pancake without starting MCP server
- Quick data lookups and reporting from Pancake
- Debugging/testing Pancake API connectivity

## Prerequisites

Set env vars before running scripts:
```bash
export PANCAKE_API_KEY="your_pos_api_key"
export PANCAKE_ACCESS_TOKEN="your_chat_token"  # optional, falls back to API key
```

Or read from `.env` file in the project directory.

## Python Client Location

The Pancake API clients live at:
- **POS API:** `/cowork_mcp/pancake-mcp-server/src/pancake_mcp/client.py`
- **Chat API:** `/cowork_mcp/pancake-mcp-server/src/pancake_mcp/chat_client.py`

## How to Run Scripts

Use the venv Python from the pancake-mcp-server package (if installed):

```bash
# Option 1: If package is installed
cd /cowork_mcp/pancake-mcp-server && python3 -c "..."

# Option 2: Use the skill scripts directly
~/.claude/skills/.venv/bin/python3 ~/.claude/skills/pancake/scripts/query.py [args]

# Option 3: Inline async Python via bash
python3 - <<'EOF'
import asyncio, os, sys
sys.path.insert(0, '/cowork_mcp/pancake-mcp-server/src')
from pancake_mcp.client import PancakeClient

async def main():
    key = os.environ.get('PANCAKE_API_KEY', '')
    if not key:
        print("Error: PANCAKE_API_KEY not set")
        return
    async with PancakeClient(key) as c:
        result = await c.get_shops()
        import json
        print(json.dumps(result, ensure_ascii=False, indent=2))

asyncio.run(main())
EOF
```

## Available API Methods

### POS API (`PancakeClient`) — Base: `https://pos.pages.fm/api/v1`

| Method | Description |
|--------|-------------|
| `get_shops()` | List all shops (get shop_id from here first) |
| `get_payment_methods(shop_id)` | Bank payment methods |
| `get_provinces(country_code="VN")` | Vietnam provinces |
| `get_districts(province_id)` | Districts in a province |
| `get_communes(district_id)` | Communes in a district |
| `list_orders(shop_id, **filters)` | Search orders (status, page, page_size, keyword, from_date, to_date) |
| `get_order(shop_id, order_id)` | Single order details |
| `create_order(shop_id, payload)` | Create new order |
| `update_order(shop_id, order_id, payload)` | Update order |
| `get_order_tags(shop_id)` | Available order tags |
| `get_order_sources(shop_id)` | Order sources (Facebook, Website...) |
| `get_active_promotions(shop_id, payload)` | Active promotions for items |
| `arrange_shipment(shop_id, payload)` | Hand order to carrier |
| `get_tracking_url(shop_id, payload)` | Customer tracking link |
| `list_warehouses(shop_id)` | All warehouses |
| `create_warehouse(shop_id, payload)` | Create warehouse |
| `update_warehouse(shop_id, warehouse_id, payload)` | Update warehouse |
| `get_inventory_history(shop_id, **filters)` | Stock movement history |
| `list_return_orders(shop_id, **filters)` | Return/exchange orders |
| `create_return_order(shop_id, payload)` | Process a return |

### Chat API (`PancakeChatClient`) — Base: `https://pages.fm/api/v1`

| Method | Description |
|--------|-------------|
| `list_conversations(page_id, **filters)` | List inbox conversations |
| `get_conversation(page_id, conversation_id)` | Conversation details |
| `update_conversation(page_id, conversation_id, payload)` | Assign/tag/close |
| `get_messages(page_id, conversation_id, **params)` | Message history |
| `send_message(page_id, conversation_id, payload)` | Send a message |
| `get_customer(page_id, psid)` | Customer profile by PSID |

## Common Query Patterns

### Get shops & orders
```python
import asyncio, os, sys, json
sys.path.insert(0, '/cowork_mcp/pancake-mcp-server/src')
from pancake_mcp.client import PancakeClient

async def main():
    async with PancakeClient(os.environ['PANCAKE_API_KEY']) as c:
        shops = await c.get_shops()
        shop_id = shops['data'][0]['id']
        orders = await c.list_orders(shop_id, status='new', page=1, page_size=20)
        print(json.dumps(orders, ensure_ascii=False, indent=2))

asyncio.run(main())
```

### List conversations
```python
import asyncio, os, sys, json
sys.path.insert(0, '/cowork_mcp/pancake-mcp-server/src')
from pancake_mcp.chat_client import PancakeChatClient

async def main():
    token = os.environ.get('PANCAKE_ACCESS_TOKEN') or os.environ['PANCAKE_API_KEY']
    async with PancakeChatClient(token) as c:
        convs = await c.list_conversations('YOUR_PAGE_ID', status='open', page_size=20)
        print(json.dumps(convs, ensure_ascii=False, indent=2))

asyncio.run(main())
```

## Error Handling

`PancakeAPIError` is raised for non-2xx responses:
```python
from pancake_mcp.client import PancakeAPIError
try:
    result = await c.get_shops()
except PancakeAPIError as e:
    print(f"API error {e.status_code}: {e}")
```

## Implementation Steps

When user asks for Pancake data:

1. **Check env vars** — verify `PANCAKE_API_KEY` is set (check `.env` file if needed)
2. **Write inline Python** — use the pattern above with `sys.path.insert` to import clients
3. **Run via Bash tool** — execute the script and capture output
4. **Parse & present** — format the JSON response for the user
5. **Handle errors** — if `PancakeAPIError`, check API key validity or endpoint availability

## Loading .env

If `PANCAKE_API_KEY` not in env, check for `.env` file:
```python
import os
from pathlib import Path

env_file = Path('/cowork_mcp/.env')
if env_file.exists():
    for line in env_file.read_text().splitlines():
        if '=' in line and not line.startswith('#'):
            k, v = line.split('=', 1)
            os.environ.setdefault(k.strip(), v.strip())
```
