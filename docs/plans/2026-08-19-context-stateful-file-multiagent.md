# Context + STATEFUL-file + Multi-agent — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Thực thi spec `docs/specs/2026-08-19-context-stateful-file-multiagent-design.md` — block "Câu hỏi & bối cảnh" + dòng chế độ trong báo cáo, STATEFUL-file backend (store + heartbeat + docs contract), skill native đa agent.

**Architecture:** Repo là skill markdown + template HTML. Phase 1 đổi template (TDD bằng browser) + trung lập hoá từ ngữ + README. Phase 2 là docs contract: file mới `store.md`, cập nhật modes/USAGE/mentor-layer/SKILL.md. Kiểm thử: Playwright MCP qua HTTP server local (`python -m http.server 8613`) + grep.

**Tech Stack:** Markdown, vanilla JS trong `render/template.html`. Không dependency mới.

## Global Constraints (từ spec)

- Không đổi logic chấm điểm (ngưỡng, trọng số, công thức, routing).
- Frontmatter SKILL.md chỉ giữ `name` + `description` (chuẩn mở).
- Nội dung không gọi tên tool riêng của một agent.
- Store tại `~/.validate-suite/` — ngoài thư mục mọi agent.
- Thay đổi schema additive, optional; `schema_version` giữ `"1.0.0"`.
- Mọi edit trong repo `D:\ZCode\.zcode\workspace\default\validate-suite`, branch `stateful-file-portable`, commit sau mỗi task.

---

### Task 1: Block "Câu hỏi & bối cảnh" + dòng chế độ (TDD)

**Files:**
- Modify: `render/template.html` (fixture meta, CSS, consts, `renderContext`, mảng `render()`, eyebrow trong `renderHeader`)

**Interfaces:**
- Consumes: `intake.objective`, `intake.case_type`, `meta.mode` (optional).
- Produces: section đầu tiên "Câu hỏi & bối cảnh"; `CASE_LABEL`, `MODE_NOTE`, `renderContext(d)`.

- [ ] **Step 1: Fixture trước** — thêm `"mode":"STATELESS"` vào `meta` của sample:

```json
  "meta":{"schema_version":"1.0.0","dossier_id":"d-9f2a","revision":0,
    "mode":"STATELESS",
    "title":"Template Designer cho Product X",
```

- [ ] **Step 2: Red check** — khởi HTTP server (`python -m http.server 8613`, background), mở `http://localhost:8613/render/template.html`, evaluate:

```js
() => ({ ctx: document.body.textContent.indexOf("Câu hỏi & bối cảnh")>=0,
         mode: document.body.textContent.indexOf("Chế độ một lần")>=0 })
```
Expected: `{ctx:false, mode:false}`.

- [ ] **Step 3: CSS** — chèn sau `.asm-note{...}`:

```css
  /* question & context block (container tái dùng .problem) */
  .qs .q{font-family:var(--disp);font-size:19px;font-weight:600;line-height:1.35}
  .qs-meta{display:flex;flex-wrap:wrap;gap:8px;align-items:center;margin-top:14px}
  .mode-chip{font-family:var(--mono);font-size:11px;padding:3px 10px;border-radius:999px;white-space:nowrap}
  .mode-STATELESS{background:#EDEFF6;color:var(--park)}
  .mode-STATEFUL{background:#E6F4EC;color:var(--go)}
  .mode-note{font-size:13px;color:var(--ink-soft)}
```

- [ ] **Step 4: Consts** — chèn sau `AARRR_LABEL`:

```js
const CASE_LABEL={A_IDEA:"Ý tưởng mới",B_PRODUCT:"Sản phẩm đang chạy",C_FEATURE:"Tính năng trên sản phẩm",
  D_PIVOT:"Xoay hướng",E_DECISION:"Quyết định điểm"};
const MODE_NOTE={STATELESS:"Chế độ một lần — dữ liệu không lưu; ngưỡng mặc định quần thể, không cá nhân hóa",
  STATEFUL:"Chế độ có trí nhớ — Dossier đã ghi vào sổ quyết định ~/.validate-suite/; ngưỡng được hiệu chỉnh theo kết quả thật"};
```

- [ ] **Step 5: `renderContext`** — chèn trước `function renderExec(d){`:

```js
function renderContext(d){
  const o=d.intake?.objective; const mode=d.meta?.mode;
  const caseLab=CASE_LABEL[d.intake?.case_type];
  if(!o&&!mode&&!caseLab) return "";
  const q=o?`<div class="problem qs"><div class="k">Câu hỏi được thẩm định</div><div class="q">${esc(o)}</div></div>`:"";
  const chips=[];
  if(caseLab) chips.push(`<span class="chip">${esc(caseLab)}</span>`);
  if(mode) chips.push(`<span class="mode-chip mode-${esc(mode)}">${mode==="STATEFUL"?"Có trí nhớ":"Một lần"}</span>`);
  const note=mode?`<span class="mode-note">${esc(MODE_NOTE[mode]||mode)}</span>`:"";
  const meta=(chips.length||note)?`<div class="qs-meta">${chips.join("")}${note}</div>`:"";
  return sec("","Câu hỏi & bối cảnh",`${q}${meta}`);
}
```

- [ ] **Step 6: Gắn vào render() + eyebrow** — mảng trong `render(d)` thành `[renderContext(d),renderExec(d),...]`. Trong `renderHeader`, dòng eyebrow:

```js
  document.getElementById("eyebrow").textContent=
    "Validation Dossier · "+(CASE_LABEL[d.intake?.case_type]||d.intake?.case_type||"")+(m.revision?(" · rev "+m.revision):"");
```

- [ ] **Step 7: Green + mutation checks** — reload, evaluate:

```js
() => {
  const out={};
  out.ctx=document.body.textContent.indexOf("Câu hỏi & bối cảnh")>=0;
  out.q=document.querySelector(".qs .q")?.textContent;
  out.modeLine=document.body.textContent.indexOf("Chế độ một lần")>=0;
  out.eyebrow=document.getElementById("eyebrow").textContent;
  out.caseChip=[...document.querySelectorAll(".qs-meta .chip")].map(c=>c.textContent).join("|");
  DOSSIER.meta.mode="STATEFUL"; render(DOSSIER);
  out.stateful=document.body.textContent.indexOf("Có trí nhớ")>=0 && document.body.textContent.indexOf("~/.validate-suite/")>=0;
  DOSSIER.meta.mode="STATELESS"; delete DOSSIER.intake.objective; delete DOSSIER.meta.mode; render(DOSSIER);
  out.hiddenWhenEmpty=!document.body.textContent.indexOf;
  out.ctxGone=[...document.querySelectorAll("h2")].every(h=>h.textContent.indexOf("Câu hỏi & bối cảnh")<0);
  return out;
}
```
Expected: ctx true, q = câu hỏi sample, modeLine true, eyebrow chứa "Tính năng trên sản phẩm", caseChip "Tính năng trên sản phẩm", stateful true, ctxGone true. (Lưu ý dòng `out.hiddenWhenEmpty` chỉ là no-op giữ chỗ xóa — bỏ qua giá trị của nó.) Reload trả fixture. Console 0 lỗi mới.

- [ ] **Step 8: Commit**

```bash
git add render/template.html
git commit -m "feat(template): block Câu hỏi & bối cảnh + dòng chế độ theo meta.mode"
```

---

### Task 2: Trung lập hoá từ ngữ + README đa agent

**Files:**
- Modify: `SKILL.md` (dòng ~45, ~99)
- Modify: `lenses/party-mode.md` (dòng ~12 — đọc file trước để lấy nguyên văn)
- Modify: `guardrails.md` (mục 3)
- Modify: `README.md` (tiêu đề + Install EN + Cài đặt VN + ghi chú store)

- [ ] **Step 1: SKILL.md** — hai thay thế:

`Only ask the user (via\n`ask_user_input`) if genuinely ambiguous` → `Only ask the user directly if genuinely ambiguous` (giữ nguyên phần câu sau).

`**MUST_RUN** → \`view\` the stage's SKILL.md` → `**MUST_RUN** → \`read\` the stage's SKILL.md`.

- [ ] **Step 2: party-mode.md** — đọc dòng 12, thay `một Claude sinh ra nhiều giọng phân tích` → `một agent AI sinh ra nhiều giọng phân tích`.

- [ ] **Step 3: guardrails.md mục 3** — `(web_search / dữ liệu nội bộ)` → `(công cụ tìm kiếm web của agent đang chạy / dữ liệu nội bộ)`.

- [ ] **Step 4: README EN** — dòng 3: `> A Claude Code skill that validates` → `> An agent-native skill (Claude Code, Codex, Gemini CLI, Cursor, Copilot, ...) that validates`. Thay cả block `### Install` (nguyên văn dòng 25–35) bằng:

```markdown
### Install (works with many AI agents)

The skill follows the open **Agent Skills** standard (`SKILL.md` + folder), supported by Claude Code,
Codex CLI, Gemini CLI, Cursor, Copilot, Windsurf and others. Drop the `validate-suite/` folder into
your agent's skills directory:

| Agent | Skills directory |
|---|---|
| Claude Code | `~/.claude/skills/validate-suite/` (project: `.claude/skills/`) |
| Gemini CLI | `~/.gemini/skills/validate-suite/` |
| Codex CLI | `~/.codex/skills/validate-suite/` |
| Cursor | `~/.cursor/skills/validate-suite/` — or read directly from `~/.claude/skills/` / `.codex/skills/` |

> **One copy, many agents:** keep a single master copy and symlink/junction it into each agent's
> skills directory instead of duplicating.

> Cloning this repo? The repo root *is* the skill (entry: `SKILL.md`).

Open your agent — the skill is auto-detected. No database, no service, nothing else to install.
```

- [ ] **Step 5: README VN** — dòng 91: `là bộ skill (Claude Code) thẩm định` → `là bộ skill chuẩn mở (Agent Skills — chạy trên Claude Code, Codex, Gemini CLI, Cursor, Copilot...) thẩm định`. Thay block `### Cài đặt` (dòng 100–110) bằng bản dịch tương ứng của block EN ở Step 4 (giữ bố cục y hệt).

- [ ] **Step 6: README — ghi chú store (cả EN & VN, ngay sau bảng modes)** — thêm:

```markdown
> STATEFUL keeps its memory in `~/.validate-suite/` — outside every agent's folder, so switching
> agents (Claude today, Codex tomorrow) keeps the same decision ledger and founder profile.
```
bản VN: `> STATEFUL giữ trí nhớ tại ~/.validate-suite/ — ngoài thư mục của mọi agent, nên đổi agent (hôm nay Claude, mai Codex) vẫn dùng chung một sổ quyết định và hồ sơ người dùng.`

- [ ] **Step 7: Greps + commit**

```bash
grep -rn "ask_user_input\|\`view\`" SKILL.md        # 0 kết quả
grep -c "Gemini CLI" README.md                       # ≥2 (EN+VN)
git add -A && git commit -m "docs: trung lập hoá từ ngữ + README cài đặt đa agent"
```

---

### Task 3: `store.md` + schema `meta.mode`

**Files:**
- Create: `store.md`
- Modify: `dossier-schema.md` (bảng §4.1 thêm dòng `mode`; comment dòng ~81 sửa khớp plan-mode)

- [ ] **Step 1: Tạo `store.md`** với nội dung: giới thiệu (store = hợp đồng backend-agnostic, STATEFUL := có store ghi được); layout 5 file đúng spec 2.1; bảng interface 8 hàm (`save_dossier`, `record_outcome`, `list_due_outcomes`, `arm_tripwires`, `eval_tripwires`, `load_profile`, `save_profile`, `recompute_calibration`) — mỗi hàm một dòng mô tả вход→ra; cách triển khai file-backend từng hàm; ví dụ jsonc cho `outcomes.jsonl`, `tripwires.json`, `founder_profile.json`, `calibration_report.json` (lấy cấu trúc y từ mentor-layer.md mục 2–4); mục "PostgreSQL sau này" (cùng interface, bảng theo schema mục 6, file shapes mirror để import một script); mục "Giới hạn trung thực" (nudge chỉ khi có phiên; calibration ≥15–20 outcome/dải; dòng hỏng JSONL bị bỏ qua lúc đọc).

- [ ] **Step 2: schema §4.1** — thêm dòng sau `one_liner`:

```markdown
| `mode` | string | `STATEFUL` \| `STATELESS` (xem `modes.md`, `store.md`). Optional — thiếu thì report không hiện dòng chế độ |
```

- [ ] **Step 3: schema dòng ~81** — `"gtm_plan":  { /* chỉ khi verdict = GO */ },` → `"gtm_plan":  { /* sau GO, hoặc plan-mode cho sản phẩm đang sống */ },`

- [ ] **Step 4: Grep + commit**

```bash
grep -c "save_dossier\|record_outcome\|list_due_outcomes\|arm_tripwires\|eval_tripwires\|load_profile\|save_profile\|recompute_calibration" store.md   # ≥8
grep -n "\`mode\`" dossier-schema.md    # có dòng mới
git add store.md dossier-schema.md && git commit -m "docs: store.md — hợp đồng store backend-agnostic + schema meta.mode"
```

---

### Task 4: modes.md + USAGE.md + mentor-layer.md — hợp đồng mới

**Files:**
- Modify: `modes.md`, `USAGE.md`, `mentor-layer.md`

- [ ] **Step 1: modes.md** — (a) thay câu `Chạy trên hạ tầng backend (PostgreSQL + lớp trí nhớ nhiều tầng + scheduled jobs).` bằng `Chạy trên một **store thật sự ghi được** — mặc định là file store tại ~/.validate-suite/ (xem store.md), không cần hạ tầng; mini-app sau này đổi sang PostgreSQL cùng interface đó.` (b) trong "Bất biến chung" thêm bullet: `- File store là store hợp lệ: câu "đã lưu" phải đi kèm đường dẫn file thật; không có store (kể cả file) → STATELESS.` (c) thêm đoạn ngắn sau phần STATEFUL: heartbeat thay scheduled job — nudge outcome quá hạn/tripwire đến hạn TẠI Step 0 của skill, tối đa 1–2 dòng.

- [ ] **Step 2: USAGE.md** — (a) TL;DR dòng 2: `**Bản private (có backend) → STATEFUL (mentor)**, bật từ quyết định đầu tiên.` → `**Bản private → STATEFUL-file (mentor)** — store tại ~/.validate-suite/, không cần backend; bật từ quyết định đầu tiên.` (b) hàng bảng `Instance private của bạn | STATEFUL + buộc vào memory org | Cấu hình instance` → `Chat riêng (mọi agent) | STATEFUL-file mặc định | Skill tự tạo ~/.validate-suite/ lần đầu (hỏi một lần)`. (c) mục "Gọi khi đang chạy dạng skill trong chat": cập nhật 2 dòng đầu cho khớp (mặc định mentor = file store; incognito = nói rõ). (d) "Checklist cho người vận hành (bản stateful)" viết lại: 1) store ở `~/.validate-suite/` đã tạo (skill hỏi lần đầu); 2) heartbeat chạy trong skill — không cần scheduled job; 3) tùy chọn: cấu hình hook/session-start của agent để mở phiên là kiểm store; 4) instance external/khách để STATELESS — không trộn store.

- [ ] **Step 3: mentor-layer.md** — (a) dòng 5 `Ánh xạ thẳng lên lớp trí nhớ nhiều tầng + scheduled jobs của backend.` → `Ánh xạ thẳng lên lớp trí nhớ nhiều tầng; scheduled jobs được thay bằng heartbeat tại Step 0 của skill (xem store.md).` (b) chèn mục mới `## File backend (mặc định — không cần PostgreSQL)` TRƯỚC mục "Ánh xạ lên một backend stateful": layout store, heartbeat thay cron, hai giới hạn trung thực (nudge chỉ khi có phiên; calibration cần ≥15–20 outcome/dải), câu chốt: PostgreSQL vẫn là đích cho mini-app, cùng interface.

- [ ] **Step 4: Greps + commit**

```bash
grep -c "file store\|~/.validate-suite" modes.md USAGE.md mentor-layer.md   # mỗi file ≥1
grep -c "buộc.*STATELESS\|buộc \`STATELESS\`" modes.md                      # 0
git add modes.md USAGE.md mentor-layer.md && git commit -m "docs: STATEFUL = có store ghi được; file backend mặc định + heartbeat"
```

---

### Task 5: SKILL.md — mode logic + heartbeat + lưu sổ

**Files:**
- Modify: `SKILL.md` (Step 0 block callout, thêm heartbeat; Step 5 thêm luồng lưu)

- [ ] **Step 1: Thay callout STATELESS cứng** (block `> **Chạy dạng skill trong chat:** ...`) bằng:

```markdown
> **Chạy dạng skill trong chat:** store mặc định là **file store** tại `~/.validate-suite/`
> (xem `store.md`) — không cần PostgreSQL. Store chưa tồn tại → hỏi người dùng MỘT câu có muốn
> bật trí nhớ không: đồng ý → tạo thư mục, chạy STATEFUL; từ chối hoặc không ghi được → STATELESS
> (nói rõ giới hạn một lần trong báo cáo). Người dùng nói "chế độ một lần / incognito / đừng lưu"
> → STATELESS cho riêng lần đó. Lý do giữ nguyên tinh thần cũ: chỉ hứa "đã lưu" khi CÓ nơi ghi thật.
```

- [ ] **Step 2: Heartbeat** — thêm ngay sau đoạn mode trong Step 0:

```markdown
**Heartbeat (chỉ STATEFUL — chạy TRƯỚC Intake):** đọc store theo `store.md`. Có Dossier quá hạn
rà outcome (>60 ngày chưa có kết quả) hoặc tripwire đến hạn → nudge người dùng tối đa 1–2 dòng,
hỏi kết quả, ghi `outcomes.jsonl` ngay trong phiên. Không có gì đến hạn → im lặng, vào Intake.
```

- [ ] **Step 3: Lưu sổ ở Step 5** — thêm cuối Step 5 (sau đoạn glossary):

```markdown
**Sau khi render (chỉ STATEFUL):** `save_dossier` vào ledger; chuyển từng
`journal.reversal_conditions` thành tripwire ARMED (`arm_tripwires`); cập nhật `founder_profile`;
nếu phiên này vừa ghi outcome mới → `recompute_calibration`. Luôn nói cho người dùng biết đã lưu
ở đâu (đường dẫn file thật) — không hứa hụt.
```

- [ ] **Step 4: Grep + commit**

```bash
grep -c "Heartbeat\|arm_tripwires\|file store" SKILL.md    # ≥3
git add SKILL.md && git commit -m "feat: SKILL.md — mode STATEFUL-file + heartbeat + lưu sổ sau render"
```

---

### Task 6: Kiểm chứng cuối theo spec

- [ ] **Step 1:** HTTP server + browser: chạy lại toàn bộ check Task 1 (xanh), console 0 lỗi mới.
- [ ] **Step 2:** Greps tổng theo spec mục "Cách kiểm chứng": 5 mục — block/eyebrow/mode (browser), trung lập (`ask_user_input`/`` `view` `` = 0; README ma trận), hợp đồng (store.md 8 hàm; modes.md không "buộc STATELESS"; SKILL.md có heartbeat + save/arm), đối chiếu từng mục spec.
- [ ] **Step 3:** Commit phần sót (nếu có); tắt HTTP server; báo cáo kết quả + đề xuất tích hợp.
