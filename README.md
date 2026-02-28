# cowork-pancake-skills

Claude Cowork skill để truy vấn **Pancake POS & Chat API** trực tiếp — Claude tự viết và chạy Python inline, không cần MCP server.

## Cài đặt

```bash
npx skills add lynguyenvu/cowork-pancake-skills -g -y
```

## Cách hoạt động

Khi bạn hỏi Claude về dữ liệu Pancake, skill này hướng dẫn Claude:

1. Tự sinh Python code dùng `httpx` để gọi Pancake API
2. Chạy code qua Bash tool ngay trong session
3. Parse JSON và trả kết quả cho bạn

Không cần MCP server đang chạy. Không cần cài thêm package ngoài `httpx`.

## Yêu cầu

Set env var trước khi dùng:

```bash
export PANCAKE_API_KEY="your_pos_api_key"
export PANCAKE_ACCESS_TOKEN="your_chat_token"  # tùy chọn
```

Hoặc tạo file `.env` trong thư mục làm việc:

```env
PANCAKE_API_KEY=your_pos_api_key
PANCAKE_ACCESS_TOKEN=your_chat_token
```

Lấy key tại: Pancake → **Cài đặt → Nâng cao → Kết nối bên thứ 3 → Webhook/API**

## Ví dụ

```
Lấy danh sách đơn hàng mới hôm nay
Xem inbox Facebook chưa xử lý của page "xxx"
Tạo đơn hàng mới cho khách Nguyễn Văn A, SĐT 0901234567
Lịch sử nhập xuất kho tháng này
```

## API được hỗ trợ

| Nhóm | Endpoints |
|------|-----------|
| 🏪 Shop | Danh sách shop, phương thức thanh toán |
| 📦 Đơn hàng | Tìm kiếm, xem, tạo, cập nhật, tags, sources, khuyến mãi |
| 🏭 Kho hàng | Danh sách, tạo, cập nhật kho, lịch sử tồn kho |
| 🚚 Vận chuyển | Tạo vận đơn, tracking URL, đơn hoàn |
| 💬 Hội thoại | Inbox, tin nhắn, gửi reply, gắn tag, chuyển xử lý |
| 🗺️ Địa lý VN | Tỉnh thành, quận huyện, phường xã |

## Liên quan

- [pancake-mcp-server](https://github.com/lynguyenvu/pancake-mcp-server) — MCP server đầy đủ cho Claude Desktop & Claude.ai custom connector
