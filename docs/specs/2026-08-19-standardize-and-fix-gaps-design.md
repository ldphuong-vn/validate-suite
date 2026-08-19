# Design — Chuẩn hoá & vá thiếu validate-suite (Phương án B)

- **Ngày:** 2026-08-19
- **Trạng thái:** hướng đã duyệt trong phiên làm việc; chờ anh duyệt bản spec này
- **Phạm vi:** vá 7 điểm lệch giữa thiết kế ↔ triển khai, cộng 2 cơ chế chống drift tái diễn

## Mục tiêu

Làm repo chạy đúng như thiết kế của chính nó: lens được wire, trường schema được render đủ,
công thức tự verify, glossary về một nguồn duy nhất — **không đổi bản chất logic thẩm định**.

## Bất biến (không được vi phạm khi triển khai)

- Không đổi ngưỡng, trọng số, công thức confidence, bảng routing case→stage.
- Không renumber gate đang tồn tại; chỉ **thêm** ID cho hardgate (hiện chưa có).
- Giữ `meta.schema_version = "1.0.0"` — mọi thay đổi schema là additive (trường optional).
- Giữ tách bạch: stage không render HTML; renderer chỉ đọc Dossier.

## 7 thay đổi

### 1. Wire lens `second-order` (hiện đang mồ côi)
- `stages/decision-gate/SKILL.md`: thêm bước — nếu `intake.tags` chứa `downstream_effects`
  → áp dụng `lenses/second-order.md`, ghi kết quả vào `lenses.second_order`
  (`[{action, then, and_then}]` — đúng schema 4.5 sẵn có).
- Tag tắt → không gọi lens, không tạo gate, không kéo confidence (đúng quy tắc lens-registry).
- Template: thêm block render "Hệ quả dây chuyền (bậc 2/bậc 3)" trong khu phản biện,
  đọc từ `lenses.second_order`.

### 2. Template render đủ schema
- `framing.assumptions`: bảng giả định (ID · nội dung · risk · uncertainty · status) đặt ngay
  sau khối 5W1H; highlight dòng H×H — đó là *riskiest assumption* mà stage 3 dùng.
- `gtm_plan` render **đủ** trường schema 4.7: messaging, launch_steps, north_star_metric,
  aarrr (5 ô), okrs (positioning/beachhead/channels đã có thì giữ).
- `lifecycle` render **đủ** trường schema 4.8: funnel_metrics, cohort_retention,
  expand_criteria, sunset_criteria, revalidation.
- **Bỏ điều kiện `verdict === "GO"`** đang chặn block gtm_plan (template, dòng
  `const g=d.decision?.verdict==="GO"?d.gtm_plan:null;`) → render khi có dữ liệu.
  Lý do: plan-mode cho sản phẩm đang sống = GO sẵn (SKILL.md, Step 1), block phải hiện.
- Đồng bộ heading schema 4.7: "(chỉ khi `verdict=GO`)" → "(sau GO, hoặc plan-mode cho sản
  phẩm đang sống)" để hết mâu thuẫn với SKILL.md.

### 3. Self-verify confidence trong template
- JS tái tính confidence **đúng công thức schema 4.6**: mean `effective_score` trên các stage
  0–5 có status ∈ {PASSED, ASSERTED, REVIEW_NEEDED}; REVIEW_NEEDED ×0.85; làm tròn 3 chữ số.
- So với `decision.confidence` (tolerance ±0.005) → badge "khớp / lệch" cạnh gauge, cùng cơ
  chế với badge weighted_totals hiện có.
- Gauge dùng `decision.confidence_threshold` nếu có; fallback theo map TYPE_1→0.75,
  TYPE_2→0.65 (thay vì hardcode 0.65 như hiện tại).

### 4. Gate ID chính thức cho hardgate
- `stages/hardgate/SKILL.md`: gán G1.1–G1.6 cho 6 cổng (THẬT / ĐAU / VỚI-TỚI-ĐƯỢC /
  HỢP PORTFOLIO / KHẢ THI / HỢP PHÁP), khớp thứ tự trình bày của stage.
- Đối chiếu với sample Dossier trong template (đang dùng G1.1, G1.2…) — nếu thứ tự lệch,
  sample là chỗ phải chỉnh theo stage doc, không phải ngược lại.

### 5. Glossary single-source (chống drift)
- `dossier-schema.md` §4.9: thêm trường optional `render.glossary`
  (`{ "CAC": "một câu định nghĩa đời thường", ... }`).
- `SKILL.md` Step 5 (render): đọc `glossary.md`, chuyển thành map thuật ngữ → 1 câu định
  nghĩa, ghi vào `render.glossary` trước khi inject Dossier vào template.
- Template: `GLOSSARY` = merge(bộ fallback hiện tại, `render.glossary`) — dữ liệu từ Dossier
  ưu tiên; fallback chỉ lấp chỗ thiếu để template mở trực tiếp vẫn chạy.

### 6. Nhất quán khai báo gate ở stage docs
- `stages/biz-model/SKILL.md`: khai Writes đủ G5.1–G5.4; trình bày G5.3 trước G5.4 (ID giữ
  nguyên, chỉ đổi chỗ đứng cho đúng số thứ tự).
- `stages/market-gtm/SKILL.md`: mục Writes thêm G4.2 (moat) — gate đã định nghĩa nhưng chưa
  khai ra output.

### 7. Fixture render chuẩn (sample Dossier trong template)
- Nâng cấp sample DOSSIER nhúng trong `render/template.html` thành bộ dữ liệu chuẩn exercise
  **mọi** trường mới render: `framing.assumptions` (có ≥1 dòng H×H), `lenses.second_order`,
  `gtm_plan` đầy đủ, `lifecycle` đầy đủ, `render.glossary` (vài thuật ngữ).
- Sample giữ verdict GO (đường đi đầy đủ nhất để demo).
- Mục đích kép: vừa là demo cho người dùng mới, vừa là kiểm tra render — mở file trực tiếp
  là thấy đủ block + 2 badge self-verify.

## Out of scope (đề phòng phình to)

- Mentor layer / backend STATEFUL (PostgreSQL, calibration, tripwire) — giữ nguyên thiết kế.
- Lens mới, case type mới, ngưỡng/công thức mới.
- Renumber gate, bump schema_version, đổi cấu trúc Dossier ngoài mục 5.

## Cách kiểm chứng khi hoàn tất

1. Mở `render/template.html` trực tiếp trong browser: mọi block mới hiện, 2 badge self-verify
   (weighted_totals + confidence) xanh, console không lỗi.
2. Sửa tay sample verdict thành `PARK` → block gtm_plan vẫn hiện (chứng minh đã bỏ cổng GO).
3. Sửa tay `decision.confidence` lệch giá trị → badge confidence chuyển đỏ "lệch" (chứng minh
   self-check thật sự kiểm tra, không phải trang trí).
4. `grep` xác nhận: không còn `verdict==="GO"` chặn gtm_plan; `stages/hardgate/SKILL.md` chứa
   G1.1–G1.6; stage 4/5 khai Writes đủ gate.
5. Đọc đối chiếu từng mục "7 thay đổi" ở trên.
