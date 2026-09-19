# SWOT + Porter lenses — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Thực thi spec `docs/specs/2026-08-20-swot-porter-lenses-design.md` — 2 lens mới (SWOT luôn-on ở stage 6; Porter điều kiện tag `competitive_landscape` ở stage 4), render + kiểm chứng, merge về main.

**Architecture:** Mô hình lens hiện có: file markdown + 1 dòng registry (no gate, no confidence impact). Template thêm 2 render block + fixture swot. Kiểm thử: Playwright MCP qua `python -m http.server 8613`.

## Global Constraints

- Lens KHÔNG tạo gate, KHÔNG kéo confidence; schema additive/optional, `schema_version` giữ `"1.0.0"`.
- Không renumber gate; không đổi routing/ngưỡng/công thức.
- Repo `D:\ZCode\.zcode\workspace\default\validate-suite`, branch `add-swot-porter`; KHÔNG commit `.mimosa/`.

---

### Task 1: Lớp markdown (lens + registry + stage docs + schema + SKILL.md)

**Files:**
- Create: `lenses/swot.md`, `lenses/porter.md`
- Modify: `lens-registry.md`, `stages/decision-gate/SKILL.md`, `stages/market-gtm/SKILL.md`, `dossier-schema.md`, `SKILL.md`

- [ ] **Step 1:** Tạo `lenses/swot.md` — `<!-- applies_when: decision_gate -->`: SWOT = tách nội lực (S/W, điều ta kiểm soát) khỏi bên ngoài (O/T, thị trường quyết định); ô O là mảnh mới hoàn toàn (mũ vàng chỉ nói lợi ích phương án, không quét cơ hội thị trường); quy tắc: 2–4 gạch đầu dòng/ô, S/W rút từ stages 0–5, O/T theo guardrails §8 (≥2 nguồn) hoặc gắn evidence_type yếu; không copy nguyên văn mũ đen/vàng; SWOT nuôi matrix + rationale, không tự set verdict.

- [ ] **Step 2:** Tạo `lenses/porter.md` — `<!-- applies_when: competitive_landscape -->`: 5 lực `rivalry/new_entrants/substitutes/supplier_power/buyer_power` mỗi lực một đoạn ngắn + `verdict` một câu (ngành thuận gió? lực đáng sợ nhất? né được không?); tuyên bố đối thủ/thị phần CITED ≥2 nguồn độc lập (guardrails §8), 1 nguồn → cap như ASSUMED; đặc biệt với Type-1; thiếu dữ liệu → UNKNOWN, cấm điền "vừa phải" cho đủ.

- [ ] **Step 3:** `lens-registry.md` — (a) vốn tag thêm dòng: `competitive_landscape : quyết định phụ thuộc cấu trúc cạnh tranh (vào thị trường mới, beachhead, định vị chạm đối thủ)`; (b) bảng + JSON: hàng `decision_gate` thêm `"swot"` vào cuối list; thêm hàng `competitive_landscape → porter`.

- [ ] **Step 4:** `stages/decision-gate/SKILL.md` — chèn Step 2c sau Step 2b (trước "### Step 3 — Critique lenses"):

```markdown
### Step 2c — SWOT (bản đồ nội lực vs bên ngoài)
Fill `lenses.swot` (`attached_to_stage: 6`) — see `../../lenses/swot.md`. Two to four short bullets
per quadrant. S/W synthesize what stages 0–5 already recorded (internal — what we control); O/T scan
outside (market, competitors, substitutes, regulation) with evidence discipline — external claims
follow the two-source rule (`guardrails.md` §8) or carry weak-evidence tags. The O quadrant is the
new ground here: yellow hat covers benefits of proceeding, not market opportunities. SWOT feeds
Step 4's matrix and Step 7's rationale — it does not set the verdict itself.
```
Và Writes list thêm `lenses.swot` (cùng dòng với six_hats...decision_matrix).

- [ ] **Step 5:** `stages/market-gtm/SKILL.md` — thêm bước 6 sau bước 5:

```markdown
6. **Five Forces** (chỉ khi tag `competitive_landscape` bật — xem `../../lens-registry.md`; tag tắt
   → không gọi, không ghi gì): assess cấu trúc ngành theo `../../lenses/porter.md`, ghi
   `lenses.porter` (`attached_to_stage: 4`). Hợp với quyết định vào thị trường mới / chọn beachhead /
   định vị chạm trực tiếp đối thủ.
```
Writes thành: ``stages[]` record (`stage_id: 4`, gates G4.1 + G4.2, status, score), appends to `lenses.red_team` (+ `lenses.porter` nếu tag `competitive_landscape`).``

- [ ] **Step 6:** `dossier-schema.md` §4.5 — chèn vào JSONC lenses: block `swot` (sau `six_hats`) và `porter` (sau `second_order`):

```jsonc
  "swot": {                                // stage 6 — tổng hợp nội lực/bên ngoài (luôn chạy cùng bộ critique)
    "attached_to_stage": 6,
    "strengths":    [ "Nội lực đang mạnh — rút từ stages 0–5" ],
    "weaknesses":   [ "Nội lực yếu" ],
    "opportunities":[ "Cơ hội bên ngoài: thị trường, kênh, xu hướng" ],
    "threats":      [ "Đe dọa bên ngoài: đối thủ, thay thế, quy định" ]
  },
```
```jsonc
  "porter": {                              // chỉ điền khi tag competitive_landscape bật (cấu trúc ngành — Five Forces)
    "attached_to_stage": 4,
    "rivalry": "Đối thủ hiện tại: ai, mạnh/yếu gì",
    "new_entrants": "Cửa ngõ gia nhập + rào cản",
    "substitutes": "Sản phẩm/cách làm thay thế",
    "supplier_power": "Nhà cung cấp nắm gì (API nền tảng, data, hạ tầng)",
    "buyer_power": "Khách nắm thế nào (chi phí chuyển đổi, mặc cả)",
    "verdict": "Một câu: ngành thuận gió không + lực đáng sợ nhất"
  },
```

- [ ] **Step 7:** `SKILL.md` Step 1 — trong câu liệt kê tags thêm `competitive_landscape` vào list (trước `(decision_gate is always on...)`).

- [ ] **Step 8:** Greps + commit:

```bash
grep -c "swot" lens-registry.md stages/decision-gate/SKILL.md dossier-schema.md   # ≥1 mỗi file
grep -c "porter\|competitive_landscape" lens-registry.md stages/market-gtm/SKILL.md SKILL.md  # ≥1 mỗi file
git add lenses/swot.md lenses/porter.md lens-registry.md stages/ SKILL.md dossier-schema.md
git commit -m "feat: lens SWOT (stage 6) + Porter Five Forces (stage 4, tag competitive_landscape)"
```

---

### Task 2: Template — fixture swot → red → render → green

**Files:**
- Modify: `render/template.html`

**Interfaces:**
- Consumes: `lenses.swot {strengths,weaknesses,opportunities,threats}`, `lenses.porter {rivalry,new_entrants,substitutes,supplier_power,buyer_power,verdict}`.
- Produces: `renderSwot(d)` (lưới 2×2), `renderPorter(d)` (5 thẻ + verdict), const `FORCE_LABEL`.

- [ ] **Step 1: Fixture** — chèn `swot` vào `lenses` của sample, ngay sau block `six_hats` (sau dòng `"blue":"Kết luận: GO có điều kiện — kèm phỏng vấn cầu song song"},`):

```json
    "swot":{"attached_to_stage":6,
      "strengths":["~80% hạ tầng Product X tái dùng được","Đã có tập khách active + kênh in-app sẵn"],
      "weaknesses":["Chưa đo được mức cầu ZNS thật","Team chưa từng làm UI designer"],
      "opportunities":["Zalo đang đẩy ZNS cho SMB — xu hướng kênh","Mở đường ZNS Trigger + Mini App sau này"],
      "threats":["Agency làm template ZNS giá rẻ","Zalo đổi chính sách/pricing ZNS"]},
```

- [ ] **Step 2: Red check** — server + navigate + evaluate: `document.body.textContent.indexOf("SWOT")>=0` → mong đợi `false`.

- [ ] **Step 3: CSS** — chèn trước `/* decision matrix */`:

```css
  /* SWOT 2x2 */
  .swot-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .swot{border:1px solid var(--line);border-radius:10px;padding:16px;background:#fff;border-top:3px solid var(--ink)}
  .swot .hl{font-family:var(--mono);font-size:12px;letter-spacing:.08em;text-transform:uppercase;margin-bottom:8px;font-weight:600}
  .swot ul{margin:0;padding-left:18px}
  .swot li{margin-bottom:5px;font-size:14px;color:var(--ink-soft)}
  .swot-S{border-top-color:var(--go)} .swot-S .hl{color:var(--go)}
  .swot-W{border-top-color:var(--nogo)} .swot-W .hl{color:var(--nogo)}
  .swot-O{border-top-color:var(--rail)} .swot-O .hl{color:var(--rail)}
  .swot-T{border-top-color:var(--pivot)} .swot-T .hl{color:var(--pivot)}

  /* Porter five forces */
  .forces{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .force{border:1px solid var(--line);border-radius:10px;padding:15px 16px;background:#fff}
  .force .fh{margin-bottom:6px}
  .force .fn{font-family:var(--mono);font-size:12px;letter-spacing:.06em;text-transform:uppercase;color:var(--rail);font-weight:600}
  .force p{margin:0;font-size:14px;color:var(--ink-soft)}
```
Và trong `@media (max-width:720px)` thêm: `.swot-grid{grid-template-columns:1fr} .forces{grid-template-columns:1fr}`.

- [ ] **Step 4: Consts** — sau block `MODE_NOTE={...};`:

```js
const FORCE_LABEL={rivalry:"Đối thủ hiện tại",new_entrants:"Đối thủ mới tiêm nhập",
  substitutes:"Sản phẩm thay thế",supplier_power:"Quyền lực nhà cung cấp",buyer_power:"Quyền lực khách hàng"};
```

- [ ] **Step 5: Render functions** — chèn trước `function renderParty(d){`:

```js
function renderSwot(d){
  const s=d.lenses?.swot; if(!s) return "";
  const cells=[["S","swot-S","Điểm mạnh · nội lực","strengths"],["W","swot-W","Điểm yếu · nội lực","weaknesses"],
    ["O","swot-O","Cơ hội · bên ngoài","opportunities"],["T","swot-T","Đe dọa · bên ngoài","threats"]];
  const q=cells.map(([tag,cls,lab,key])=>{
    const items=(s[key]||[]).map(x=>`<li>${esc(x)}</li>`).join("")||"<li>—</li>";
    return `<div class="swot ${cls}"><div class="hl">${tag} · ${lab}</div><ul>${items}</ul></div>`;
  }).join("");
  return sec("","SWOT — nội lực & bên ngoài",`<div class="swot-grid">${q}</div>`);
}

function renderPorter(d){
  const p=d.lenses?.porter;
  const keys=p?Object.keys(FORCE_LABEL).filter(k=>p[k]):[];
  if(!keys.length) return "";
  const cards=keys.map(k=>`<div class="force"><div class="fh"><span class="fn">${FORCE_LABEL[k]}</span></div>
    <p>${esc(p[k])}</p></div>`).join("");
  const v=p.verdict?`<div class="pf-verdict">${esc(p.verdict)}</div>`:"";
  return sec("","Cấu trúc ngành (Porter — năm lực lượng)",`<div class="forces">${cards}</div>${v}`);
}
```

- [ ] **Step 6: Render array** — `[renderContext(d),renderExec(d),renderFraming(d),renderPipeline(d), renderCritique(d),renderHats(d),renderParty(d),...]` → chèn `renderSwot(d),renderPorter(d)` ngay sau `renderHats(d),`.

- [ ] **Step 7: Green + battery + porter mutation** — navigate lại, evaluate:

```js
() => {
  const out={};
  out.swotCells=document.querySelectorAll(".swot").length;                    // 4
  out.swotHeadings=[...document.querySelectorAll(".swot .hl")].map(h=>h.textContent[0]).join(""); // "SWOT"
  out.battery={
    ctx:document.body.textContent.indexOf("Câu hỏi & bối cảnh")>=0,
    confGreen:(document.getElementById("confverify")?.textContent||"").indexOf("khớp")>=0,
    asmRows:document.querySelectorAll("table.asm tbody tr").length,
    soChains:document.querySelectorAll(".crit-list li b").length,
    hardIds:[...document.querySelectorAll(".gate .gid")].slice(0,6).map(g=>g.textContent).join(",")
  };
  DOSSIER.lenses.porter={attached_to_stage:4,
    rivalry:"Agency template ZNS nhiều, giá cạnh tranh",new_entrants:"Rào cản thấp — dev cá nhân làm được",
    substitutes:"Khách tự gửi qua Zalo OA console",supplier_power:"Zalo nắm toàn bộ chính sách + pricing",
    buyer_power:"Khách SMB dễ chuyển sang agency",verdict:"Ngành ngược gió ở supplier power — Zalo là điểm rủi ro đơn lớn nhất"};
  render(DOSSIER);
  out.porterCards=document.querySelectorAll(".force").length;                 // 5
  out.porterVerdict=document.body.textContent.indexOf("Ngành ngược gió")>=0;
  delete DOSSIER.lenses.porter; render(DOSSIER);
  out.porterGone=document.querySelectorAll(".force").length;                  // 0
  return out;
}
```
Expected: swotCells 4, swotHeadings "SWOT", battery nguyên xanh, porterCards 5, porterVerdict true, porterGone 0. Reload trả fixture. Console 0 lỗi mới.

- [ ] **Step 8: Commit**

```bash
git add render/template.html && git commit -m "feat(template): render SWOT 2x2 + Porter Five Forces, fixture swot"
```

---

### Task 3: Chốt — push + merge main

- [ ] **Step 1:** Push branch `add-swot-porter` lên origin.
- [ ] **Step 2:** `git checkout main && git merge add-swot-porter && git push origin main`.
- [ ] **Step 3:** Xoá branch local + remote; báo cáo kết quả.
