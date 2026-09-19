# Design — Redesign UX báo cáo HTML (3 tầng)

- **Ngày:** 2026-09-19
- **Trạng thái:** đã duyệt phạm vi A trong phiên (nghiên cứu bằng skill ui-ux-pro-max)
- **Stack trên:** `main` (`3b1dfe8`)
- **Phạm vi:** THUẦN RENDERER — chỉ `render/template.html`. Không đổi Dossier/schema/stage docs.

## Căn cứ nghiên cứu

Skill ui-ux-pro-max: khuyến nghị style **Data-Dense Dashboard** (palette xanh chuyên nghiệp + xám —
khớp token indigo hiện có), UX rules áp dụng: `progressive-disclosure`, `heading-hierarchy` (h1→h2→h3
liên tục, cho screen reader), `line-length-control`, `legend-visible`, `color-not-only`,
`adaptive-navigation` (≥1024px sidebar), `smooth-scroll`, `visual-hierarchy` qua size/spacing chứ
không chỉ màu. Anti-pattern tránh: ornament rườm rà, "ornate design".

## Tầng 1 — Kiến trúc 4 chương + điều hướng

Chương = câu hỏi người đọc tự hỏi; coaching kéo từ cuối lên đầu (người không chuyên cần nhất):

| # | Chương | Câu hỏi mở đầu | Sections (số mới) |
|---|---|---|---|
| 1 | Kết luận | Vậy nên làm gì? | 1.1 Câu hỏi & bối cảnh · 1.2 Tóm tắt điều hành · 1.3 Lời cố vấn (từ cuối lên) |
| 2 | Vì sao | Căn cứ đâu? | 2.1 Bằng chứng theo tầng (+ legend màu) · 2.2 Đóng khung vấn đề & giả định |
| 3 | Rủi ro & phản biện | Có gì đáng sợ? | 3.1 Critique · 3.2 Sáu mũ · 3.3 SWOT · 3.4 Porter · 3.5 Party · 3.6 Matrix · 3.7 Profit First |
| 4 | Làm gì tiếp | Kế hoạch ra sao? | 4.1 Kế hoạch & vòng đời (sơ đồ mới) · 4.2 Nhật ký quyết định |

- Chapter rỗng (mọi section con rỗng) → ẩn cả chương.
- **Mục lục sticky** cột phải ≥1024px (`adaptive-navigation`); mobile → hàng pills đầu trang.
  TOC dựng động sau render (duyệt `.chapter` + `.sec-head`), highlight chương đang đọc bằng
  IntersectionObserver; `scroll-behavior:smooth`.
- Heading hierarchy: h1 tiêu đề → h2 chương → h3 section → h4 sub-block trong section.
- Skip-link "Bỏ tới nội dung".

## Tầng 2 — Lớp đọc cho người không chuyên

- **Dòng "Nghĩa là" trong band:** `verdict-sub` hiện đang leak mã (`GO`) — thay bằng map hành động
  đời thường: GO → "Cứ tiến hành — canh chừng rủi ro ở chương 3"; NO_GO → "Dừng — đọc lý do và bài
  học"; PIVOT → "Xoay hướng — giữ bằng chứng đã thu được"; PARK → "Tạm gác — còn thiếu đầu vào,
  xem phần cố vấn".
- **Thang confidence đời thường** dưới gauge: dưới ngưỡng → "Chưa đủ tự tin để tiến hành sạch —
  verdict bị hạ thành xem lại"; đủ → "Đủ tự tin so với ngưỡng cần đạt".
- **Legend 5 màu bằng chứng** đầu chương 2: mỗi màu + nghĩa đời thường 1 câu (fix `color-not-only`).
- **Progressive disclosure:** persona Party Mode — card nổi lean + concern, lập luận đầy đủ thu vào
  `<details>` gập mặc định (clashes + synthesis vẫn nổi). Gate evidence đã gập sẵn từ trước.

## Tầng 3 — Kế hoạch & vòng đời thành sơ đồ (section 4.1)

Chia 3 sub-block, giữ nguyên **tất cả chuỗi nhãn cũ** (battery kiểm chứng không vỡ):

1. **"Ra mắt như thế nào"** — Định vị · Nhóm khách đầu cầu · Thông điệp chính · Kênh (giữ .kv rows)
   + **Bước launch → stepper trình tự**: các bước đánh số nối mũi tên `→` (CSS flex + arrow span,
   wrap dọc trên mobile vẫn đọc đúng trình tự).
2. **"Đo cái gì"** — **North Star card nổi bật** (viền trái rail, chữ lớn) · **Khung đo AARRR → phễu
   5 bước nối mũi tên** (không còn chips rời) · Chỉ số phễu chi tiết (chips) · Giữ khách theo cohort
   · OKR (giữ nhãn "Mục tiêu: ...").
3. **"Rồi sao nữa"** — 2 nhánh song song: **Đạt → Mở rộng** (viền xanh) / **Không đạt → Thu hẹp**
   (viền đỏ), mỗi nhánh list tiêu chí + **vòng re-validation**: icon SVG mũi tên tròn + "Xem lại
   định kỳ: {trigger} → chạy lại tầng {n}".

## Không đổi

- Dữ liệu/schema/Dossier, bảng màu (đã tune), self-verify badges, glossify, hay giá trị nhãn
  `.kv .k` hiện có (battery phụ thuộc).
- renderPlan vẫn render khi có dữ liệu bất kể verdict (quy tắc plan-mode đợt trước).

## Kiểm chứng

1. Battery cũ nguyên vẹn: ctx block, conf verify (xanh+đỏ mutation), asm 3 dòng, so-chains, gate IDs
   G1.1–G1.6, gtm labels đầy đủ ("Thông điệp chính","Bước launch","Khung đo AARRR","Mục tiêu",
   "Chỉ số phễu cần đo","Giữ khách theo cohort"), mode/eyebrow mutations.
2. Mới: 4 chương đúng thứ tự; TOC có đủ link anchor + active highlight; `verdict-sub` không còn mã
   thô; legend 5 mục; stepper 4 bước; phễu 5 bước nối mũi tên; 2 nhánh + vòng lặp; coaching nằm
   chương 1; console 0 lỗi; không tràn ngang.
3. Screenshot before/after full page.
