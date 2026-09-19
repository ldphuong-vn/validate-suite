# Report UX Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Thực thi spec `docs/specs/2026-09-19-report-ux-redesign-design.md` — 4 chương + TOC + lớp đọc đơn giản + sơ đồ Kế hoạch. Thuần renderer, 1 file `render/template.html`.

**Global Constraints:** Không đổi Dossier/schema/nhãn `.kv .k` hiện có; battery cũ phải xanh nguyên vẹn; template vẫn self-contained (CSS thuần + SVG inline, không dependency); branch `report-ux-redesign` từ `main`.

---

### Task 1: Hạ tầng chương + điều hướng

**Files:** Modify `render/template.html`

- [ ] CSS mới (chèn trước `footer{`): `.layout` grid `1fr 230px` ≥1024px; `.toc` sticky + link active; `.ch-head/.ch-num/.ch-q`; `section.sub` + `.sec-head h3` (17px); `.ch-q` câu hỏi mở đầu; `.skip-link`; mobile: layout 1 cột, toc ẩn, `.ch-pills` hiện.
- [ ] `sec()` → output `section.sub` với `id="sN"` (biến đếm `SEC_N` reset mỗi render) + **h3** thay h2; 14 render function đổi số sec: 1.1 ctx · 1.2 exec · 1.3 coaching · 2.1 pipeline · 2.2 framing · 3.1 critique(h3 trong .critique) · 3.2 hats · 3.3 swot · 3.4 porter · 3.5 party · 3.6 matrix · 3.7 profit · 4.1 plan · 4.2 journal.
- [ ] `render(d)`: dựng 4 chuỗi con → `chapter(num,q,inner,id)` bọc `<section class="chapter" id="chN">`; chương rỗng → ẩn; sau `innerHTML` gọi `buildNav()` (quét `.chapter`/`.sec-head` → TOC + pills) + IntersectionObserver highlight; body thêm skip-link; `html{scroll-behavior:smooth}`.
- [ ] `renderCoaching` gọi trước `renderPlan` trong render() (đổi chỗ trong mảng — hàm giữ nguyên).

### Task 2: Lớp đọc đơn giản

- [ ] `const MEANS_LABEL={GO:"Cứ tiến hành — canh chừng rủi ro ở chương 3",NO_GO:"Dừng — đọc lý do và bài học bên dưới",PIVOT:"Xoay hướng — giữ bằng chứng đã thu được",PARK:"Tạm gác — còn thiếu đầu vào, xem phần cố vấn"};` — `verdict-sub` nhận `MEANS_LABEL[v]||v` (hết leak mã thô).
- [ ] Gauge note: sau confverify thêm `.gauge-note`: dưới ngưỡng → "Chưa đủ tự tin để tiến hành sạch — verdict bị hạ thành xem lại"; đủ → "Đủ tự tin so với ngưỡng cần đạt".
- [ ] Legend bằng chứng đầu `renderPipeline`: hàng `.ev-legend` 5 mục — `Đo được = số liệu thật · Có dẫn nguồn = trích nguồn ngoài · Người dùng nêu = nói lời anh · Giả định = chưa kiểm chứng · Chưa rõ = không có cơ sở` (dùng lại `.ev` badges + text).
- [ ] `renderParty`: mỗi card → persona + role + lean nổi; `.parg`/`.pcon` bọc `<details class="pdetails"><summary>Lập luận của persona này</summary>...`. Clashes + synthesis nguyên trạng.

### Task 3: Sơ đồ Kế hoạch & vòng đời (renderPlan mới)

- [ ] CSS: `.pv-sub` (h4 sub-block label); `.stepper` flex wrap + `.step-arrow` mũi tên; `.ns-card` North Star nổi bật; `.funnel` + `.funnel-step` + connector (AARRR); `.branches` 2 cột (`.branch-ok` xanh / `.branch-stop` đỏ) + `@media` 1 cột; `.loop` icon SVG tròn + text.
- [ ] `renderPlan(d)` viết lại — 3 sub-block, GIỮ NHÃN CŨ:
  1. **Ra mắt như thế nào**: .kv rows Định vị/Nhóm khách đầu cầu/Thông điệp chính/Kênh + hàng `.k`"Bước launch" chứa `.stepper` (item nối `<span class="step-arrow">→</span>`).
  2. **Đo cái gì**: `.ns-card` (label "Chỉ số quan trọng nhất (North Star)") + hàng `.k`"Khung đo AARRR — 5 bước khách đi qua" chứa `.funnel` (5 step: `<b>nhãn</b> + desc`, nối arrow) + "Chỉ số phễu cần đo" chips + "Giữ khách theo cohort" + OKR rows "Mục tiêu: ...".
  3. **Rồi sao nữa**: `.branches` — "Nếu Đạt → Mở rộng" (expand_criteria) / "Nếu Không đạt → Thu hẹp" (sunset_criteria) + `.loop`: SVG refresh + "Xem lại định kỳ: {trigger} → chạy lại tầng {list}".

### Task 4: Kiểm chứng + chốt

- [ ] Battery cũ (spec mục Kiểm chứng 1) + mới (mục 2) qua Playwright; console 0 lỗi; không tràn ngang.
- [ ] Screenshot before/after full page lưu workspace.
- [ ] Commit theo task; push branch; merge `main`; push; xoá branch.
