# Design — Ngữ cảnh báo cáo + STATEFUL-file backend + đa agent

- **Ngày:** 2026-08-19
- **Trạng thái:** hướng đã duyệt trong phiên; chờ duyệt spec
- **Stack trên:** branch `standardize-fixes` (phần báo cáo xây trên fixture/render lần trước)
- **Phạm vi:** 3 phần — (1) bổ ngữ cảnh cho báo cáo, (2) mentor layer chạy bằng file store,
  (3) skill native với nhiều agent AI, không bó Claude

## Bất biến

- Không đổi logic chấm điểm (ngưỡng, trọng số, công thức confidence, routing).
- Không thêm frontmatter key riêng của vendor vào SKILL.md (chỉ `name` + `description` — chuẩn mở).
- Nội dung stage/lens không được gọi tên tool riêng của một agent; nói tính năng ("đọc file",
  "hỏi người dùng", "tìm kiếm web") thay vì tên tool.
- Store nằm ngoài thư mục của mọi agent (`~/.validate-suite/`) — mentor theo người, không theo agent.
- STATELESS vẫn tồn tại nguyên vẹn (incognito / external); guardrails áp cho cả hai mode.
- Mọi thay đổi schema là additive, optional; `schema_version` giữ `"1.0.0"`.

## Phần 1 — Báo cáo đủ ngữ cảnh (sửa 2 gap bắt được)

### 1.1 Block "Câu hỏi & bối cảnh"
- Vị trí: section ĐẦU TIÊN của `main`, trước "Tóm tắt điều hành".
- Nội dung:
  - **Câu hỏi** = `intake.objective`, trình bày nổi (pattern giống `.problem`: viền trái +
    nền nhạt), nhãn "Câu hỏi được thẩm định".
  - **Nhãn case** tiếng Việt (không mã nội bộ — guardrails §9):
    `A_IDEA:"Ý tưởng mới" · B_PRODUCT:"Sản phẩm đang chạy" · C_FEATURE:"Tính năng trên sản phẩm" ·
    D_PIVOT:"Xoay hướng" · E_DECISION:"Quyết định điểm"`.
  - **Dòng chế độ** từ `meta.mode` (xem 1.2).
- `intake.objective` thiếu → ẩn block (backward-compatible với Dossier cũ).
- Eyebrow ở band: thay mã `case_type` thô bằng nhãn tiếng Việt cùng bảng trên.

### 1.2 Dòng chế độ (`meta.mode`) — đúng yêu cầu minh bạch của modes.md
- `STATELESS` → pill xám + dòng: *"Chế độ một lần — dữ liệu không lưu; ngưỡng mặc định quần thể,
  không cá nhân hóa"*.
- `STATEFUL` → pill xanh rail + dòng: *"Chế độ có trí nhớ — Dossier đã ghi vào sổ quyết định
  `~/.validate-suite/`; ngưỡng được hiệu chỉnh theo kết quả thật"*.
- `meta.mode` thiếu → không render dòng này (Dossier cũ không vỡ).
- Schema §4.1: thêm trường optional `meta.mode` (`STATEFUL | STATELESS`) — SKILL.md vốn đã ghi
  trường này, giờ schema + báo cáo khớp nhau.
- Fixture: thêm `"mode":"STATELESS"` vào sample `meta` (demoDisclaimer đúng tinh thần modes.md);
  test STATEFUL bằng mutation trong lúc kiểm chứng.

## Phần 2 — STATEFUL-file backend (mentor không cần PostgreSQL)

### 2.1 Store layout
```
~/.validate-suite/
  ledger/<dossier_id>.dossier.json   # episodic — mỗi Dossier hoàn tất một file nguyên vẹn
  outcomes.jsonl                     # append-only, mỗi dòng một bản ghi outcome
  tripwires.json                     # [{dossier_id, title, condition, armed_at, due_at, status}]
  founder_profile.json               # derived — cấu trúc theo mentor-layer.md mục 4
  calibration_report.json            # derived — tính lại khi có outcome mới (mentor-layer.md mục 3)
```
- `outcomes.jsonl`: dòng hỏng format → bỏ qua khi đọc (self-healing), không chặn cả store.
- File shapes mirror bảng SQL ở `dossier-schema.md` mục 6 để sau này import PostgreSQL một script.

### 2.2 File mới `store.md` — hợp đồng store (backend-agnostic)
Định nghĩa interface mà mọi backend phải có:
`save_dossier`, `record_outcome`, `list_due_outcomes(now)`, `arm_tripwires(dossier)`,
`eval_tripwires(now)`, `load_profile`, `save_profile`, `recompute_calibration`.
- **File backend (mặc định)**: cách triển khai từng hàm bằng đọc/ghi file thường.
- **PostgreSQL backend (tương lai, mini-app)**: cùng interface, bảng theo schema mục 6.
- Kèm ví dụ jsonc từng loại file (làm cả tài liệu lẫn "fixture" đối chiếu mắt).

### 2.3 Heartbeat — thay scheduled job, portable
- Nằm trong **Step 0 của SKILL.md** (không phụ thuộc hook của agent nào): nếu STATEFUL, đọc store:
  outcome quá hạn (>60 ngày chưa có kết quả) / tripwire đến hạn → nudge người dùng TỐI ĐA 1–2 dòng
  trước khi vào intake. Người dùng trả lời → ghi `outcomes.jsonl` ngay.
- Hook/session-start per-agent: chỉ là enhancement tùy chọn, ghi chú trong README, không phải điều kiện.

### 2.4 Thay đổi hợp đồng trong docs
- `modes.md`: STATEFUL := "**có một store ghi được**" (file backend mặc định | PostgreSQL cho
  mini-app). Bỏ câu "buộc STATELESS khi chạy chat"; giữ tinh thần: không có store (kể cả file) →
  STATELESS + nói rõ trong báo cáo.
- `SKILL.md`: Step 0 — chọn mode mới: private/chat → STATEFUL-file (tạo store nếu chưa có, hỏi
  một lần); người dùng nói "một lần/incognito" hoặc store không ghi được → STATELESS. Thêm heartbeat
  (2.3). Step 5 — nếu STATEFUL: `save_dossier`, `arm_tripwires` từ `reversal_conditions`, cập nhật
  `founder_profile` (nếu có outcome mới thì `recompute_calibration`).
- `USAGE.md`: bảng bề mặt cập nhật — chat private → STATEFUL-file mặc định; incognito là ngoại lệ
  từng lần; mini-app external → STATELESS khóa cứng; mini-app sau này → STATEFUL-PG.
- `mentor-layer.md`: thêm mục "File backend" — ánh xạ scheduled jobs → heartbeat tại Step 0,
  nêu giới hạn trung thực (nudge chỉ khi có phiên làm việc; calibration cần ≥15–20 outcome/dải).

## Phần 3 — Native đa agent

### 3.1 Trung lập hoá từ ngữ (các chỗ bó Claude tìm thấy khi audit)
- `SKILL.md:45`: `(via ask_user_input)` → hỏi người dùng trực tiếp.
- `SKILL.md:99`: `view the stage's SKILL.md` → `read the stage's SKILL.md`.
- `lenses/party-mode.md`: "đây vẫn là một Claude sinh ra nhiều giọng" → "một agent AI sinh ra
  nhiều giọng".
- `guardrails.md` §3: "web_search" → "công cụ tìm kiếm web của agent đang chạy".
- README: tiêu đề/mở đầu "A Claude Code skill" → "An agent-native skill (Claude Code, Codex,
  Gemini CLI, Cursor, Copilot, Windsurf, ...)" — tiếng Việt tương ứng.

### 3.2 README — ma trận cài đặt đa agent (cả mục EN lẫn VN)
| Agent | Thư mục skill |
|---|---|
| Claude Code | `~/.claude/skills/validate-suite/` (project: `.claude/skills/`) |
| Gemini CLI | `~/.gemini/skills/` |
| Codex CLI | `~/.codex/skills/` |
| Cursor | `~/.cursor/skills/` — hoặc đọc thẳng `~/.claude/skills/`, `.codex/skills/` |
- Mẹo "một bản nhiều agent": giữ MỘT bản gốc, symlink/junction vào từng thư mục trên.
- Nêu rõ: store `~/.validate-suite/` dùng chung — đổi agent vẫn cùng một sổ quyết định.

## Out of scope

- Cài đặt PostgreSQL backend thật (chỉ định nghĩa interface trong `store.md`).
- Hook/session-start script cho từng agent (chỉ ghi chú trong README).
- Công cụ tính calibration đứng riêng (tính tại chỗ trong luồng skill).

## Cách kiểm chứng khi hoàn tất

1. Browser mở `render/template.html`: block "Câu hỏi & bối cảnh" hiện câu hỏi + nhãn case + dòng
   chế độ STATELESS; mutation `meta.mode="STATEFUL"` → dòng đổi; mutation xoá `objective` → block ẩn.
2. Eyebrow hiện nhãn tiếng Việt thay mã case.
3. Greps trung lập: không còn `ask_user_input`, `` `view` `` trong SKILL.md; README có ma trận
   đa agent ở cả hai ngôn ngữ.
4. Greps hợp đồng: `store.md` tồn tại với đủ 8 hàm interface; `modes.md` không còn cụm "buộc
   STATELESS"; SKILL.md có heartbeat + save/arm ở đúng Step; 3 file docs (modes/USAGE/mentor-layer)
   cùng nói "STATEFUL = có store ghi được".
5. Đọc đối chiếu từng mục spec.
