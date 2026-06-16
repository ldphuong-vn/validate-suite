# validate-suite

> Bộ skill (Claude Code) thẩm định ý tưởng / sản phẩm / tính năng / pivot / quyết định kinh doanh
> (giá, kênh, kill/sunset) theo một quy trình **có cổng — dựa bằng chứng** rồi xuất ra **báo cáo HTML**
> (Validation Dossier). Mục tiêu: thay vì phân tích ngẫu hứng, mọi kết luận đều có điểm số, có phản
> biện, và truy được dựa trên loại bằng chứng nào.

## Cài đặt

Thả nguyên thư mục `validate-suite/` vào một trong hai nơi:

- **Dùng chung mọi dự án:** `~/.claude/skills/validate-suite/`
- **Chỉ trong 1 dự án:** `<project>/.claude/skills/validate-suite/`

Mở Claude Code, skill tự được nhận diện (entry: `SKILL.md` ở gốc). Không cần cài thêm gì —
không database, không service.

## Dùng

Nói tự nhiên trong Claude Code, skill tự kích hoạt:

> "validate ý tưởng X" · "có nên build/ship/kill tính năng Y không" · "thẩm định chiến lược giá Z"
> · "stress-test ý tưởng này" · "kiểm tra mức sẵn sàng cho GTM"

Skill sẽ: phân loại tình huống → in lộ trình các bước sẽ chạy → hỏi xác nhận → chạy từng cổng
(in 1 dòng/bước) → ra **kết luận (Tiến hành / Dừng / Xoay hướng / Tạm gác) + độ tin cậy + lời cố vấn**
→ xuất một file **HTML báo cáo** (kèm `*.dossier.json` là dữ liệu nguồn).

## Hai chế độ (`meta.mode`)

| mode | dùng cho | trí nhớ |
|---|---|---|
| `STATELESS` (mặc định khi chạy trong chat) | dùng một lần, không lưu | không |
| `STATEFUL` | cần một backend lưu trữ (PostgreSQL + theo dõi outcome) | có: hiệu chỉnh, nhắc, hồ sơ người dùng |

`STATEFUL` bật "lớp mentor" (xem `mentor-layer.md`) — chỉ hoạt động khi đã nối với hạ tầng lưu trữ.
Chạy dạng skill chat: để `STATELESS`.

## Cấu trúc

```
SKILL.md            # orchestrator: phân loại + điều phối các tầng
modes.md            # STATELESS vs STATEFUL
guardrails.md       # kỷ luật bằng chứng & chống "bịa số" (đọc kỹ)
lens-registry.md    # lớp intent: tag → lens (chỉ gọi lens liên quan)
dossier-schema.md   # cấu trúc dữ liệu Dossier (single source of truth)
glossary.md         # từ điển thuật ngữ (đời thường)
mentor-layer.md     # lớp trí nhớ/hiệu chỉnh (chỉ STATEFUL)
stages/<0..8>/      # 9 tầng: framing → hardgate → ... → decision-gate → gtm-plan → lifecycle
lenses/             # các lăng kính phản biện (six-hats, premortem, pricing, digital-ads, profit-first...)
render/template.html# template báo cáo HTML (tự verify lại số, tự chú giải thuật ngữ)
```

## Nguyên tắc cốt lõi

- **Chống ảo giác:** mỗi điểm số gắn loại bằng chứng (đo được / có dẫn nguồn / người dùng nêu / giả
  định / chưa rõ); bằng chứng yếu **tự động** kéo độ tin cậy xuống; cấm điền số "nghe hợp lý".
- **Có cổng:** ý tưởng dở bị chặn sớm (rẻ) trước khi phân tích tốn kém.
- **Đời thường hoá:** báo cáo dịch nhãn sang tiếng Việt + chú giải rê-chuột cho thuật ngữ — đọc được
  cho cả người không chuyên.
