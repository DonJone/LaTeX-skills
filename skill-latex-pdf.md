---
name: latex-pdf
description: Convert Markdown with LaTeX formulas to PDF using Pandoc + Tectonic (zero-dependency, CJK-ready)
---

# LaTeX PDF Generation Skill

Generate well-formatted PDFs from Markdown files containing LaTeX math formulas, with full Chinese/CJK support.

## Prerequisites Check

Before starting, verify the tools exist or download them:

```bash
# Check or download Pandoc
test -x /tmp/pandoc-3.6.4/bin/pandoc || \
  curl -sL "https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz" | tar xz -C /tmp

# Check or download Tectonic
test -x /tmp/tectonic || \
  curl -sL "https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz" | tar xz -C /tmp

# Check Chinese fonts available
fc-list :lang=zh | grep -i "noto\|wqy" || echo "WARNING: No CJK fonts found"
```

## Header Template

Always create `/tmp/header.tex` before compiling:

```bash
cat > /tmp/header.tex << 'EOF'
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{tabularx}
\usepackage{array}
EOF
```

## Compilation Command

### Standard report (11pt, 2.5cm margins, sans-serif)

```bash
PANDOC=/tmp/pandoc-3.6.4/bin/pandoc
TECTONIC=/tmp/tectonic

"$PANDOC" INPUT.md \
  -f markdown -t latex --standalone \
  -V mainfont="Noto Sans CJK SC" \
  -V sansfont="Noto Sans CJK SC" \
  -V monofont="DejaVu Sans Mono" \
  -V CJKmainfont="Noto Sans CJK SC" \
  -V CJKsansfont="Noto Sans CJK SC" \
  -V CJKmonofont="WenQuanYi Micro Hei" \
  -V geometry:margin=2.5cm \
  -V fontsize=11pt \
  -V linkcolor=blue \
  --metadata title="TITLE" \
  --metadata author="AUTHOR" \
  --metadata date="DATE" \
  --include-in-header=/tmp/header.tex \
  -o /tmp/output.tex

cd /tmp && "$TECTONIC" output.tex
# Produces: /tmp/output.pdf
```

### Appendix / technical docs (10pt, tighter margins)

Same as above but with `-V geometry:margin=2cm -V fontsize=10pt`.

## Font Variable Reference

| Variable | Effect | Required? |
|----------|--------|-----------|
| `mainfont` | Western text body font | Recommended |
| `CJKmainfont` | **Chinese body font** | **MANDATORY for CJK** |
| `CJKsansfont` | Chinese sans-serif | Optional |
| `monofont` | Monospace/code font | Optional |

**CRITICAL**: `-V CJKmainfont` triggers pandoc to auto-load `xeCJK`. Without it, Chinese text cannot line-break properly and will overflow the right margin.

## Common Issues & Fixes

### 1. Chinese text cut off at right margin
**Cause**: Missing `-V CJKmainfont`.
**Fix**: Add `-V CJKmainfont="Noto Sans CJK SC"` (or any available CJK font).

### 2. LaTeX formulas appear as raw text
**Cause**: Formulas in markdown not wrapped in `$...$` or `$$...$$`.
**Fix**: Wrap all LaTeX: `$E=mc^2$` (inline) or `$$\frac{a}{b}$$` (display).

### 3. `Missing $ inserted` error
**Cause**: Underscores `_` or carets `^` outside math mode (e.g., `F_0` in table cells).
**Fix**: Use `$F_0$` to wrap any variable names with TeX special chars.

### 4. Tables too wide for page
**Fix**: Reduce margins (`-V geometry:margin=2cm`) or font size (`-V fontsize=10pt`).

### 5. "Font not found" warnings
Check available fonts: `fc-list :lang=zh`. Fallback options:
- `Noto Sans CJK SC` (recommended, sans-serif)
- `Noto Serif CJK SC` (serif, for academic papers)
- `WenQuanYi Micro Hei` (lightweight fallback)

## Font Selection by Use Case

| Use Case | Recommended | Rationale |
|----------|------------|-----------|
| Business reports | Noto Sans CJK SC | Modern, clean |
| Academic papers | Noto Serif CJK SC | Traditional, formal |
| Tech documentation | Noto Sans CJK SC | Screen-friendly |
| Code blocks | DejaVu Sans Mono | Good Unicode coverage |

## Post-Compilation

After successful compilation, copy the PDF to the project directory:

```bash
cp /tmp/output.pdf ./desired-filename.pdf
```

Clean up intermediate `.tex` and `.aux`/`.log` files if desired.
