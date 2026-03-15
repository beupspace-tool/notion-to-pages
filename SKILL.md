---
name: notion-to-pages
description: Convert a Notion page into a BEUP-branded public static site on GitHub Pages. Handles VI/EN bilingual, Mermaid diagrams, callouts, tables, and all Notion block types. Use when user wants to publish Notion content as a website or GitHub Pages site.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - mcp__claude_ai_Notion__notion-fetch
  - mcp__claude_ai_Notion__notion-search
---

# /notion-to-pages — BEUP Notion → GitHub Pages Publisher

Chuyển 1 hoặc 2 Notion page (VI + EN) thành static site BEUP-branded trên GitHub Pages.

## Usage

```
/notion-to-pages --url <notion-url>
/notion-to-pages --url <vi-notion-url> --en <en-notion-url>
/notion-to-pages --url <notion-url> --repo <owner/repo-name>
/notion-to-pages --url <vi-url> --en <en-url> --repo <owner/repo> --title "My Guide"
```

**Flags:**
- `--url` — Notion page URL (Vietnamese / default)
- `--en` — Notion page URL for English version (optional, adds language switcher)
- `--repo` — GitHub repo `owner/repo-name` (default: auto-generate from page title)
- `--title` — Override page title
- `--out` — Local output dir (default: `/tmp/notion-pages-output`)

---

## BEUP Brand Identity

Luôn áp dụng đầy đủ các yếu tố sau:

### Color Palette (CSS Variables)
```css
:root {
  --brand:        #3fb7b6;   /* BEUP Teal — màu chủ đạo */
  --brand-dark:   #2d9190;
  --brand-darker: #1e6b6a;
  --brand-light:  #e8f8f8;
  --brand-border: #a8dedd;
  --blue:         #0693e3;
  --blue-light:   #e8f4fd;
  --blue-border:  #93d1f5;
  --green:        #00d084;
  --green-dark:   #009e62;
  --green-light:  #e6faf4;
  --green-border: #80e9c2;
  --yellow:       #e5a000;
  --yellow-light: #fef9e7;
  --yellow-border:#f7d87a;
  --purple:       #9b51e0;
  --purple-light: #f3ebfd;
  --purple-border:#d5aaff;
  --red:          #cf2e2e;
  --red-light:    #fdf0f0;
  --red-border:   #f0aaaa;
  --orange-light: #fff3e8;
  --orange-border:#f7c99a;
  --gray:         #6b7280;
  --gray-light:   #f7f8fa;
  --gray-mid:     #e8eaed;
  --gray-border:  #dde1e7;
  --text:         #1a1a2e;
  --text-muted:   #687080;
}
```

### Logo
```html
<img src="https://beup.space/wp-content/uploads/2025/12/logo-final-1536x530-1.png" alt="BEUP Logo">
```
Đặt logo tại: sidebar header, hero top-right corner, footer.

### Hero Gradient
```css
background: linear-gradient(140deg, var(--brand-darker) 0%, var(--brand) 55%, #00d084 100%);
```

### Typography
- Font: Inter (Google Fonts) + JetBrains Mono cho code
- Body: `font-size: 15px; line-height: 1.75`
- Headings: `font-weight: 700/800`, color `var(--text)`

### BEUP Signature Elements
- Beaver emoji 🦫 trong footer: `🦫 BEUP Learning Solutions`
- Link `beup.space` trong footer
- Tag line: "Learning Solutions"
- Copyright: `© 2026 BEUP Learning Solutions. All rights reserved.`

---

## Step 1: Parse Arguments

Đọc flags từ user input. Xác định:
- `PAGE_URL` — Notion URL chính (VI hoặc default)
- `EN_URL` — Notion URL tiếng Anh (nếu có)
- `REPO` — GitHub repo target
- `OUT_DIR` — thư mục output local (default `/tmp/notion-pages-output`)
- `BILINGUAL` — true nếu có `--en`

Tạo thư mục output:
```bash
mkdir -p $OUT_DIR/en   # nếu BILINGUAL
```

---

## Step 2: Fetch Notion Content

Dùng `notion-fetch` MCP tool để lấy nội dung trang.

### Với trang chính (VI/default):
```
notion-fetch(url: PAGE_URL)
```

### Nếu BILINGUAL:
```
notion-fetch(url: EN_URL)
```

Lấy toàn bộ: title, blocks, sub-pages.

**Lưu ý quan trọng:**
- Notion MCP trả về Markdown-like hoặc block JSON — cần parse thủ công
- Ghi nhớ: title, mô tả, danh sách sections/headings, callouts, code blocks, tables, toggles, mermaid diagrams

---

## Step 3: Parse & Convert Blocks to HTML

Chuyển từng block type sang HTML tương ứng:

### Heading Rules
```html
# h1  → <h1 id="s{n}">text</h1>
## h2 → <h2>text</h2>
### h3 → <h3>text</h3>
```

### Paragraph
```html
<p>text</p>
```

### Callout Block
```html
<div class="callout callout-{color}">
  <span class="callout-icon">{emoji}</span>
  <div class="callout-body">
    <b class="ct">{Title nếu có}</b>
    <p>nội dung...</p>
  </div>
</div>
```

**CRITICAL — Callout Title Rule:**
- Tiêu đề callout (dòng đầu in đậm kiểu "Lưu ý:", "Pro tip:", "Mẹo:") → dùng `<b class="ct">` (KHÔNG dùng `<strong>`)
- Text in đậm trong câu thông thường → dùng `<strong>` (luôn inline)
- CSS: `.ct { display: block; font-weight: 700; margin-bottom: 6px; }`
- CSS: `strong { font-weight: 700; }` — KHÔNG có display:block, KHÔNG có margin

### Code Block / Mermaid
Nếu block là ```mermaid:
```html
<div class="mermaid">{diagram code}</div>
```
Khác:
```html
<pre><code class="language-{lang}">{code}</code></pre>
```

### Table
```html
<div class="table-wrap">
  <table>
    <thead><tr><th>col1</th>...</tr></thead>
    <tbody><tr><td>val</td>...</tr></tbody>
  </table>
</div>
```

### Toggle / Details
```html
<details>
  <summary>{heading}</summary>
  <div class="details-body">{content}</div>
</details>
```

### Bullet / Numbered List
```html
<ul><li>item</li></ul>
<ol><li>item</li></ol>
```

### Column Layout (2 columns)
```html
<div class="cols">
  <div class="col">{col1}</div>
  <div class="col">{col2}</div>
</div>
```

### Quote / Blockquote
```html
<blockquote>{text}</blockquote>
```

### Divider
```html
<hr class="divider">
```

### Inline Formatting
- **bold** → `<strong>text</strong>`
- *italic* → `<em>text</em>`
- `code` → `<code>text</code>`
- [link](url) → `<a href="url">text</a>`
- ==highlight== → `<mark>text</mark>`

---

## Step 4: Build HTML — Vietnamese / Default Page

Tạo file `$OUT_DIR/index.html` theo template sau:

```html
<!DOCTYPE html>
<html lang="{vi|en}">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{TITLE} — BEUP Learning Solutions</title>
  <meta name="description" content="{DESCRIPTION}">
  <meta property="og:title" content="{TITLE} | BEUP">
  <meta property="og:description" content="{DESCRIPTION}">
  <meta property="og:type" content="article">
  <meta property="og:image" content="https://beup.space/wp-content/uploads/2025/12/logo-final-1536x530-1.png">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:ital,wght@0,400;0,500;0,600;0,700;0,800;1,400&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <style>
    /* FULL CSS — xem Section CSS bên dưới */
  </style>
</head>
<body>
  <div class="layout">
    <!-- SIDEBAR -->
    <nav class="sidebar">
      <div class="sidebar-header">
        <img class="sidebar-logo" src="https://beup.space/wp-content/uploads/2025/12/logo-final-1536x530-1.png" alt="BEUP Logo">
        <div class="sidebar-tagline">Learning Solutions</div>
      </div>
      <!-- LANG SWITCH — nếu BILINGUAL -->
      <div class="lang-switch">
        <a class="lang-btn active" href="#">🇻🇳 Tiếng Việt</a>
        <a class="lang-btn" href="en/">🇬🇧 English</a>
      </div>
      <!-- /LANG SWITCH -->
      <div class="sidebar-toc">
        <h3>Mục lục</h3>
        <ul>
          <!-- TOC links — tự động generate từ h1 headings -->
          <li><a href="#s1">1. Section 1</a></li>
          ...
        </ul>
      </div>
      <div class="sidebar-footer">
        🦫 BEUP Learning Solutions<br>
        <a href="http://beup.space">beup.space</a>
      </div>
    </nav>

    <!-- MAIN CONTENT -->
    <div class="main">
      <div class="hero">
        <div class="hero-brand">
          <img src="https://beup.space/wp-content/uploads/2025/12/logo-final-1536x530-1.png" alt="BEUP" class="hero-logo">
          <!-- Nếu BILINGUAL: -->
          <a class="hero-pill" href="en/">🇬🇧 English</a>
        </div>
        <h1 class="hero-title">{TITLE}</h1>
        <p class="hero-sub">{DESCRIPTION}</p>
        <div class="hero-meta">{META: tác giả, ngày, tags...}</div>
      </div>

      <div class="content">
        <!-- CONVERTED HTML CONTENT -->
      </div>

      <footer class="site-footer">
        <img src="https://beup.space/wp-content/uploads/2025/12/logo-final-1536x530-1.png" alt="BEUP" class="footer-logo">
        <p>© 2026 BEUP Learning Solutions. All rights reserved.</p>
        <p><a href="http://beup.space">beup.space</a></p>
      </footer>
    </div>
  </div>
  <script>
    /* Mermaid init + IntersectionObserver cho active TOC */
    mermaid.initialize({ startOnLoad: true, theme: 'base',
      themeVariables: { primaryColor: '#3fb7b6', primaryTextColor: '#1a1a2e',
        primaryBorderColor: '#2d9190', lineColor: '#3fb7b6', fontSize: '14px' }});

    const obs = new IntersectionObserver(entries => {
      entries.forEach(e => {
        const a = document.querySelector(`.sidebar a[href="#${e.target.id}"]`);
        if (a) a.classList.toggle('active', e.isIntersecting);
      });
    }, { rootMargin: '-10% 0px -70% 0px' });
    document.querySelectorAll('h1[id], h2[id]').forEach(h => obs.observe(h));
  </script>
</body>
</html>
```

---

## Step 5: Full CSS Template

Đây là CSS hoàn chỉnh cần nhúng vào `<style>` — COPY NGUYÊN SI, không rút gọn:

```css
/* ── BEUP Brand Palette ── */
:root {
  --brand:#3fb7b6;--brand-dark:#2d9190;--brand-darker:#1e6b6a;
  --brand-light:#e8f8f8;--brand-border:#a8dedd;
  --blue:#0693e3;--blue-light:#e8f4fd;--blue-border:#93d1f5;
  --green:#00d084;--green-dark:#009e62;--green-light:#e6faf4;--green-border:#80e9c2;
  --yellow:#e5a000;--yellow-light:#fef9e7;--yellow-border:#f7d87a;
  --purple:#9b51e0;--purple-light:#f3ebfd;--purple-border:#d5aaff;
  --red:#cf2e2e;--red-light:#fdf0f0;--red-border:#f0aaaa;
  --orange-light:#fff3e8;--orange-border:#f7c99a;
  --gray:#6b7280;--gray-light:#f7f8fa;--gray-mid:#e8eaed;
  --gray-border:#dde1e7;--text:#1a1a2e;--text-muted:#687080;
  --sidebar-w:268px;--content-max:860px;--radius:10px;
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html{scroll-behavior:smooth;font-size:15px}
body{font-family:'Inter',-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
  line-height:1.75;color:var(--text);background:#fff;
  -webkit-font-smoothing:antialiased;text-rendering:optimizeLegibility;
  word-break:normal;overflow-wrap:break-word;hyphens:none}

/* Layout */
.layout{display:flex;min-height:100vh}

/* Sidebar */
.sidebar{width:var(--sidebar-w);flex-shrink:0;background:#fafbfc;
  border-right:1px solid var(--gray-border);position:sticky;
  top:0;height:100vh;overflow-y:auto;display:flex;flex-direction:column}
.sidebar-header{padding:20px 16px 12px;border-bottom:1px solid var(--gray-border)}
.sidebar-logo{width:100%;max-width:140px;height:auto;display:block}
.sidebar-tagline{font-size:11px;color:var(--text-muted);margin-top:4px;font-weight:500}
.sidebar-toc{padding:16px 16px 0;flex:1}
.sidebar-toc h3{font-size:10px;font-weight:700;text-transform:uppercase;
  letter-spacing:.08em;color:var(--text-muted);margin-bottom:10px}
.sidebar ul{list-style:none}
.sidebar ul li{margin-bottom:1px}
.sidebar ul li a{display:block;padding:5px 10px;border-radius:6px;font-size:12px;
  color:var(--text-muted);text-decoration:none;transition:all .15s;
  word-break:keep-all;hyphens:none;line-height:1.4}
.sidebar ul li a:hover{background:var(--brand-light);color:var(--brand-dark)}
.sidebar ul li a.active{background:var(--brand-light);color:var(--brand-dark);font-weight:600}
.sidebar-footer{padding:16px;margin-top:8px;border-top:1px solid var(--gray-border);
  font-size:11px;color:var(--text-muted);line-height:1.6}
.sidebar-footer a{color:var(--brand-dark);text-decoration:none}

/* Lang Switch */
.lang-switch{display:flex;gap:6px;padding:12px 16px;border-bottom:1px solid var(--gray-border)}
.lang-btn{flex:1;padding:6px 0;border-radius:6px;font-size:12px;font-weight:600;
  text-align:center;text-decoration:none;border:1px solid var(--gray-border);
  color:var(--text-muted);background:#fff;transition:all .15s;cursor:pointer}
.lang-btn.active{background:var(--brand);color:#fff;border-color:var(--brand)}
.lang-btn:hover:not(.active){border-color:var(--brand);color:var(--brand-dark)}

/* Main */
.main{flex:1;min-width:0}

/* Hero */
.hero{background:linear-gradient(140deg,var(--brand-darker) 0%,var(--brand) 55%,#00d084 100%);
  color:#fff;padding:52px 52px 48px;position:relative;overflow:hidden}
.hero-brand{display:flex;align-items:center;gap:12px;margin-bottom:28px}
.hero-logo{height:32px;width:auto;filter:brightness(0) invert(1);opacity:.9}
.hero-pill{padding:5px 12px;border-radius:20px;background:rgba(255,255,255,.18);
  color:#fff;font-size:12px;font-weight:600;text-decoration:none;border:1px solid rgba(255,255,255,.3);
  transition:background .2s}
.hero-pill:hover{background:rgba(255,255,255,.3)}
.hero-title{font-size:clamp(1.6rem,3.5vw,2.4rem);font-weight:800;
  line-height:1.2;margin-bottom:16px;word-break:keep-all}
.hero-sub{font-size:1.05rem;opacity:.92;max-width:640px;line-height:1.6;margin-bottom:20px}
.hero-meta{display:flex;flex-wrap:wrap;gap:10px}
.hero-tag{padding:4px 10px;border-radius:20px;background:rgba(255,255,255,.18);
  font-size:12px;font-weight:500}

/* Content */
.content{max-width:var(--content-max);margin:0 auto;padding:48px 52px 64px}
.content h1{font-size:1.65rem;font-weight:800;margin:48px 0 20px;color:var(--text);
  border-bottom:2px solid var(--brand-light);padding-bottom:10px;word-break:keep-all}
.content h2{font-size:1.25rem;font-weight:700;margin:36px 0 14px;color:var(--text)}
.content h3{font-size:1.05rem;font-weight:600;margin:24px 0 10px;color:var(--text)}
.content p{margin-bottom:14px}
.content a{color:var(--brand-dark);text-decoration:underline;text-underline-offset:2px}
.content a:hover{color:var(--brand)}
.content ul,.content ol{margin:0 0 16px 22px}
.content li{margin-bottom:4px}
.content blockquote{border-left:3px solid var(--brand);background:var(--brand-light);
  padding:12px 16px;border-radius:0 var(--radius) var(--radius) 0;margin:16px 0;
  font-style:italic;color:var(--text-muted)}
.content pre{background:#1a1a2e;color:#e2e8f0;padding:18px 20px;border-radius:var(--radius);
  overflow-x:auto;margin:16px 0;font-family:'JetBrains Mono',monospace;font-size:13px;line-height:1.6}
.content code{font-family:'JetBrains Mono',monospace;font-size:.875em;
  background:#f0f2f5;padding:2px 5px;border-radius:4px}
.content pre code{background:none;padding:0;font-size:inherit}
mark{background:#fef3c7;padding:1px 3px;border-radius:3px}
hr.divider{border:none;border-top:2px solid var(--gray-border);margin:40px 0}

/* Callouts */
.callout{display:flex;gap:12px;padding:16px 18px;border-radius:var(--radius);
  margin:16px 0;border:1px solid}
.callout-icon{font-size:1.25rem;flex-shrink:0;margin-top:1px}
.callout-body{flex:1;min-width:0}
.callout-body p{margin-bottom:8px}
.callout-body p:last-child{margin-bottom:0}
/* Callout Title — BLOCK display, never inline */
.ct{display:block;font-weight:700;margin-bottom:6px}
/* Inline strong — ALWAYS inline, no display override */
strong{font-weight:700}
/* Callout colors */
.callout-blue{background:var(--blue-light);border-color:var(--blue-border);color:#1a4a72}
.callout-green{background:var(--green-light);border-color:var(--green-border);color:#1a5c3a}
.callout-yellow{background:var(--yellow-light);border-color:var(--yellow-border);color:#7a5400}
.callout-purple{background:var(--purple-light);border-color:var(--purple-border);color:#5b2d9e}
.callout-red{background:var(--red-light);border-color:var(--red-border);color:#8b1a1a}
.callout-orange{background:var(--orange-light);border-color:var(--orange-border);color:#8b4a00}
.callout-teal{background:var(--brand-light);border-color:var(--brand-border);color:var(--brand-darker)}
.callout-gray{background:var(--gray-light);border-color:var(--gray-border);color:var(--gray)}

/* Tables */
.table-wrap{overflow-x:auto;margin:20px 0;border-radius:var(--radius);
  border:1px solid var(--gray-border)}
table{width:100%;border-collapse:collapse;font-size:.9rem}
thead{background:var(--brand-light)}
th{padding:10px 14px;text-align:left;font-weight:700;font-size:.8rem;
  text-transform:uppercase;letter-spacing:.04em;color:var(--brand-darker);
  border-bottom:2px solid var(--brand-border)}
td{padding:10px 14px;border-bottom:1px solid var(--gray-border);vertical-align:top}
tr:last-child td{border-bottom:none}
tbody tr:hover{background:var(--gray-light)}

/* Details / Toggle */
details{margin:10px 0;border:1px solid var(--gray-border);
  border-radius:var(--radius);overflow:hidden}
summary{padding:12px 16px;font-weight:600;cursor:pointer;
  background:var(--gray-light);list-style:none;user-select:none}
summary::-webkit-details-marker{display:none}
summary::before{content:"▶ ";font-size:.75em;color:var(--brand)}
details[open] summary::before{content:"▼ "}
.details-body{padding:14px 16px;border-top:1px solid var(--gray-border)}

/* Columns */
.cols{display:grid;grid-template-columns:1fr 1fr;gap:20px;margin:16px 0}
@media(max-width:640px){.cols{grid-template-columns:1fr}}

/* Mermaid */
.mermaid{background:var(--gray-light);border-radius:var(--radius);
  padding:20px;margin:20px 0;overflow-x:auto;text-align:center}

/* Footer */
.site-footer{background:var(--brand-darker);color:rgba(255,255,255,.8);
  padding:32px 52px;text-align:center}
.footer-logo{height:28px;width:auto;filter:brightness(0) invert(1);
  opacity:.8;margin-bottom:12px}
.site-footer p{font-size:13px;margin-bottom:6px}
.site-footer a{color:rgba(255,255,255,.8);text-decoration:none}
.site-footer a:hover{color:#fff}

/* Responsive */
@media(max-width:900px){
  .content{padding:32px 24px 48px}
  .hero{padding:36px 24px 32px}
}
@media(max-width:768px){
  .sidebar{display:none}
  .hero-title{font-size:1.5rem}
}
```

---

## Step 6: Build English Page (if BILINGUAL)

Tạo `$OUT_DIR/en/index.html` với cấu trúc giống hệt, nhưng:
- `<html lang="en">`
- `<h3>Table of Contents</h3>`
- Lang switcher: `🇻🇳 Tiếng Việt` → `../`, `🇬🇧 English` → `#` (active)
- Hero pill: `<a class="hero-pill" href="../">🇻🇳 Tiếng Việt</a>`
- Footer copyright bằng tiếng Anh
- Nội dung từ EN_URL Notion page

---

## Step 7: Generate TOC

Scan tất cả `<h1 id="sN">` trong content → tạo `<li><a href="#sN">N. Title</a></li>` trong sidebar.

Dùng counter: mỗi h1 tăng `n++`, gán `id="s{n}"`.

---

## Step 8: Create README.md

```markdown
# {TITLE}

{DESCRIPTION}

**🌐 Đọc online:** [{pages-url}]({pages-url})
{nếu EN: **🌐 Read in English:** [{pages-url}/en/]({pages-url}/en/)}

---

**Produced by:** 🦫 [BEUP Learning Solutions](http://beup.space)

> © 2026 BEUP Learning Solutions. All rights reserved.
```

---

## Step 9: Setup GitHub Repository

### Xác định repo name
- Nếu `--repo` được cung cấp → dùng nguyên
- Nếu không → slugify từ title: lowercase, replace spaces với `-`, remove special chars

### Kiểm tra repo đã tồn tại chưa
```bash
gh repo view {REPO} 2>/dev/null && echo EXISTS || echo NEW
```

### Tạo repo mới nếu chưa có
```bash
gh repo create {REPO} --public --description "{DESCRIPTION}"
```

### Init & push
```bash
cd $OUT_DIR
git init
git add .
git commit -m "feat: initial publish — {TITLE}"
git remote add origin https://github.com/{REPO}.git
git branch -M main
git push -u origin main
```

### Nếu repo đã tồn tại — chỉ push thay đổi
```bash
cd $OUT_DIR
git add .
git commit -m "feat: update content from Notion"
git push
```

---

## Step 10: Enable GitHub Pages

Dùng GitHub API (curl) — KHÔNG dùng `gh api --field source[branch]=main` vì zsh glob expansion sẽ fail:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $(gh auth token)" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/{REPO}/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

Nếu Pages đã enabled (409 Conflict) → bỏ qua, không phải lỗi.

Sau đó lấy URL:
```bash
curl -s \
  -H "Authorization: Bearer $(gh auth token)" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/{REPO}/pages | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('html_url',''))"
```

---

## Step 11: Quality Checks

Sau khi build, tự kiểm tra:

### Callout Title Check
```bash
grep -n "callout-body.*strong\|<strong>" $OUT_DIR/index.html | head -20
```
Không được có `display:block` trong CSS của `strong`. Nếu có → sửa ngay.

### Text Break Check
Kiểm tra CSS có đủ các rules sau:
```css
body { word-break: normal; overflow-wrap: break-word; hyphens: none; }
.sidebar ul li a { word-break: keep-all; hyphens: none; }
.hero-title { word-break: keep-all; }
```

### Mermaid Check
```bash
grep -c "class=\"mermaid\"" $OUT_DIR/index.html
```
Phải match số lượng mermaid blocks trong Notion.

### Logo Check
```bash
grep -c "beup.space/wp-content/uploads/2025/12/logo-final" $OUT_DIR/index.html
```
Phải >= 3 (sidebar + hero + footer).

---

## Step 12: Output Summary

Báo cáo cho user:

```
✅ Published successfully!

📄 Vietnamese: {pages-url}
🇬🇧 English:   {pages-url}/en/   (nếu bilingual)
📦 Repo:        https://github.com/{REPO}

⚠️  GitHub Pages may take 1-2 min to go live on first deploy.
```

---

## Common Notion Block Mappings Quick Reference

| Notion Block | HTML |
|---|---|
| `paragraph` | `<p>` |
| `heading_1` | `<h1 id="sN">` |
| `heading_2` | `<h2>` |
| `heading_3` | `<h3>` |
| `bulleted_list_item` | `<ul><li>` |
| `numbered_list_item` | `<ol><li>` |
| `to_do` | `<ul class="todo"><li>` với checkbox |
| `callout` | `<div class="callout callout-{color}">` |
| `code` (mermaid) | `<div class="mermaid">` |
| `code` (other) | `<pre><code>` |
| `quote` | `<blockquote>` |
| `toggle` | `<details><summary>` |
| `divider` | `<hr class="divider">` |
| `table` | `<div class="table-wrap"><table>` |
| `column_list` | `<div class="cols">` |
| `image` | `<figure><img><figcaption>` |

## Callout Color Map (Notion → CSS class)

| Notion background | CSS class |
|---|---|
| `blue_background` / `default` | `callout-blue` |
| `green_background` | `callout-green` |
| `yellow_background` / `orange_background` | `callout-yellow` |
| `purple_background` / `pink_background` | `callout-purple` |
| `red_background` | `callout-red` |
| `gray_background` | `callout-gray` |
| `brown_background` | `callout-orange` |
| (teal/brand) | `callout-teal` |
