# Hai chế độ vận hành — STATEFUL vs STATELESS

> Cùng một lõi (logic stages + Dossier schema + lenses + render), hai lớp vỏ. Khác biệt duy nhất là
> một **lớp trí nhớ & hiệu chỉnh** phủ lên trên, bật/tắt bằng cờ `meta.mode`. Không nhân đôi lõi.

```
meta.mode : STATEFUL | STATELESS
```

## STATEFUL — bản có backend + private use (hướng "super mentor")
Có trí nhớ. Mỗi Dossier được lưu vào sổ quyết định; outcome được theo dõi; ngưỡng được hiệu chỉnh
theo kết quả thật; bộ học pattern của chính người dùng và chủ động nhắc. Đây là track biến validator
thành mentor — xem `mentor-layer.md`. Chạy trên hạ tầng backend (PostgreSQL + lớp trí nhớ nhiều tầng + scheduled jobs).

Bật:
- Đọc `founder_profile` để cá nhân hóa critique ("anh hay overestimate cầu — giả định này cần soi kỹ").
- Dùng ngưỡng **đã hiệu chỉnh** thay vì mặc định.
- Ghi Dossier vào ledger; lên lịch theo dõi outcome; canh reversal_conditions.
- Nhắc chủ động (scheduled job): tripwire đã chạm, Dossier quá hạn rà outcome, lặp lại điểm mù cũ.

## STATELESS — bản external user, dùng một lần
Không trí nhớ. Chạy đúng lõi pipeline, sinh một Dossier + HTML, **không giữ lại gì** sau phiên. Không
profile, không calibration cá nhân, không nhắc chủ động, không theo dõi outcome. Một ảnh chụp sạch.

Tắt toàn bộ lớp mentor. Cụ thể:
- KHÔNG đọc/ghi memory, KHÔNG ledger, KHÔNG outcome tracking, KHÔNG founder profile.
- Dùng **ngưỡng mặc định tĩnh** (TYPE_1=0.75, TYPE_2=0.65) — không cá nhân hóa.
- Privacy-first: nêu rõ trong báo cáo rằng đây là phiên dùng một lần, dữ liệu không được lưu.

### Giới hạn phải nói thẳng (transparency)
Báo cáo STATELESS phải ghi rõ giới hạn của chính nó, đúng tinh thần guardrail: *một lần chấm không thể
hiệu chỉnh theo bạn hay nhắc bạn theo thời gian; ngưỡng là mặc định quần thể, không cá nhân hóa; chất
lượng quyết định bền vững đến từ việc theo dõi kết quả qua nhiều lần.* Nói trung thực, không bán hàng.

## Bất biến chung
- Lõi (stages/gates/lenses/render/schema) GIỐNG HỆT giữa hai mode — chỉ lớp phủ khác.
- `meta.mode` luôn được ghi và kiểm khi đọc.
- Guardrails (`guardrails.md`) áp cho cả hai. STATELESS không được "bịa" calibration mà nó không có.
