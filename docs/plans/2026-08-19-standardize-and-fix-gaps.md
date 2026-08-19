# Standardize & Fix Gaps — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Làm validate-suite chạy đúng như thiết kế của chính nó — theo spec `docs/specs/2026-08-19-standardize-and-fix-gaps-design.md` (phương án B).

**Architecture:** Repo là skill thuần markdown + một template HTML tự chứa. Thay đổi rơi vào 3 nhóm: (1) fixture sample Dossier trong template (làm trước — TDD red baseline), (2) code render JS + CSS trong template, (3) doc markdown (stage docs, schema, orchestrator). Kiểm thử bằng browser thực (Playwright MCP mở `file://` URL, assert DOM) + grep cho phần markdown.

**Tech Stack:** Markdown, vanilla JS/HTML trong `render/template.html` (không build step, không dependency mới). Kiểm thử: Playwright MCP (`browser_navigate`, `browser_evaluate`, `browser_console_messages`), `grep`.

## Global Constraints (từ spec — mọi task ngầm kế thừa)

- Không đổi ngưỡng, trọng số, công thức confidence, bảng routing case→stage.
- Không renumber gate đang tồn tại; chỉ THÊM ID G1.1–G1.6 cho hardgate.
- Giữ `meta.schema_version = "1.0.0"`; mọi thay đổi schema là additive (trường optional).
- Stage không render HTML; renderer chỉ đọc Dossier.
- Văn bản hướng-người-đọc: tiếng Việt đời thường, không nhét mã nội bộ (guardrails.md §9).
- Mọi edit file đều trong repo `D:\ZCode\.zcode\workspace\default\validate-suite`, branch `standardize-fixes`, commit sau mỗi task.

---

### Task 1: Nâng cấp sample Dossier thành fixture chuẩn (red baseline)

**Files:**
- Modify: `render/template.html` (chỉ khối `const DOSSIER = {...}`, dòng ~285–351)

**Interfaces:**
- Produces: fixture dữ liệu cho Task 2–6: `framing.assumptions` có 1 dòng H×H; `lenses.second_order`; `gtm_plan.messaging/launch_steps/aarrr/okrs`; `lifecycle.funnel_metrics/cohort_retention`; `render.glossary`; `decision.confidence` sửa thành 0.752 (sample cũ ghi 0.72 — sai công thức schema 4.6; mean(0.8, 1.0, 0.7, 0.6×0.85, 0.75) = 0.752).

- [ ] **Step 1: Sửa `"assumptions"` trong sample** — thêm dòng A3 H×H (giữ nguyên A1, A2):

```json
    "assumptions":[
      {"id":"A1","text":"Khách hiện tại thực sự cần gửi ZNS, không chỉ chat","risk":"H","uncertainty":"M","status":"PARTIAL"},
      {"id":"A2","text":"Hạ tầng tái dùng được ~80% như ước tính","risk":"H","uncertainty":"L","status":"VALIDATED"},
      {"id":"A3","text":"Khách sẵn lòng trả thêm cho add-on ZNS (chưa đo mức giá chấp nhận)","risk":"H","uncertainty":"H","status":"UNTESTED"}]},
```

- [ ] **Step 2: Thêm `"second_order"` vào `lenses`** — chèn sau dòng `"red_team":[...]`, trước `"decision_matrix"`:

```json
    "second_order":[
      {"action":"Build ZNS Designer vào Product X","then":"Khách chuyển chăm sóc khách sang ZNS tự phục vụ","and_then":"Ticket support giảm nhưng kỳ vọng phản hồi nhanh dâng lên ở mọi kênh"},
      {"action":"Định giá add-on tách khỏi gói chính","then":"Khách gói thấp muốn mua lẻ ZNS","and_then":"Áp lực mở pricing theo module — phải tính lại hàng rào giữa các gói"}],
```

- [ ] **Step 3: Bổ sung `gtm_plan`** — các khóa mới thêm vào đối tượng hiện có (giữ positioning/beachhead/channels/north_star_metric nguyên bản):

```json
    "messaging":["Tạo ZNS có thương hiệu ngay trong Product X — không cần nhờ agency","Dùng dữ liệu khách sẵn có để gửi đúng người, đúng lúc"],
    "launch_steps":["Phỏng vấn 5 khách có nhu cầu ZNS (kiểm giả định A1, A3)","Pre-sell add-on cho 10 khách hiện tại","Beta 2 tuần với 5 khách","Công bố toàn bộ"],
    "aarrr":{"acquisition":"Upsell in-app tới khách đang active","activation":"Tạo template ZNS đầu tiên trong 7 ngày đầu","retention":"Gửi ZNS đều đặn ≥2 lần/tháng","referral":"Khách giới thiệu bạn cùng ngành","revenue":"Tỷ lệ khách active trả tiền add-on"},
    "okrs":[{"objective":"Chứng minh cầu ZNS trả tiền trong 1 quý","key_results":["≥10 khách trả tiền add-on","≥500 ZNS gửi thành công mỗi tháng","Điểm hài lòng add-on ≥ 40/100"]}]
```

- [ ] **Step 4: Bổ sung `lifecycle`** — thêm 2 khóa vào đối tượng hiện có:

```json
    "funnel_metrics":["Phần trăm khách active thấy chỗ giới thiệu add-on","Phần trăm bấm tìm hiểu rồi dùng thử","Phần trăm dùng thử chuyển trả tiền"],
    "cohort_retention":"Theo từng tháng: phần trăm khách còn gửi ZNS sau 30/60/90 ngày",
```

- [ ] **Step 5: Sửa `decision.confidence` thành `0.752`** (khớp công thức — sample cũ 0.72 là sai).

- [ ] **Step 6: Thêm `"glossary"` vào `render`** — đối tượng `render` thành:

```json
  "render":{"rendered_at":"2026-06-13T09:41:00Z","output_path":"zns-designer-validation.html","template_version":"1.0.0",
    "glossary":{"North Star":"Một số duy nhất nói lên giá trị khách nhận được — mọi nỗ lực soi về số này","AARRR":"Khung đo 5 bước khách đi qua: biết đến → khởi động → ở lại → giới thiệu → trả tiền"}}
```

- [ ] **Step 7: Kiểm chứng red baseline** — mở `file:///D:/ZCode/.zcode/workspace/default/validate-suite/render/template.html` bằng Playwright MCP (`browser_navigate`), rồi `browser_evaluate`:

```js
() => ({
  asmTable: document.querySelectorAll("table.asm").length,          // mong đợi 0 (chưa render)
  secondOrder: [...document.querySelectorAll(".critique h3")].map(h=>h.textContent).join("|"), // mong đợi KHÔNG có "Hệ quả dây chuyền"
  gtmRows: [...document.querySelectorAll(".kv .k")].map(x=>x.textContent).join("|"),           // mong đợi KHÔNG có "Thông điệp chính"/"Bước launch"/"AARRR"/"Mục tiêu"
  consoleClean: true
})
```

Expected: `asmTable: 0`; các chuỗi mới vắng mặt. Trang vẫn render bình thường, console không lỗi (`browser_console_messages` level error → rỗng).

- [ ] **Step 8: Commit**

```bash
git add render/template.html
git commit -m "test: nâng cấp sample Dossier thành fixture chuẩn cho các block sắp render"
```

---

### Task 2: Render bảng giả định `framing.assumptions`

**Files:**
- Modify: `render/template.html` (CSS block ~dòng 88, label consts ~dòng 360, hàm `renderFraming` ~dòng 442–451)

**Interfaces:**
- Consumes: `framing.assumptions: [{id, text, risk, uncertainty, status}]` (schema 4.3; status ∈ UNTESTED|VALIDATED|INVALIDATED|PARTIAL).
- Produces: bảng `<table class="asm">`, dòng H×H có class `asm-hot`.

- [ ] **Step 1: Thêm CSS** — chèn sau block `.problem .k{...}` (trước `/* routing / pipeline ledger */`):

```css
  /* assumption table */
  table.asm{width:100%;border-collapse:collapse;font-size:14px;margin-top:18px}
  table.asm th,table.asm td{padding:9px 12px;border-bottom:1px solid var(--line);text-align:left;vertical-align:top}
  table.asm thead th{font-family:var(--mono);font-size:11px;letter-spacing:.04em;text-transform:uppercase;color:var(--rail-soft);font-weight:500}
  table.asm .aid{font-family:var(--mono);color:var(--rail-soft);white-space:nowrap}
  table.asm .lv{font-family:var(--mono);font-size:12px}
  .lv-H{color:var(--nogo)} .lv-M{color:var(--pivot)} .lv-L{color:var(--go)}
  tr.asm-hot{background:#FBE9E7}
  .asm-note{font-size:12.5px;color:var(--ink-soft);margin-top:6px}
```

- [ ] **Step 2: Thêm label map** — chèn sau dòng `const TYPE_LABEL={...}`:

```js
const ASM_STATUS_LABEL={UNTESTED:"Chưa kiểm",VALIDATED:"Đã xác thực",INVALIDATED:"Đã bác bỏ",PARTIAL:"Một phần"};
```

- [ ] **Step 3: Viết lại `renderFraming`** — thay toàn bộ hàm:

```js
function renderFraming(d){
  const w=d.framing?.five_w_one_h;
  const as=d.framing?.assumptions||[];
  if(!w&&!as.length) return "";
  const map=[["what","What · Cái gì"],["why","Why · Vì sao"],["who","Who · Ai"],
    ["when","When · Khi nào"],["where","Where · Ở đâu"],["how","How · Thế nào"]];
  const cells=w?map.map(([k,lab])=>`<div class="w-cell"><div class="k">${lab}</div><div>${esc(w[k])}</div></div>`).join(""):"";
  let extra="";
  if(d.framing.problem_statement)
    extra=`<div class="problem"><div class="k">Vấn đề cốt lõi</div><div>${esc(d.framing.problem_statement)}</div></div>`;
  let asm="";
  if(as.length){
    const lv=x=>`<span class="lv lv-${esc(x)}">${esc(x)}</span>`;
    const rows=as.map(a=>`<tr class="${(a.risk==="H"&&a.uncertainty==="H")?"asm-hot":""}">
      <td class="aid">${esc(a.id)}</td><td>${esc(a.text)}</td>
      <td>${lv(a.risk)}</td><td>${lv(a.uncertainty)}</td>
      <td>${esc(ASM_STATUS_LABEL[a.status]||a.status||"—")}</td></tr>`).join("");
    asm=`<table class="asm"><thead><tr><th>ID</th><th>Giả định</th><th>Rủi ro</th><th>Bất định</th><th>Trạng thái</th></tr></thead>
      <tbody>${rows}</tbody></table>
      <p class="asm-note">Dòng đỏ = giả định rủi ro cao và chưa rõ — cần kiểm chứng gấp nhất, tầng xác thực giải pháp sẽ nhắm vào nó trước.</p>`;
  }
  return sec("","Đóng khung vấn đề (6 câu hỏi nền)",`<div class="w-grid">${cells}</div>${extra}${asm}`);
}
```

- [ ] **Step 4: Kiểm chứng xanh** — reload trang, `browser_evaluate`:

```js
() => ({
  rows: document.querySelectorAll("table.asm tbody tr").length,       // 3
  hot: document.querySelectorAll("tr.asm-hot").length,                // 1
  hotIsA3: document.querySelector("tr.asm-hot .aid")?.textContent,    // "A3"
  lastCol: [...document.querySelectorAll("tr.asm-hot td")].pop().textContent // "Chưa kiểm"
})
```

Expected: `{rows:3, hot:1, hotIsA3:"A3", lastCol:"Chưa kiểm"}`. Console không lỗi.

- [ ] **Step 5: Commit**

```bash
git add render/template.html
git commit -m "feat(template): render bảng giả định framing.assumptions, nổi bật dòng rủi ro cao"
```

---

### Task 3: Wire lens `second-order` (stage 6 + block render)

**Files:**
- Modify: `stages/decision-gate/SKILL.md` (Step 3 ~dòng 56–65, Writes ~dòng 171–175)
- Modify: `render/template.html` (hàm `renderCritique` ~dòng 494–513)

**Interfaces:**
- Consumes: `intake.tags` chứa `downstream_effects`; lens `lenses/second-order.md`.
- Produces: `lenses.second_order: [{action, then, and_then}]` (schema 4.5); block render trong critique.

- [ ] **Step 1: Sửa stage doc** — trong `### Step 3 — Critique lenses`, thêm bullet thứ tư sau `lenses.red_team`:

```markdown
- `lenses.second_order` — **chỉ khi tag `downstream_effects` bật** (`../../lens-registry.md`;
  tag tắt → không gọi, không ghi gì): với mỗi hành động chính của quyết định, truy "rồi sao nữa?"
  hai bậc (`action → then → and_then`) — phản ứng đối thủ, thay đổi hành vi khách, hệ quả lên các
  sản phẩm khác trong portfolio. Đặc biệt soi kỹ với quyết định Type-1. Xem `../../lenses/second-order.md`.
```

- [ ] **Step 2: Sửa Writes list của stage doc** — dòng đầu của "What this stage writes" thành:

```markdown
- `lenses.six_hats`, `lenses.party_mode`, `lenses.premortem`, `lenses.inversion`, `lenses.red_team`, `lenses.second_order` (chỉ khi tag `downstream_effects`), `lenses.decision_matrix`
```

- [ ] **Step 3: Sửa `renderCritique` trong template** — thay 2 dòng đầu và cột phải:

```js
function renderCritique(d){
  const L=d.lenses||{}; const pm=L.premortem||[]; const inv=L.inversion||[]; const rt=L.red_team||[]; const so=L.second_order||[];
  if(!pm.length&&!inv.length&&!rt.length&&!so.length) return "";
```

và trong khối `<div class="crit-col">` thứ hai, sau `${rtHtml}` thêm biến + chèn. Định nghĩa `soHtml` cạnh `rtHtml`:

```js
  const soHtml=so.length?`<h3 style="margin-top:18px"><span class="dot"></span>Hệ quả dây chuyền (rồi sao nữa?)</h3>
    <ul class="crit-list">${so.map(x=>`<li><b>${esc(x.action)}</b> → ${esc(x.then)} → ${esc(x.and_then)}</li>`).join("")}</ul>`:"";
```

cột phải thành `<div class="crit-col">${invHtml}${rtHtml}${soHtml}</div>`.

- [ ] **Step 4: Kiểm chứng** — reload, `browser_evaluate`:

```js
() => ({
  heading: [...document.querySelectorAll(".critique h3")].map(h=>h.textContent).includes("Hệ quả dây chuyền (rồi sao nữa?)"), // true
  chains: document.querySelectorAll(".crit-list li b").length,   // ≥2
  docWired: null // grep riêng bên dưới
})
```

Expected: heading true, ≥2 chuỗi. Grep stage doc:

```bash
grep -n "second_order" stages/decision-gate/SKILL.md   # ≥2 dòng (Step 3 + Writes)
```

- [ ] **Step 5: Commit**

```bash
git add stages/decision-gate/SKILL.md render/template.html
git commit -m "feat: wire lens second-order vào stage 6 + block render hệ quả dây chuyền"
```

---

### Task 4: `renderPlan` đầy đủ + bỏ cổng `verdict === "GO"`

**Files:**
- Modify: `render/template.html` (hàm `renderPlan` ~dòng 608–625, CSS ~dòng 236)
- Modify: `dossier-schema.md` (heading §4.7, dòng 264)

**Interfaces:**
- Consumes: `gtm_plan.messaging/launch_steps/aarrr/okrs`, `lifecycle.funnel_metrics/cohort_retention` (schema 4.7/4.8).
- Produces: block "Kế hoạch & vòng đời" hiện khi có dữ liệu, bất kể verdict.

- [ ] **Step 1: Thêm CSS** — chèn sau `.chip{...}`:

```css
  .steps{margin:0;padding-left:18px}
  .steps li{margin-bottom:4px;font-size:14px}
```

- [ ] **Step 2: Thêm label map** — cạnh các LABEL khác:

```js
const AARRR_LABEL={acquisition:"Biết đến",activation:"Khởi động",retention:"Ở lại",referral:"Giới thiệu",revenue:"Trả tiền"};
```

- [ ] **Step 3: Viết lại `renderPlan`**:

```js
function renderPlan(d){
  const g=d.gtm_plan; const lc=d.lifecycle; // render khi có dữ liệu — không chặn theo verdict (plan-mode: sản phẩm đang sống = GO sẵn)
  if(!g&&!lc) return "";
  const chips=arr=>`<div class="chips">${(arr||[]).map(x=>`<span class="chip">${esc(x)}</span>`).join("")}</div>`;
  const steps=arr=>`<ul class="steps">${(arr||[]).map(x=>`<li>${esc(x)}</li>`).join("")}</ul>`;
  let rows="";
  if(g){
    if(g.positioning) rows+=`<div class="row"><div class="k">Định vị</div><div>${esc(g.positioning)}</div></div>`;
    if(g.beachhead) rows+=`<div class="row"><div class="k">Nhóm khách đầu cầu</div><div>${esc(g.beachhead)}</div></div>`;
    if(g.messaging) rows+=`<div class="row"><div class="k">Thông điệp chính</div>${steps(g.messaging)}</div>`;
    if(g.channels) rows+=`<div class="row"><div class="k">Kênh</div>${chips(g.channels)}</div>`;
    if(g.north_star_metric) rows+=`<div class="row"><div class="k">Chỉ số quan trọng nhất (North Star)</div><div>${esc(g.north_star_metric)}</div></div>`;
    if(g.launch_steps) rows+=`<div class="row"><div class="k">Bước launch</div><ol class="steps">${g.launch_steps.map(x=>`<li>${esc(x)}</li>`).join("")}</ol></div>`;
    if(g.aarrr&&Object.keys(g.aarrr).length) rows+=`<div class="row"><div class="k">Khung đo AARRR — 5 bước khách đi qua</div><div class="chips">${Object.entries(g.aarrr).map(([k,v])=>`<span class="chip"><b>${AARRR_LABEL[k]||esc(k)}</b>: ${esc(v)}</span>`).join("")}</div></div>`;
    if(g.okrs) rows+=g.okrs.map(o=>`<div class="row"><div class="k">Mục tiêu: ${esc(o.objective)}</div>${steps(o.key_results)}</div>`).join("");
  }
  if(lc){
    if(lc.funnel_metrics) rows+=`<div class="row"><div class="k">Chỉ số phễu cần đo</div>${chips(lc.funnel_metrics)}</div>`;
    if(lc.cohort_retention) rows+=`<div class="row"><div class="k">Giữ khách theo cohort</div><div>${esc(lc.cohort_retention)}</div></div>`;
    if(lc.expand_criteria) rows+=`<div class="row"><div class="k">Tiêu chí mở rộng</div>${chips(lc.expand_criteria)}</div></div>`;
    if(lc.sunset_criteria) rows+=`<div class="row"><div class="k">Tiêu chí ngừng / thu hẹp</div>${chips(lc.sunset_criteria)}</div></div>`;
    if(lc.revalidation) rows+=`<div class="row"><div class="k">Xem lại định kỳ</div><div>${esc(lc.revalidation.trigger)} → tầng ${(lc.revalidation.stages_to_recheck||[]).join(", ")}</div></div>`;
  }
  return sec("","Kế hoạch & vòng đời",`<div class="kv">${rows}</div>`);
}
```

- [ ] **Step 4: Sửa heading schema §4.7** (`dossier-schema.md` dòng 264):

```markdown
### 4.7 `gtm_plan` (sau GO, hoặc plan-mode cho sản phẩm đang sống) & 4.8 `lifecycle`
```

- [ ] **Step 5: Kiểm chứng** — reload, `browser_evaluate`:

```js
() => {
  const ks=[...document.querySelectorAll(".kv .k")].map(x=>x.textContent);
  const hasAll=["Thông điệp chính","Bước launch","Khung đo AARRR","Mục tiêu:","Chỉ số phễu cần đo","Giữ khách theo cohort"].every(s=>ks.some(k=>k.includes(s)));
  DOSSIER.decision.verdict="PARK"; render(DOSSIER);   // mô phỏng plan-mode: không còn GO
  const ks2=[...document.querySelectorAll(".kv .k")].map(x=>x.textContent);
  return {hasAll, stillRendersGtmWithoutGO: ks2.some(k=>k.includes("Thông điệp chính"))};
}
```

Expected: `{hasAll:true, stillRendersGtmWithoutGO:true}`. (Sau test, reload trang để trả fixture về GO.) Console không lỗi.

- [ ] **Step 6: Grep xác nhận cổng GO đã bỏ:**

```bash
grep -n 'verdict==="GO"' render/template.html   # 0 kết quả
```

- [ ] **Step 7: Commit**

```bash
git add render/template.html dossier-schema.md
git commit -m "feat(template): render đủ gtm_plan/lifecycle; bỏ điều kiện verdict GO chặn plan"
```

---

### Task 5: Self-verify confidence + ngưỡng động theo decision_type

**Files:**
- Modify: `render/template.html` (hàm `renderHeader` ~dòng 403–427, thêm hàm `computeConfidence`)

**Interfaces:**
- Consumes: `stages[]` (stage_id 0–5, status, score) + `decision.confidence/confidence_threshold/decision_type`.
- Produces: badge `#confverify` dưới gauge: xanh "✓ Confidence khớp với số tự tính lại (x.xxx)" / đỏ "⚠ ... lệch ...".

- [ ] **Step 1: Thêm hàm tính lại** — chèn trước `function renderHeader(d){`:

```js
// Tái tạo công thức confidence của dossier-schema.md §4.6 — renderer tự đối chiếu.
function computeConfidence(d){
  const ok=new Set(["PASSED","ASSERTED","REVIEW_NEEDED"]);
  const vals=(d.stages||[]).filter(s=>s.stage_id<=5&&ok.has(s.status))
    .map(s=>(s.status==="REVIEW_NEEDED"?0.85:1)*(s.score||0));
  if(!vals.length) return null;
  const m=vals.reduce((a,b)=>a+b,0)/vals.length;
  return Math.min(1,Math.max(0,Math.round(m*1000)/1000));
}
```

- [ ] **Step 2: Sửa `renderHeader`** — thay dòng gauge và thêm badge:

Dòng `const c=dec.confidence, thr=dec.confidence_threshold??0.65;` thành:

```js
  const c=dec.confidence, thr=dec.confidence_threshold??(dec.decision_type==="TYPE_1"?0.75:0.65);
```

Sau dòng `requestAnimationFrame(...)` (trước comment `// makers`) chèn:

```js
  // self-verify confidence
  const oldCv=document.getElementById("confverify"); if(oldCv) oldCv.remove();
  const cc=computeConfidence(d);
  if(c!=null&&cc!=null){
    const okc=Math.abs(c-cc)<=0.005;
    const badge=document.createElement("div");
    badge.className="verify"+(okc?"":" bad"); badge.id="confverify";
    badge.textContent=(okc?"✓ Confidence khớp":"⚠ Confidence LỆCH")+" với số hệ thống tự tính lại ("+fmt(cc)+")";
    document.querySelector(".gauge").appendChild(badge);
  }
```

- [ ] **Step 3: Kiểm chứng xanh + đỏ** — reload, `browser_evaluate`:

```js
() => document.getElementById("confverify")?.textContent
// mong đợi: "✓ Confidence khớp với số hệ thống tự tính lại (0.752)"
```

Rồi test badge thật sự bắt lỗi — mutate rồi render lại:

```js
() => { DOSSIER.decision.confidence=0.9; render(DOSSIER);
  return document.getElementById("confverify")?.className + " | " + document.getElementById("confverify")?.textContent; }
// mong đợi: "verify bad | ⚠ Confidence LỆCH với số hệ thống tự tính lại (0.752)"
```

Test ngưỡng động:

```js
() => { DOSSIER.decision.confidence=0.752; DOSSIER.decision.confidence_threshold=null; DOSSIER.decision.decision_type="TYPE_1"; render(DOSSIER);
  return document.getElementById("gthr").textContent; }
// mong đợi: "/ ngưỡng 0.75" (fallback theo TYPE_1)
```

Reload trả fixture. Console không lỗi.

- [ ] **Step 4: Commit**

```bash
git add render/template.html
git commit -m "feat(template): tự verify confidence theo công thức schema + ngưỡng động theo decision_type"
```

---

### Task 6: Glossary single-source qua `render.glossary`

**Files:**
- Modify: `dossier-schema.md` (§4.9, dòng 279–282)
- Modify: `SKILL.md` (Step 5 — Render, ~dòng 117–120)
- Modify: `render/template.html` (`const TERMS` ~dòng 361–377)

**Interfaces:**
- Produces: `render.glossary: {term → 1 câu định nghĩa đời thường}` (optional, additive); template merge fallback + dữ liệu (dữ liệu thắng).

- [ ] **Step 1: Sửa schema §4.9** — thay cả block:

```jsonc
### 4.9 `render`
```jsonc
{ "rendered_at": "ISO8601", "output_path": "validation-report.html", "template_version": "1.0.0",
  "glossary": { "CAC": "một câu định nghĩa đời thường" } }
```
> `glossary` (optional): orchestrator đọc từ `glossary.md` và đổ vào đây trước khi render —
> **nguồn duy nhất** của bộ chú giải tooltip. Template chỉ giữ bộ fallback cho trường hợp mở
> file trực tiếp không có dữ liệu này; dữ liệu từ Dossier ghi đè fallback theo từng thuật ngữ.
```

(Chú ý giữ fences markdown đúng — block trên là nội dung thay thế cho mục 4.9 cũ.)

- [ ] **Step 2: Sửa `SKILL.md` Step 5** — sau câu "inject the full Dossier JSON into `render/template.html`...", thêm:

```markdown
Before injecting, populate `render.glossary`: read `glossary.md` and convert it into a
`{term: one-line plain-language definition}` map. The report's tooltips draw from that single
source — the template only keeps a static fallback for when a Dossier carries no glossary.
```

- [ ] **Step 3: Sửa template** — đổi `const TERMS={` thành `const FALLBACK_TERMS={` (nguyên vẹn nội dung bên trong), và chèn ngay sau dấu `};` đóng của nó:

```js
// Nguồn duy nhất của chú giải là render.glossary (orchestrator đổ từ glossary.md);
// bộ fallback chỉ lấp thuật ngữ thiếu khi mở file trực tiếp không kèm Dossier thật.
const TERMS=Object.assign({},FALLBACK_TERMS,DOSSIER.render?.glossary||{});
```

- [ ] **Step 4: Kiểm chứng** — reload, `browser_evaluate`:

```js
() => {
  const ab=[...document.querySelectorAll("abbr.term")].find(a=>a.textContent==="AARRR");
  const ns=[...document.querySelectorAll("abbr.term")].find(a=>a.textContent==="North Star");
  return { aarr: ab?.title, northStar: ns?.title };
}
// mong đợi: cả hai có title đúng như render.glossary trong fixture (chứa "5 bước khách đi qua" / "giá trị khách nhận được")
```

Và test dữ liệu thắng fallback:

```js
() => TERMS["CAC"] === FALLBACK_TERMS["CAC"] && TERMS["AARRR"] !== undefined
// mong đợi: true (CAC vẫn từ fallback vì glossary không có; AARRR từ render.glossary)
```

Console không lỗi. Grep:

```bash
grep -n "render.glossary\|FALLBACK_TERMS" render/template.html SKILL.md dossier-schema.md
```

- [ ] **Step 5: Commit**

```bash
git add dossier-schema.md SKILL.md render/template.html
git commit -m "feat: glossary một nguồn duy nhất qua render.glossary (fallback chỉ lấp thiếu)"
```

---

### Task 7: Gate ID cho hardgate + nhất quán khai báo Writes (stage 4/5) + căn sample theo ID chuẩn

**Files:**
- Modify: `stages/hardgate/SKILL.md` (mục Gates, dòng 14–21)
- Modify: `stages/biz-model/SKILL.md` (đổi chỗ G5.3/G5.4, dòng 42–46; Writes dòng 48–49)
- Modify: `stages/market-gtm/SKILL.md` (Writes dòng 31–32)
- Modify: `render/template.html` (sample `stages[1]` gates — căn theo ID chuẩn)

**Interfaces:**
- Produces: hardgate gate IDs chuẩn G1.1 THẬT · G1.2 ĐAU · G1.3 VỚI-TỚI-ĐƯỢC · G1.4 HỢP PORTFOLIO · G1.5 KHẢ THI · G1.6 HỢP PHÁP. Sample Dossier chỉnh theo stage doc (không ngược).

- [ ] **Step 1: hardgate — gán ID** — thay 6 dòng list trong `## Gates`:

```markdown
1. **G1.1 · THẬT** — is the problem real (does the situation actually occur for real people)?
2. **G1.2 · ĐAU** — is it painful enough that someone would pay / change behavior to solve it?
3. **G1.3 · VỚI-TỚI-ĐƯỢC** — is there a realistic channel to reach these people?
4. **G1.4 · HỢP PORTFOLIO** — does it fit the company's focus/strategy? (Flag, don't auto-fail, if it's a
   defensible departure — note it for the decision gate's Six Hats.)
5. **G1.5 · KHẢ THI** — is it buildable with available resources/skills in a sane timeframe?
6. **G1.6 · HỢP PHÁP** — no legal/compliance blocker (data, payments, sector rules)?
```

- [ ] **Step 2: biz-model — đổi chỗ + Writes** — cắt nguyên block `G5.4` (dòng 37–41) đặt TRƯỚC block `G5.3` (nội dung mỗi block giữ nguyên), rồi Writes thành:

```markdown
`stages[]` record (`stage_id: 5`, gates G5.1 + G5.2 (+ G5.3 nếu chạm giá, + G5.4 nếu chạy quảng cáo trả tiền), status, score, assumptions in notes).
```

- [ ] **Step 3: market-gtm — Writes** — dòng Writes thành:

```markdown
`stages[]` record (`stage_id: 4`, gates G4.1 + G4.2, status, score), appends to `lenses.red_team`.
```

- [ ] **Step 4: Căn sample theo ID chuẩn** — trong template, `stages[1].gates` của fixture (hiện chỉ có G1.1 "Hợp portfolio", G1.2 "Khả thi") thay bằng 6 gate đủ + evidence_type (đồng thời bật được dòng tóm tắc chất lượng bằng chứng):

```json
    {"stage_id":1,"name":"HARD_GATE","mode":"MUST_RUN","status":"PASSED","score":1.0,"gates":[
      {"id":"G1.1","question":"Vấn đề có THẬT không?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"Khách hỏi tính năng ZNS qua support 3 tháng liền","evidence_type":"MEASURED"},
      {"id":"G1.2","question":"Có ĐAU đủ để trả tiền / đổi hành vi?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"Nhu cầu gửi ZNS có thương hiệu nêu trong phiếu hỗ trợ","evidence_type":"USER_STATED"},
      {"id":"G1.3","question":"VỚI-TỚI-ĐƯỢC qua kênh nào?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"Upsell in-app tới chính tập khách hiện có","evidence_type":"MEASURED"},
      {"id":"G1.4","question":"Hợp portfolio Product X?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"Fast-follow đã nêu trong roadmap sản phẩm","evidence_type":"CITED","evidence_refs":["roadmap-2026.md#q2"]},
      {"id":"G1.5","question":"Khả thi kỹ thuật trong khung thời gian hợp lý?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"~80% hạ tầng sẵn, ước tính 2–3 tuần","evidence_type":"ASSUMED"},
      {"id":"G1.6","question":"Có chướng ngại pháp lý / tuân thủ?","verdict":"PASS","score":1.0,"threshold":1.0,"evidence":"ZNS là kênh chính thống của Zalo OA, không vi phạm chính sách","evidence_type":"CITED","evidence_refs":["zalo.ca/docs/zns"]}]},
```

- [ ] **Step 5: Kiểm chứng** — greps:

```bash
grep -n "G1\.[1-6]" stages/hardgate/SKILL.md            # 6 dòng
grep -n "G5.3" stages/biz-model/SKILL.md | head -1      # số dòng G5.3 < số dòng G5.4
grep -n "G4.1 + G4.2" stages/market-gtm/SKILL.md        # 1 dòng
```

Reload template — `browser_evaluate`:

```js
() => ({
  gateCount: document.querySelectorAll(".gate").length,                       // ≥6 (chỉ tính tầng 1 là 6)
  hardGates: [...document.querySelectorAll(".gate .gid")].map(g=>g.textContent).slice(0,6).join(","),
  // "G1.1, G1.2, G1.3, G1.4, G1.5, G1.6"
  evqPresent: !!document.querySelector(".evq"),                                 // true — tóm tắt chất lượng bằng chứng bật
  confidenceStillGreen: document.getElementById("confverify")?.textContent.includes("khớp")  // true
})
```

Console không lỗi.

- [ ] **Step 6: Commit**

```bash
git add stages/hardgate/SKILL.md stages/biz-model/SKILL.md stages/market-gtm/SKILL.md render/template.html
git commit -m "docs: gate ID chuẩn cho hardgate; nhất quán Writes ở stage 4/5; sample căn theo ID"
```

---

### Task 8: Đối chiếu kiểm chứng cuối theo spec

**Files:**
- Không tạo file mới; chỉ kiểm chứng + commit nếu phát hiện sót.

- [ ] **Step 1: Mở `file:///D:/ZCode/.zcode/workspace/default/validate-suite/render/template.html`** — `browser_console_messages` (level error) → rỗng. Screenshot toàn trang lưu lại để đối chiếu thị giác.

- [ ] **Step 2: Chạy checklist spec** (5 mục "Cách kiểm chứng khi hoàn tất" trong spec):
  1. Mọi block mới hiện (assumptions, second_order, gtm đủ, lifecycle đủ) + 2 badge self-verify xanh.
  2. Mutate verdict → PARK: gtm_plan vẫn hiện.
  3. Mutate confidence lệch: badge đỏ.
  4. Greps: không còn `verdict==="GO"`; hardgate có G1.1–G1.6; stage 4/5 Writes đủ gate.
  5. Đọc đối chiếu 7 mục thay đổi trong spec.

- [ ] **Step 3: Nếu mọi thứ xanh** — commit phần còn sót (nếu có) bằng `chore: đối chiếu kiểm chứng cuối theo spec`; nếu không có gì đổi thì không commit.

- [ ] **Step 4: Báo cáo kết quả** cho user: danh sách đã làm × kết quả kiểm chứng, đề xuất bước merge branch `standardize-fixes` về `main`.
