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

- Raised by: Consumer / Provider
- Endpoint:
- Concern:
- Proposal:
- Resolution: Accepted / Rejected / Modified
- Rationale:
- Impact:

---

## Issue #4

- Raised by: Consumer / Provider
- Endpoint:
- Concern:
- Proposal:
- Resolution: Accepted / Rejected / Modified
- Rationale:
- Impact:

---

## Issue #5

- Raised by: Consumer / Provider
- Endpoint:
- Concern:
- Proposal:
- Resolution: Accepted / Rejected / Modified
- Rationale:
- Impact:

---

## Issue #6

- Raised by: Consumer / Provider
- Endpoint:
- Concern:
- Proposal:
- Resolution: Accepted / Rejected / Modified
- Rationale:
- Impact:

---

# Chốt hợp đồng v1.0

Provider sign-off:  
Consumer sign-off:  
Witness (GV/TA):    
Date:               

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
|  |  |  |
