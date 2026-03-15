# notion-to-pages

> Claude Code skill: chuyển Notion page thành static site BEUP-branded trên GitHub Pages — song ngữ VI/EN, Mermaid, callouts, tables — chỉ 1 lệnh.

**Tiết kiệm:** ~2 giờ/lần so với build thủ công

---

## Vấn đề giải quyết

Trước khi có skill này:
- Fetch Notion content → parse thủ công từng block → mất 30–60 phút
- HTML/CSS branding BEUP phải viết lại từ đầu mỗi lần
- Lỗi text broken line trong callout, sidebar link ngắt sai thường xuyên xảy ra
- GitHub Pages cần enable thủ công qua API (dễ bị lỗi zsh glob expansion)
- Song ngữ VI/EN + language switcher phải tự ghép bằng tay

Với skill này:
- Cái trước mất 2 giờ → giờ mất < 2 phút
- BEUP branding, logo, màu sắc tự động áp dụng
- Language switcher tự sinh khi có `--en`
- Mọi lỗi known đã được bake-in và fix sẵn

---

## Cài đặt

**Tổng thời gian: ~1 phút**

**Bước 1** (~1 phút) — Copy skill vào Claude

```bash
cp -r notion-to-pages ~/.claude/skills/
```

Hoặc clone thẳng vào skills:

```bash
git clone https://github.com/beupspace-tool/notion-to-pages ~/.claude/skills/notion-to-pages
```

**Bước 2** — Không cần cài thêm gì. Dùng ngay.

---

## Cách dùng

### Cách nhanh nhất
```bash
/notion-to-pages --url https://www.notion.so/your-page-id
```

### Đầy đủ options
```bash
/notion-to-pages \
  --url <vi-notion-url> \    # Notion page chính (VI hoặc default)
  --en <en-notion-url> \     # Trang tiếng Anh (optional — tạo language switcher)
  --repo owner/repo-name \   # GitHub repo target (default: auto từ title)
  --title "Tên trang"        # Override title (optional)
```

### Ví dụ thực tế
```bash
# Chỉ 1 trang tiếng Việt
/notion-to-pages --url https://www.notion.so/f0fb7bb9ccfb4523b7dc3c3b1d61d16b

# Song ngữ VI + EN — tự thêm language switcher
/notion-to-pages \
  --url https://www.notion.so/vi-page \
  --en https://beupspace.notion.site/en-page

# Push vào repo sẵn có
/notion-to-pages --url https://notion.so/xxx --repo beupspace-tool/my-guide
```

---

## Output

```
/tmp/notion-pages-output/
├── index.html      ← Trang chính (VI / default), BEUP branded
├── en/
│   └── index.html  ← Trang tiếng Anh (nếu có --en)
└── README.md       ← README với link GitHub Pages
```

Tự động push lên `https://github.com/{repo}` và enable GitHub Pages.

---

## Những gì đã fix sẵn trong skill

| Vấn đề | Fix |
|--------|-----|
| Text broken trong callout | `.ct` class cho tiêu đề, `strong` luôn inline |
| Tiếng Việt ngắt sai | `word-break: normal; hyphens: none` trên body |
| Sidebar link bị chặt đôi | `word-break: keep-all` trên nav links |
| Mermaid diagram | Auto-detect → `<div class="mermaid">` |
| GitHub Pages API (zsh glob bug) | `curl` + JSON body, không dùng `--field` |
| Logo BEUP | Tự đặt ở sidebar + hero + footer |
| Language switcher | Tự sinh nếu có `--en` |

---

## Giới hạn

| Tình huống | Kết quả |
|-----------|---------|
| Notion page public / shared | ✅ Hoạt động tốt |
| Notion page private (cần token) | ⚠️ Cần Notion MCP đã auth |
| Nhiều hơn 2 ngôn ngữ | ❌ Hiện chỉ hỗ trợ VI + EN |
| Notion database view | ⚠️ Chỉ lấy được content page, không phải table data |

---

## Liên quan

- Dùng với Notion MCP (`notion-fetch`) đã được cấu hình trong Claude Code
- Command: `/notion-to-pages` trong Claude Code
- Skill repo: [beupspace-tool/notion-to-pages](https://github.com/beupspace-tool/notion-to-pages)

---

**Produced by:** 🦫 [BEUP Learning Solutions](http://beup.space)
