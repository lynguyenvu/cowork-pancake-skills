# cowork-pancake-skills

Claude Cowork skill để truy vấn **Pancake POS & Chat API** trực tiếp — không cần MCP server đang chạy.

## Cài đặt

```bash
npx skills add lynguyenvu/cowork-pancake-skills@pancake -g -y
```

## Yêu cầu

- [pancake-mcp-server](https://github.com/lynguyenvu/pancake-mcp-server) đã được clone/cài đặt
- `PANCAKE_API_KEY` env var (lấy từ Pancake → Cài đặt → Nâng cao → Webhook/API)
- `PANCAKE_ACCESS_TOKEN` env var (tùy chọn, cho Chat/Inbox API)

## Tính năng

Skill này hướng dẫn Claude truy vấn trực tiếp qua Python, bao gồm:

| API | Công cụ |
|-----|---------|
| 🏪 Shop | get_shops, get_payment_methods |
| 📦 Đơn hàng | list_orders, get_order, create_order, update_order, get_order_tags, get_order_sources |
| 🏭 Kho hàng | list_warehouses, create_warehouse, update_warehouse, get_inventory_history |
| 🚚 Vận chuyển | arrange_shipment, get_tracking_url, list_return_orders, create_return_order |
| 💬 Hội thoại | list_conversations, get_conversation, get_messages, send_message, update_conversation |
| 🗺️ Địa lý VN | get_provinces, get_districts, get_communes |

## Ví dụ sử dụng

Sau khi cài skill, chat với Claude:

```
Lấy danh sách đơn hàng mới hôm nay từ Pancake
Xem inbox Facebook chưa xử lý của page ID "xxx"
Tạo đơn hàng mới cho khách Nguyễn Văn A
```

## Liên quan

- [pancake-mcp-server](https://github.com/lynguyenvu/pancake-mcp-server) — MCP server đầy đủ cho Claude Desktop & Claude.ai
