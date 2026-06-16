# USAGE — cách dùng bộ & set cờ `mode`

> Đọc nhanh cho người dùng và người vận hành. Chi tiết: `modes.md`, `mentor-layer.md`,
> `dossier-schema.md`. File này chỉ trả lời: *gọi thế nào* và *bật/tắt mentor ra sao*.

## TL;DR
- **External / dùng một lần → STATELESS**, khóa cứng khi deploy. Người dùng không thấy, không đổi được.
- **Bản private (có backend) → STATEFUL (mentor)**, bật từ quyết định đầu tiên.
- Chỉ ép STATELESS lẻ khi muốn một quyết định "incognito" (không vào hồ sơ).
- **Không có default toàn cục — bề mặt quyết định default.**

## Vì sao không có một default chung
STATEFUL cần một `user_id` ổn định để gắn memory/founder_profile. Người external ẩn danh không có danh
tính đó — mentor mode chẳng có gì để bám, mà lưu ý tưởng của người lạ lại là rủi ro quyền riêng tư.
Ngược lại, để bản private chạy STATELESS thì vứt sạch giá trị (không học, không calibrate, không
accountability). Nên mỗi bề mặt mang default đúng của nó.

## Set cờ ở tầng triển khai (chính) — set MỘT LẦN, không hỏi mỗi quyết định

| Bề mặt | mode | Set ở đâu |
|---|---|---|
| Mini-app public (external) | `STATELESS` hard-code | Cấu hình deploy; ẩn khỏi người dùng |
| Instance private của bạn | `STATEFUL` + buộc vào memory org | Cấu hình instance |
| Khách tư vấn (kiểu NSTech) | `STATELESS` mặc định | Chỉ dựng instance STATEFUL riêng nếu khách ĐỒNG Ý |

Trong code (mini-app/backend): set `meta.mode` khi khởi tạo Dossier, không để route người dùng cuối chạm.

## Gọi khi đang chạy dạng skill trong chat

- **Mặc định mentor (bản private của anh):** không cần nói gì.
  > "validate ý tưởng X"  → STATEFUL
- **Ép một lần không lưu (incognito decision):** nói rõ.
  > "validate X, chế độ một lần, đừng lưu vào hồ sơ"  → STATELESS cho riêng lần đó
- **Khi mơ hồ:** orchestrator hỏi đúng một câu (mentor hay một-lần?) rồi nhớ cho cả phiên.

## Khi nào thật sự bật/tắt giữa chừng
Hầu như không. Lần duy nhất hợp lý: trong bản STATEFUL, anh muốn MỘT quyết định nhạy cảm không vào
founder_profile → ép lần đó thành STATELESS. Đây là "incognito decision", không phải đổi default.

## Kỳ vọng đúng về STATEFUL ngày đầu
Ngày đầu bật mentor, nó hành xử gần như STATELESS — `founder_profile` rỗng, chưa có calibration. Khác
biệt là nó BẮT ĐẦU GHI. Mentor mode không cho giá trị tức thì; nó gieo hạt. Phải qua ~chục quyết định
CÓ OUTCOME thì calibration + profile mới đủ dữ liệu để thật sự "biết anh". → Với bản private, **bật
STATEFUL từ quyết định đầu tiên**: càng sớm gieo, càng sớm thu.

## Checklist cho người vận hành (bản stateful)
1. Đặt `mode = STATEFUL` ở cấu hình instance; gắn `user_id`/org_id vào memory.
2. Bật scheduled job: outcome polling + tripwire eval + nudges (xem `mentor-layer.md` mục 2,5,6).
3. Nối `confidence < threshold` và tripwire `TRIPPED` vào cổng phê duyệt con người sẵn có.
4. Đảm bảo instance external/khách tách riêng và để `STATELESS` — không trộn memory giữa các org.
