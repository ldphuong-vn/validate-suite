# Design — Gắn lens SWOT + Porter's Five Forces

- **Ngày:** 2026-08-20
- **Trạng thái:** đã duyệt trong phiên (sau rà soát "bộ đã có gì / thiếu gì")
- **Stack trên:** `main` (đã chứa 18 commit các đợt trước)
- **Phạm vi:** 2 lens mới (additive) — file lens, registry/tag, stage docs, schema §4.5, template render + fixture

## Vì sao là 2 lens này

Rà soát cho thấy bộ **đã có** decision gates đầy đủ (hardgate G1.1–G1.6 + decision-gate + gate giữa các
tầng) và **rủi ro** phủ 4 lớp (assumptions risk×uncertainty, premortem, inversion, red-team), nhưng
**thiếu SWOT** (không có ma trận 4 ô nội-lực/bên-ngoài — Sáu mũ không thay thế được, nhất là ô "Cơ hội
bên ngoài" mà bộ đang không có chỗ nào quét) và **thiếu Porter's Five Forces** (tầng 4 có TAM/SAM/SOM
+ red-team nhưng không có khung cấu trúc ngành).

## Thiết kế — tuân thủ mô hình lens hiện có

**Nguyên tắc registry:** "Thêm lens = thêm một dòng, không đụng logic". Lens mới KHÔNG tạo gate,
KHÔNG kéo confidence — chỉ thêm lớp phân tích (đúng quy tắc lens-registry).

### SWOT — stage 6, luôn chạy cùng bộ critique
- File: `lenses/swot.md`, `applies_when: decision_gate` → thêm `swot` vào hàng `decision_gate` của registry.
- Lý do luôn-on: rẻ (tổng hợp từ bằng chứng Dossier đã có, 2–4 gạch đầu dòng/ô), và lấp gap thật
  (O = quét cơ hội bên ngoài — không lens nào làm).
- Dữ liệu: `lenses.swot = {attached_to_stage: 6, strengths[], weaknesses[], opportunities[], threats[]}`.
  S/W rút từ stages 0–5 (nội lực); O/T quét bên ngoài — tuyên bố đối thủ/thị trường theo guardrails §8
  (≥2 nguồn độc lập) hoặc gắn evidence_type yếu.
- Render: lưới 2×2, đặt ngay sau Sáu mũ.

### Porter's Five Forces — stage 4, điều kiện theo tag mới
- Tag mới `competitive_landscape` (thêm vào vốn tag của registry + danh sách tag ở SKILL.md Step 1):
  bật khi quyết định phụ thuộc cấu trúc cạnh tranh (vào thị trường mới, chọn beachhead, định vị chạm
  đối thủ). Intake suy ra rẻ, như các tag khác.
- File: `lenses/porter.md`, `applies_when: competitive_landscape` → hàng mới trong registry.
- Dữ liệu: `lenses.porter = {attached_to_stage: 4, rivalry, new_entrants, substitutes,
  supplier_power, buyer_power, verdict}` — mỗi lực một đoạn ngắn neo bằng chứng; `verdict` một câu.
  Tuyên bố về đối thủ phải CITED ≥2 nguồn (guardrails §8); thiếu dữ liệu → UNKNOWN, không điền "vừa phải".
- Stage doc: tầng 4 thêm bước 6 (chỉ chạy khi tag bật); Writes ghi rõ `+ lenses.porter (nếu tag)`.
- Render: 5 thẻ lực + dòng verdict; block chỉ hiện khi có dữ liệu (fixture hiện tại stage 4 SKIP
  nên KHÔNG có porter — kiểm chứng render bằng mutation test, giữ sample coherent).

## Schema §4.5 (additive, optional, schema_version giữ 1.0.0)

Thêm 2 block ví dụ vào JSONC lenses: `swot` (sau `six_hats`) và `porter` (sau `second_order`).

## Fixture

- Thêm `lenses.swot` vào sample (coherent — stage 6 đã chạy).
- KHÔNG thêm porter vào fixture (stage 4 SKIP — thêm vào sẽ mâu thuẫn nội dung sample).

## Kiểm chứng

1. Red: fixture có swot nhưng template chưa render → block vắng; sau implement → 4 ô đầy đủ.
2. Porter: mutation test (`DOSSIER.lenses.porter = {...}; render(DOSSIER)`) → 5 thẻ + verdict hiện;
   xoá dữ liệu → block biến mất.
3. Battery chức năng nguyên vẹn: context block, 2 badge self-verify, bảng giả định, second-order,
   gate IDs, gtm đủ trường, mutation mode/objective — tất cả xanh; console 0 lỗi mới.
4. Greps: registry có `swot` + `porter` + `competitive_landscape`; stage 4/6 docs wire đúng;
   schema có 2 block; SKILL.md tag list có tag mới.
