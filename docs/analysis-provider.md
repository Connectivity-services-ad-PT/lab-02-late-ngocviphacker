# Phân tích yêu cầu — vai Provider

- Cặp đàm phán:
- Product: A / B
- Provider service:Notification
- Consumer service:Core Business
- Người viết:Nguyễn Thế Ngọc
- Ngày:20/05/2026

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| Alert Event | Event cảnh báo do Core Business gửi sang Notification | eventId, eventType, alertId, correlationId, severity | channels, metadata |
| Notification Delivery | Thông tin kết quả gửi notification | deliveryId, alertId, channel, status | errorMessage, processedAt |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| POST | `/events/alert.created` | Nhận event tạo alert mới | Khi phát sinh cảnh báo |
| POST | `/events/alert.escalated` | Nhận event nâng mức cảnh báo | Khi severity tăng |
| POST | `/events/alert.resolved` | Nhận event đóng alert | Khi sự cố được xử lý |
| GET | `/notifications/{id}` | Kiểm tra trạng thái notification | Khi cần monitoring/debug |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng | `Problem` |
| 401 | Thiếu Bearer token | `Problem` |
| 403 | Token hợp lệ nhưng không có quyền | `Problem` |
| 404 | Resource không tồn tại | `Problem` |
| 409 | Event bị duplicate do retry | `Problem` |
| 422 | Thiếu correlationId hoặc severity | `Problem` |
| 429 | Queue quá tải | `Problem` |
| 500 | Lỗi gửi Telegram/Email/App | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Mỗi event phải có `eventId` duy nhất.
- Giả định 2: Notification service hỗ trợ retry tối đa 3 lần.
- Giả định 3: Severity dùng enum `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.

---

## 5. Câu hỏi cho Consumer

1. Event `alert.created` có bắt buộc field severity không?
2. Consumer có gửi danh sách channels hay Notification tự routing?
3. Retry sẽ do Queue xử lý hay Notification tự retry?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Field name không thống nhất | Consumer parse lỗi | Chốt schema chung |
| Duplicate event do retry | Gửi notification lặp | Dùng eventId để check |
| Severity hiểu khác nhau | Sai mức cảnh báo | Chuẩn hóa enum severity |
| Queue delay | Alert đến chậm | Retry + Dead Letter Queue |
| Downstream lỗi | Mất notification | Retry policy |
