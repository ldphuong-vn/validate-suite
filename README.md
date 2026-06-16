# validate-suite

> A Claude Code skill that validates business ideas / products / features / pivots / point decisions
> (pricing, channel, kill/sunset) through a **gated, evidence-based** pipeline, then renders an **HTML
> report** (a Validation Dossier). Instead of ad-hoc analysis, every conclusion is scored, critiqued,
> and traceable to the *type* of evidence behind it.

**🌐 Language:** [English](#english) · [Tiếng Việt](#tiếng-việt)

---

## English

### What it is

`validate-suite` runs an idea/decision through up to **9 staged gates** (framing → hard gate →
problem → solution → market → business model → decision → GTM → lifecycle). Every stage reads and
writes a single **Validation Dossier** (JSON), and the run ends with an **HTML report**. The template
self-verifies its own decision math and auto-glosses jargon for non-expert readers.

The core idea is **anti-hallucination discipline**: every score is tagged with an evidence type
(measured / cited / user-stated / assumed / unknown), and weak evidence *automatically* lowers
confidence. A confident "GO" can never rest on guesses.

### Install

Drop the `validate-suite/` folder into one of:

- **Shared across all projects:** `~/.claude/skills/validate-suite/`
- **One project only:** `<project>/.claude/skills/validate-suite/`

> Cloning this repo? The repo root *is* the skill (entry: `SKILL.md`). Put the cloned folder where
> Claude Code looks for skills, e.g. `~/.claude/skills/validate-suite/`.

Open Claude Code — the skill is auto-detected. No database, no service, nothing else to install.

### Use

Just say it naturally; the skill activates itself:

> "validate idea X" · "should we build/ship/kill feature Y" · "stress-test this pricing strategy"
> · "is this worth pursuing" · "check GTM readiness"

It classifies the case → prints the route of stages it will run → asks for confirmation → runs each
gate (one console line each) → emits a **verdict (Go / No-go / Pivot / Park) + confidence + coaching**
→ writes an **HTML report** (plus a `*.dossier.json` data file).

### Two modes (`meta.mode`)

| mode | for | memory |
|---|---|---|
| `STATELESS` (default when run as a chat skill) | one-shot, nothing saved | none |
| `STATEFUL` | needs a storage backend (PostgreSQL + outcome tracking) | yes: calibration, nudges, founder profile |

`STATEFUL` enables a "mentor layer" (see `mentor-layer.md`) and only works when wired to a storage
backend. Running as a plain chat skill: keep it `STATELESS`.

### Structure

```
SKILL.md             # orchestrator: classify + route the stages
modes.md             # STATELESS vs STATEFUL
guardrails.md        # evidence discipline & no-fabrication rules (read this)
lens-registry.md     # intent layer: tag → lens (only runs relevant lenses)
dossier-schema.md    # the Dossier data structure (single source of truth)
glossary.md          # plain-language glossary of every term
mentor-layer.md      # memory / calibration layer (STATEFUL only)
stages/<0..8>/       # 9 stages: framing → hardgate → ... → decision → gtm → lifecycle
lenses/              # critique lenses (six-hats, premortem, pricing, digital-ads, profit-first...)
render/template.html # HTML report template (self-verifies math, auto-glosses terms)
```

### Core principles

- **Anti-hallucination:** each score carries an evidence type; weak evidence drags confidence down
  automatically; no plausible-sounding figures.
- **Gated:** weak ideas die early (cheap) before any costly analysis.
- **Plain-language output:** the report translates labels and shows hover tooltips for terms — readable
  by non-experts.

### License

MIT — see [LICENSE](LICENSE).

---

## Tiếng Việt

### Là gì

`validate-suite` là bộ skill (Claude Code) thẩm định ý tưởng / sản phẩm / tính năng / pivot / quyết
định kinh doanh (giá, kênh, kill/sunset) theo một quy trình **có cổng — dựa bằng chứng** rồi xuất ra
**báo cáo HTML** (Validation Dossier). Thay vì phân tích ngẫu hứng, mọi kết luận đều có điểm số, có
phản biện, và truy được dựa trên *loại* bằng chứng nào.

Cốt lõi là **kỷ luật chống ảo giác**: mỗi điểm số gắn loại bằng chứng (đo được / có dẫn nguồn / người
dùng nêu / giả định / chưa rõ); bằng chứng yếu **tự động** kéo độ tin cậy xuống. Một kết luận "Tiến
hành" tự tin không bao giờ được dựa trên phỏng đoán.

### Cài đặt

Thả thư mục `validate-suite/` vào một trong hai nơi:

- **Dùng chung mọi dự án:** `~/.claude/skills/validate-suite/`
- **Chỉ trong 1 dự án:** `<project>/.claude/skills/validate-suite/`

> Clone repo này? Gốc repo *chính là* skill (entry: `SKILL.md`). Đặt thư mục clone vào nơi Claude Code
> tìm skill, vd `~/.claude/skills/validate-suite/`.

Mở Claude Code, skill tự được nhận diện. Không cần database, không service, không cài thêm gì.

### Dùng

Nói tự nhiên, skill tự kích hoạt:

> "validate ý tưởng X" · "có nên build/ship/kill tính năng Y không" · "thẩm định chiến lược giá Z"
> · "stress-test ý tưởng này" · "kiểm tra mức sẵn sàng cho GTM"

Skill sẽ: phân loại tình huống → in lộ trình các bước sẽ chạy → hỏi xác nhận → chạy từng cổng (1
dòng/bước) → ra **kết luận (Tiến hành / Dừng / Xoay hướng / Tạm gác) + độ tin cậy + lời cố vấn** →
xuất một file **HTML báo cáo** (kèm `*.dossier.json` là dữ liệu nguồn).

### Hai chế độ (`meta.mode`)

| mode | dùng cho | trí nhớ |
|---|---|---|
| `STATELESS` (mặc định khi chạy dạng skill chat) | dùng một lần, không lưu | không |
| `STATEFUL` | cần một backend lưu trữ (PostgreSQL + theo dõi outcome) | có: hiệu chỉnh, nhắc, hồ sơ người dùng |

`STATEFUL` bật "lớp mentor" (xem `mentor-layer.md`) — chỉ hoạt động khi đã nối hạ tầng lưu trữ. Chạy
dạng skill chat: để `STATELESS`.

### Cấu trúc

```
SKILL.md             # orchestrator: phân loại + điều phối các tầng
modes.md             # STATELESS vs STATEFUL
guardrails.md        # kỷ luật bằng chứng & chống "bịa số" (đọc kỹ)
lens-registry.md     # lớp intent: tag → lens (chỉ gọi lens liên quan)
dossier-schema.md    # cấu trúc dữ liệu Dossier (single source of truth)
glossary.md          # từ điển thuật ngữ (đời thường)
mentor-layer.md      # lớp trí nhớ/hiệu chỉnh (chỉ STATEFUL)
stages/<0..8>/       # 9 tầng: framing → hardgate → ... → decision → gtm → lifecycle
lenses/              # các lăng kính phản biện (six-hats, premortem, pricing, digital-ads, profit-first...)
render/template.html # template báo cáo HTML (tự verify số, tự chú giải thuật ngữ)
```

### Nguyên tắc cốt lõi

- **Chống ảo giác:** mỗi điểm số gắn loại bằng chứng; bằng chứng yếu tự động kéo độ tin cậy xuống;
  cấm điền số "nghe hợp lý".
- **Có cổng:** ý tưởng dở bị chặn sớm (rẻ) trước khi phân tích tốn kém.
- **Đời thường hoá:** báo cáo dịch nhãn sang tiếng Việt + chú giải rê-chuột cho thuật ngữ — đọc được
  cho cả người không chuyên.

### Giấy phép

MIT — xem [LICENSE](LICENSE).
