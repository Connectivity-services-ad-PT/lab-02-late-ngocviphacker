# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: 6A / 7A
- Product: Smart Campus Operations Platform
- Consumer service: Core Business Service
- Provider service: Notification Service
- Người viết: Nguyễn Thế Ngọc
- Ngày: 19/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| Alert Event | Gửi thông tin cảnh báo đến Notification service | eventId, eventType, alertId, correlationId, severity, timestamp, payload | channels, channelDetails, metadata |
| Notification Delivery | Kiểm tra trạng thái gửi thông báo | deliveryId, alertId, eventId, channel, status, sentAt | deliveredAt, errorMessage |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/events/alert.created` | Khi tạo alert mới trong Core Business | 202 queued, echo eventId |
| POST | `/events/alert.escalated` | Khi alert được nâng cấp severity | 202 queued, giữ eventId và status |
| POST | `/events/alert.resolved` | Khi alert đã được giải quyết | 202 queued, xác nhận event received |
| GET | `/notifications/{notificationId}` | Khi cần kiểm tra kết quả gửi notification | 200 với trạng thái delivery hoặc 404 nếu không tồn tại |

---

## 3. Error case Consumer cần xử lý

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Payload sai định dạng hoặc JSON malformed | Ghi log, sửa cấu trúc payload và retry nếu cần |
| 401 | Thiếu hoặc token không hợp lệ | Kiểm tra cấu hình auth và lấy lại token |
| 403 | Service không có quyền gửi alert | Báo lỗi bảo mật, thông báo admin hoặc retry với credentials khác |
| 404 | Notification delivery không tồn tại | Xác nhận notificationId hoặc báo trạng thái chưa có |
| 409 | Duplicate event do retry | Dừng gửi lại event, dùng eventId mới cho event mới |
| 422 | Vi phạm business rule (thiếu severity/correlationId) | Cập nhật payload, đúng yêu cầu contract mới retry |
| 429 | Service quá tải | Retry sau ít nhất 10 giây theo header Retry-After |

---

## 4. Giả định bổ sung

- Giả định 1: Core Business có trách nhiệm sinh `eventId` duy nhất cho mỗi event.
- Giả định 2: Notification service có thể nhận 3 lần retry tại tầng queue trước khi dừng.
- Giả định 3: Nếu `channelDetails` không được cung cấp, Notification service sẽ dùng `channels` list để routing.

---

## 5. Câu hỏi cho Provider

1. Notification service có thể chấp nhận event với `channels` rỗng và tự chọn default như thế nào?
2. `correlationId` có được phép trùng giữa các event khác nhau trong cùng một business transaction không?
3. Có nên support `sms` mặc định nếu severity = CRITICAL, hoặc chỉ dùng khi consumer explicit gửi `sms`?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Tên field không thống nhất | Consumer gửi sai, provider reject payload | Dùng contract chung, fix trong `openapi.yaml` |
| Severity enum khác nhau | Gửi notification không đúng mức | Chốt enum `LOW/MEDIUM/HIGH/CRITICAL` |
| Không xử lý duplicate event | Gửi thông báo lặp | Dùng `eventId` để idempotency |
| Response error không chuẩn | Consumer không parse được lỗi | Dùng `Problem Details` chung |
| Queue retry vượt quá | Gửi nhiều notification lặp | Giới hạn retry và trả 409 nếu duplicate |
