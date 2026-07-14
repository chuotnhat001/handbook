# CLAUDE.md

## Project Overview

Personal knowledge base — static HTML pages served via `file://`. No build tools, no server, no dependencies. Each card = 1 topic, self-contained.

## Constraints (non-negotiable)

- All CSS/JS inline in each page. No external files. Must work via `file://`.
- Vietnamese for UI text. English for code/technical terms.
- Relative links only. Subfolder pages use `../` or `../../`.
- Light theme only. Font: `-apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif`.

## Architecture

- `index.html` — Hub page with card grid, rendered from inline JS `groups[]` array.
- Groups = folders (`ai-engineering/`, `productivity/`, `database/`, `english/`).
- Each page is self-contained (inline `<style>` + `<script>`).

### Hub Page `groups[]` Format

```js
{ name: "Group Name", topics: [
  { title: "Title", url: "folder/file.html", desc: "Vietnamese desc <100 chars", icon: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">...</svg>', color: "#hex" }
]}
```

## Page Templates

### 1. Content pages (most common)

Layout: fixed sidebar (260px) + main content (max-width 860px).

### 1b. Curriculum pages (multi-page sets: Claude AIO, Agile Practice Guide, CRM Knowledge Base...)

Layout: fixed sidebar trái (260px, nav trong trang) + main content + fixed aside phải (220px, nav giữa các trang).

Áp dụng khi: nhóm có ≥5 sub-pages liên kết (giáo trình, knowledge base, series bài học).

Aside phải chứa: danh sách tất cả pages trong nhóm, highlight trang hiện tại (`class="current"`), chia sections bằng `.level-label`.

CSS bổ sung cho aside:
```css
.aside{width:220px;position:fixed;top:0;right:0;bottom:0;display:flex;flex-direction:column;background:#fafbff;border-left:1px solid rgba(102,126,234,.12);font-size:.78rem;overflow-y:auto;padding:1rem 0}
.aside-title{padding:.6rem 1rem;font-size:.7rem;font-weight:700;text-transform:uppercase;letter-spacing:.05em;color:#4c51bf}
.aside a{display:block;padding:.25rem 1rem;color:#64748b;text-decoration:none;line-height:1.4;transition:all .15s}
.aside a:hover{color:#4c51bf;background:rgba(102,126,234,.06)}
.aside a.current{color:#4c51bf;font-weight:600;background:rgba(102,126,234,.1);border-right:3px solid #667eea}
.aside .level-label{padding:.4rem 1rem .15rem;font-size:.65rem;font-weight:700;color:#94a3b8;text-transform:uppercase;letter-spacing:.04em;margin-top:.4rem}
.main{margin-right:220px}
@media(max-width:1024px){.aside{display:none}.main{margin-right:0}}
@media(max-width:768px){.nav{display:none}.main{margin-left:0;margin-right:0;padding:1.5rem 1rem}}
```

HTML aside đặt giữa `</nav>` và `<main>`:
```html
<aside class="aside">
  <div class="aside-title">Tên Nhóm</div>
  <div class="level-label">Section Label</div>
  <a href="page.html">Page Title</a>
  <a href="current.html" class="current">Current Page</a>
</aside>
```

Components: `.source-box`, `.callout-{blue,green,amber,red}`, `table`, `pre` with `.copy-btn`, `.flow` diagrams, `blockquote`.

Content pattern: `h1` + `.subtitle` → `.source-box` → intro paragraph → `.callout-blue` "tư duy cốt lõi" → sections (`h2`/`h3` with `id` matching nav links).

Callout usage: blue=info, green=tip, amber=warning, red=critical.

### 2. Grammar cards (`english/01-12*.html`, `review-*.html`)

Same layout + `.formula`, `.compare-vn`, `.mistakes`, `.exercise` + `.toggle-btn` + `.answer`, breadcrumb + prev/next nav.

### 3. Exercise pages (`english/exercises/day-*.html`)

Same layout + `.q` blocks, radio/text inputs, `.score-box`, `.btn-submit`. Paths go up two levels (`../../index.html`).

## Adding Content

**New page:** Copy existing page from same group → replace content (keep CSS/layout identical) → add entry to `groups[]` in `index.html`.

**New group:** Create folder → add pages → add new object to `groups[]`.

## Tạo thẻ từ nguồn bên ngoài

### Workflow cơ bản (1 URL ngắn)

1. **Research** — Agent `web-researcher` fetch + tổng hợp. Chỉ trả TEXT.
   ```
   Agent(subagent_type: "web-researcher", prompt: "Fetch và tổng hợp từ {URL}. CHỈ trả TEXT.")
   ```
2. **Implement** — Main context viết HTML từ CSS/JS reference bên dưới → cập nhật `index.html`.

### Workflow nâng cao (nhiều URL hoặc nguồn dài)

Khi có nhiều URL hoặc nguồn thông tin lớn:
- Mỗi URL giao cho 1 agent riêng biệt, chạy song song.
- Mỗi agent chỉ trả TEXT tổng hợp của phần mình phụ trách.
- Main context nhận kết quả từ tất cả agents → tổng hợp thành bản hoàn chỉnh → viết HTML.

```
// Chạy song song nhiều agents
Agent(subagent_type: "web-researcher", prompt: "Fetch {URL1}. Tập trung vào {khía cạnh A}. CHỈ trả TEXT.")
Agent(subagent_type: "web-researcher", prompt: "Fetch {URL2}. Tập trung vào {khía cạnh B}. CHỈ trả TEXT.")
Agent(subagent_type: "web-researcher", prompt: "Fetch {URL3}. Tập trung vào {khía cạnh C}. CHỈ trả TEXT.")
```

Lý do tách agent:
- Agent viết HTML lớn hay fail. Research song song (~30s) → main tổng hợp + viết HTML = nhanh + reliable.
- Nguồn dài vượt context 1 agent → chia nhỏ giữ chất lượng tổng hợp.
- Mỗi agent tập trung 1 khía cạnh → output chính xác hơn.

## Tạo giáo trình (Curriculum)

Khi user cung cấp file danh sách bài học (nhiều URL, chia theo cấp độ), tạo một nhóm topic riêng với cấu trúc:

### Cấu trúc output

```
{folder-name}/
├── index.html              ← Trang tổng quan giáo trình (card grid)
├── 01-slug-bai-1.html      ← Sub-page bài 1
├── 02-slug-bai-2.html      ← Sub-page bài 2
└── ...
```

- Folder riêng cho giáo trình (không nằm trong `ai-engineering/`).
- File đặt tên numeric prefix (`01-`, `02-`...) để sort tự nhiên.
- Links ngắn (cùng folder), relative path lên hub dùng `../`.

### Trang Index giáo trình

- Header gọn (1 dòng ngang): back-link + title + mô tả + metadata.
- Body: card grid chia sections theo cấp độ.
- Mỗi section: badge số cấp + heading + số bài + cards.
- Mỗi card: icon (màu theo cấp) + số bài + tên + mô tả ngắn.
- Màu phân biệt cấp: xanh lá (nhập môn) → tím (trung cấp) → vàng cam (nâng cao) → đỏ (chuyên sâu).
- Footer: link nguồn gốc.

### Sub-pages

- Dùng Content Page template chuẩn (sidebar + main, xem CSS/JS Reference bên dưới).
- Source box link về URL bài gốc.
- Nội dung: full extraction — giữ nguyên toàn bộ kiến thức, ví dụ, bước thực hành từ bài gốc.
- Sidebar nav theo sections trong bài.
- Prev/Next navigation giữa các bài (tương tự English section).

### Workflow build

1. **Research song song** — Mỗi URL giao 1 agent `web-researcher`, chạy 3-4 URL/batch song song.
   ```
   Agent(subagent_type: "web-researcher", prompt: "Fetch {URL}. Trích xuất TOÀN BỘ nội dung bài học. CHỈ trả TEXT có cấu trúc (headings, lists, code blocks, tips).")
   ```
2. **Gen sub-pages** — Main context nhận text → viết HTML theo template chuẩn, mỗi bài 1 file.
3. **Tạo index** — Gen trang index card grid với links đến tất cả sub-pages.
4. **Cập nhật hub** — Thêm 1 card đại diện vào `groups[]` trong `index.html` gốc, link đến `{folder}/index.html`.

### Hub page entry

Thêm vào group "AI Engineering" (hoặc group phù hợp):
```js
{ title: "Tên Giáo Trình", url: "{folder}/index.html", desc: "Mô tả ngắn <100 ký tự", icon: '<svg ...>', color: "#hex" }
```

## Content Page CSS/JS Reference

Dùng trực tiếp khi tạo thẻ mới — không cần đọc lại template file.

### Full CSS (copy nguyên khối vào `<style>`)

```css
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',system-ui,sans-serif;background:#ffffff;color:#1a1a2e;line-height:1.7;min-height:100vh;font-size:16px}
.layout{display:flex;min-height:100vh}
.nav{width:260px;position:fixed;top:0;left:0;bottom:0;display:flex;flex-direction:column;background:#f0f4ff;border-right:1px solid rgba(102,126,234,.15);font-size:.83rem}
.nav-home{display:flex;align-items:center;gap:8px;padding:.75rem 1.25rem;color:#4c51bf;text-decoration:none;font-weight:700;font-size:.88rem;border-bottom:1px solid rgba(102,126,234,.15);flex-shrink:0}
.nav-home:hover{color:#667eea;background:rgba(102,126,234,.08)}
.nav-home svg{width:16px;height:16px}
.nav-scroll{flex:1;overflow-y:auto;padding:1rem 0}
.nav-group{margin-bottom:.25rem}
.nav-group-title{display:flex;align-items:center;gap:6px;padding:.5rem 1.25rem;font-size:.72rem;font-weight:700;text-transform:uppercase;letter-spacing:.06em;color:#4c51bf}
.nav-items{padding:0 .75rem .5rem 1.25rem}
.nav-items a{display:block;padding:.35rem .7rem;color:#2d3748;text-decoration:none;border-radius:6px;margin-bottom:2px;line-height:1.4;font-size:.82rem;font-weight:500;transition:all .15s}
.nav-items a:hover{background:rgba(102,126,234,.12);color:#4c51bf}
.nav-items a.active{background:#667eea;color:#fff;font-weight:600}
pre{position:relative}
.copy-btn{position:absolute;top:8px;right:8px;background:rgba(255,255,255,.15);border:1px solid rgba(255,255,255,.2);color:#e2e8f0;padding:4px 10px;border-radius:5px;font-size:.72rem;cursor:pointer;opacity:0;transition:opacity .2s}
pre:hover .copy-btn{opacity:1}
.copy-btn:hover{background:rgba(255,255,255,.25)}
.copy-btn.copied{background:#48bb78;border-color:#48bb78;color:#fff}
.main{flex:1;margin-left:260px;padding:2rem 2.5rem 4rem;max-width:860px}
h1{font-size:1.6rem;font-weight:700;margin-bottom:.4rem;color:#1a1a2e}
.subtitle{color:#555b6e;font-size:.9rem;margin-bottom:2rem}
h2{font-size:1.2rem;font-weight:700;margin:2.5rem 0 .75rem;padding-bottom:.5rem;border-bottom:1px solid rgba(0,0,0,.08);color:#1a1a2e}
h3{font-size:.95rem;font-weight:600;margin:1.5rem 0 .5rem;color:#1a1a2e}
p{margin-bottom:.75rem;font-size:.95rem;color:#1a1a2e}
ul,ol{margin:.5rem 0 1rem 1.5rem;font-size:.95rem}
li{margin-bottom:.3rem}
strong{font-weight:600}
code{font-family:'SF Mono','Cascadia Code','Consolas',monospace;font-size:.85rem;background:#f1f3f5;padding:2px 6px;border-radius:4px;border:1px solid rgba(0,0,0,.06)}
pre{background:#1a1a2e;color:#e2e8f0;padding:1rem 1.25rem;border-radius:8px;overflow-x:auto;margin:1rem 0;font-size:.85rem;line-height:1.65;border:1px solid rgba(0,0,0,.1)}
pre code{background:none;padding:0;color:inherit;border:none}
.callout{border-radius:8px;padding:.85rem 1.1rem;margin:1rem 0;font-size:.92rem;border-left:3px solid}
.callout strong{display:block;margin-bottom:4px;font-size:.82rem}
.callout-blue{background:#e6f1fb;color:#185fa5;border-color:#185fa5}
.callout-green{background:#eaf3de;color:#3b6d11;border-color:#3b6d11}
.callout-amber{background:#faeeda;color:#854f0b;border-color:#854f0b}
.callout-red{background:#fcebeb;color:#a32d2d;border-color:#a32d2d}
.source-box{display:flex;align-items:center;gap:.75rem;background:linear-gradient(135deg,#f0f4ff 0%,#e8eeff 100%);border:1px solid rgba(102,126,234,.25);border-radius:10px;padding:.75rem 1.1rem;margin:0 0 2rem;font-size:.85rem}
.source-box svg{width:18px;height:18px;color:#4c51bf;flex-shrink:0}
.source-box a{color:#4c51bf;font-weight:600;text-decoration:none}
.source-box a:hover{text-decoration:underline}
.source-box span{color:#555b6e}
table{width:100%;border-collapse:collapse;font-size:.88rem;margin:1rem 0}
th,td{text-align:left;padding:.5rem .75rem;border:1px solid rgba(0,0,0,.08)}
th{background:#f1f3f5;font-weight:600;font-size:.78rem;text-transform:uppercase;letter-spacing:.03em;color:#555b6e}
blockquote{border-left:3px solid #667eea;padding:.75rem 1rem;margin:1rem 0;background:#f0f4ff;border-radius:0 8px 8px 0;font-size:.85rem;color:#4a5568;font-style:italic}
.flow{display:flex;align-items:center;flex-wrap:wrap;gap:.5rem;margin:1.5rem 0}
.flow-step{background:#2563eb;color:#fff;padding:.4rem .9rem;border-radius:6px;font-size:.82rem;font-weight:600}
.flow-arrow{color:#94a3b8;font-size:1.1rem}
@media(max-width:768px){.nav{display:none}.main{margin-left:0;padding:1.5rem 1rem}}
```

### HTML Skeleton

```html
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{Title} - Handbook</title>
<style>/* CSS above */</style>
</head>
<body>
<div class="layout">
<nav class="nav" id="nav">
  <a href="../index.html" class="nav-home">
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 12l9-9 9 9"/><path d="M9 21V9h6v12"/></svg>
    Trang chủ
  </a>
  <div class="nav-scroll">
  <div class="nav-group">
    <div class="nav-group-title">{Section}</div>
    <div class="nav-items">
      <a href="#id">Link</a>
    </div>
  </div>
  </div>
</nav>
<main class="main">
<h1 id="overview">{Title}</h1>
<p class="subtitle">{One-line desc}</p>
<div class="source-box">
<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M13.19 8.688a4.5 4.5 0 011.242 7.244l-4.5 4.5a4.5 4.5 0 01-6.364-6.364l1.757-1.757m9.86-2.54a4.5 4.5 0 00-6.364-6.364L4.5 8.25a4.5 4.5 0 001.242 7.244"/></svg>
<span>Nguồn: <a href="{URL}" target="_blank">{display}</a> — {meta}</span>
</div>
<!-- Content here -->
</main>
</div>
<script>
function copyCode(btn){var pre=btn.parentElement;var code=pre.querySelector('code');navigator.clipboard.writeText(code.innerText).then(function(){btn.textContent='Copied!';btn.classList.add('copied');setTimeout(function(){btn.textContent='Copy';btn.classList.remove('copied')},2000)})}
document.querySelectorAll('.nav-items a[href^="#"]').forEach(function(a){a.addEventListener('click',function(e){var el=document.getElementById(this.getAttribute('href').slice(1));if(el){e.preventDefault();el.scrollIntoView({behavior:'smooth'})}})});
var observer=new IntersectionObserver(function(entries){entries.forEach(function(entry){if(entry.isIntersecting){document.querySelectorAll('.nav-items a.active').forEach(function(a){a.classList.remove('active')});var link=document.querySelector('.nav-items a[href="#'+entry.target.id+'"]');if(link)link.classList.add('active')}})},{threshold:0.3,rootMargin:'-80px 0px -60% 0px'});
document.querySelectorAll('h2[id],h3[id]').forEach(function(el){observer.observe(el)});
</script>
</body>
</html>
```

### Key HTML Snippets

Code block: `<pre><code>...</code><button class="copy-btn" onclick="copyCode(this)">Copy</button></pre>`

Flow diagram: `<div class="flow"><span class="flow-step">A</span><span class="flow-arrow">→</span><span class="flow-step">B</span></div>`

## English Section

`english/` — 12 grammar cards, 10 topic cards, 4 reviews, 30 daily exercises, 1 quick-reference. Has its own `index.html` with status indicators (`done`, `current`, `upcoming`).

## Mockups

`mockups/` — Design prototypes (mockup-a through g). Not listed in hub page.
