# Biên bản đàm phán hợp đồng API

- Cặp đàm phán:
- Product: 6A / 7A
- Provider:Notification
- Consumer:Core
- Phiên: v1.0
- Ngày:19/05/2026

---

## Issue #1

- Raised by: Consumer
- Endpoint: `Event alert.created`
- Concern: Event chưa thống nhất field `severity`, có thể gây Notification xử lý sai mức cảnh báo.
- Proposal: Bổ sung field `severity` bắt buộc với enum `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.
- Resolution: Accepted
- Rationale: Notification cần severity để quyết định mức ưu tiên và channel gửi phù hợp.
- Impact: Consumer phải luôn gửi severity trong payload event.

---

## Issue #2

- Raised by: Provider
- Endpoint: `Event alert.created`, `alert.escalated`, `alert.resolved`
- Concern: Retry từ queue có thể gây duplicate event và gửi notification lặp.
- Proposal: Mỗi event phải có `eventId` duy nhất để Notification kiểm tra idempotency.
- Resolution: Accepted
- Rationale: Queue async sử dụng cơ chế at-least-once delivery nên duplicate có thể xảy ra.
- Impact: Consumer cần sinh `eventId` duy nhất cho từng event.

---

## Issue #3

- Raised by: Provider
- Endpoint: `Event alert.created`, `alert.escalated`, `alert.resolved`
- Concern: Consumer có thể gửi `channels` rỗng, provider không biết phải route theo kênh nào.
- Proposal: Nếu `channels` rỗng thì Notification service sẽ dùng cấu hình mặc định; nếu muốn chi tiết thì Consumer dùng `channelDetails`.
- Resolution: Accepted
- Rationale: Giúp consumer giữ payload tối giản khi không cần tùy chỉnh từng channel.
- Impact: Consumer có thể gửi `channels` rỗng hoặc `channelDetails` với cấu hình chi tiết.

---

## Issue #4

- Raised by: Consumer
- Endpoint: `/notifications/{notificationId}`
- Concern: Không rõ provider có hỗ trợ truy vấn trạng thái notification hay chỉ trả 202 và không có trace.
- Proposal: Thêm endpoint GET `/notifications/{notificationId}` để kiểm tra trạng thái gửi notification.
- Resolution: Accepted
- Rationale: Core Business cần monitoring và audit khi notification không thành công.
- Impact: Provider bổ sung resource `NotificationDelivery`, consumer có thể kiểm tra bằng `notificationId`.

---

## Issue #5

- Raised by: Provider
- Endpoint: `Event alert.created`, `alert.escalated`, `alert.resolved`
- Concern: Service cần báo lỗi chuẩn để consumer dễ xử lý mà không phải parse message text.
- Proposal: Sử dụng `application/problem+json` với schema `Problem` cho error response.
- Resolution: Accepted
- Rationale: Giữ nhất quán và thuận tiện cho xử lý lỗi toàn bộ hệ thống.
- Impact: Consumer sẽ xử lý các lỗi 400/401/403/404/409/422/429/500 theo RFC 7807.

---

## Issue #6

- Raised by: Consumer
- Endpoint: `Event alert.created`, `alert.escalated`, `alert.resolved`
- Concern: Không rõ retry policy nên consumer không biết khi nào nên gửi lại event.
- Proposal: Định nghĩa retry tối đa 3 lần cho lỗi 5xx/timeout và nếu event duplicate thì trả 409.
- Resolution: Accepted
- Rationale: Queue async sử dụng at-least-once delivery; cần tránh duplicate notification.
- Impact: Consumer chỉ retry tối đa 3 lần theo rule và dùng `eventId` khác cho event mới.

---

# Chốt hợp đồng v1.0

Provider sign-off:  ____________________  
Consumer sign-off:  ____________________  
Witness (GV/TA):    ____________________  
Date: 19/05/2026

Provider sign-off:  
Consumer sign-off:  
Witness (GV/TA):    
Date:               

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
|  |  |  |
