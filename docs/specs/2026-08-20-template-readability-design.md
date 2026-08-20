# Design — Template readability (Phương án A: tune tokens + cỡ chữ + khoảng cách)

- **Ngày:** 2026-08-20
- **Trạng thái:** đã duyệt hướng A trong phiên
- **Stack trên:** `stateful-file-portable`
- **Phạm vi:** CHỈ CSS trong `render/template.html`. Không đụng DOM, JS, dữ liệu, schema.

## Mục tiêu

Khắc phục 3 gốc rễ khó đọc (thep ảnh anh chụp + phân tích): (1) nền cỡ chữ quá sâu (11–13px
cho nội dung thật), (2) màu nhạt (`--rail-soft`, `--ink-soft` kết hợp cỡ nhỏ), (3) lệch bậc thị
giác (H1 42px vs kicker 11px) + khối `.kv` thiếu phân tách dòng.

## Bảng thay đổi (rule-by-rule, old → new)

### Tokens
| Rule | Old | New |
|---|---|---|
| `:root` `--ink-soft` | `#4A4F5A` | `#3C414B` |
| `:root` `--rail-soft` | `#5C6B96` | `#46567F` |

### Cỡ chữ — nâng sàn
| Rule | Old | New |
|---|---|---|
| `.title-block h1` | `clamp(28px,4vw,42px)` | `clamp(26px,3.5vw,36px)` |
| `.qs .q` | 19px | 20px |
| `.w-cell .k`, `.problem .k`, `.kv .row .k`, `.jrow .jk`, `.coach-grid .ck` | 11px | 12px + `font-weight:600` |
| `.maker`, `.gauge-meta`, `footer` | 11px | 12px |
| `.journal .meta` | 12px | 13px |
| `.pill` | 10.5px | 11.5px |
| `.ev` | 9.5px | 10.5px |
| `.pcard .prole`, `.plean` | 10px | 11px |
| `.move .mt` | 10.5px | 11px |
| `.gate .gev` | 13px | 14px |
| `.chip` | 13px | 14px |
| `.mode-note` | 13px | 14px |
| `.asm-note` | 12.5px | 13.5px |
| `.pcard .pcon` | 12.5px | 13.5px |

### Nội dung chính — thêm nửa bậc
| Rule | Old | New |
|---|---|---|
| `.crit-list li` | 14.5px | 15px |
| `.pm .cz` | 13.5px `#A9AEBA` | 14px `#B8BDC9` |
| `.pm .mt` | 13.5px | 14px |
| `.pcard .parg` | 13.5px | 14px |
| `.clash` | 13.5px | 14px |
| `.crit-list` (color) | `#C3C7D0` | `#D3D7DF` |

### Component polish
| Rule | Thay đổi |
|---|---|
| `.chip` | padding `4px 10px` → `5px 12px`; thêm `background:#F6F7FA` |
| `.kv` | `gap:12px` → `gap:20px`; thêm `.kv>.row:not(:last-child){padding-bottom:18px;border-bottom:1px dashed var(--line)}` |

## Không đổi

- DOM, JS, dữ liệu sample, schema — mọi kiểm chứng chức năng cũ phải xanh nguyên vẹn.
- `.verdict-word` giữ cỡ lớn (điểm nhấn); `.eyebrow`, `.critique .sub`, `.pscore` giữ 12–13px
  (đã đủ, thuộc meta).

## Kiểm chứng

1. Ảnh BEFORE/AFTER cùng nguồn dữ liệu — so sánh trực quan.
2. Chạy lại trọn bộ check chức năng: block Câu hỏi & bối cảnh, 2 badge self-verify, bảng giả định,
   second-order, gtm đầy đủ, mutation STATEFUL/xoá objective, console 0 lỗi mới.
3. Tương phản: mọi cặp màu chữ/nền chính ≥ 4.5:1 (hai token mới ≥ 7:1).
