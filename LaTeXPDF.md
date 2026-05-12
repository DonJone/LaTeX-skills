# LaTeX PDF 生成工作流

> 零系统依赖的 Markdown → PDF 方案，支持中文 + LaTeX 数学公式 + 自动表格缩放。

## 1. 工具链获取

两个独立静态二进制文件，无需 root 权限即可部署。

### Pandoc（Markdown → LaTeX 转换）

```bash
# 下载官方静态二进制（当前推荐 3.6.4）
curl -sL "https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz" \
  -o /tmp/pandoc.tar.gz
tar xzf /tmp/pandoc.tar.gz -C /tmp
export PANDOC=/tmp/pandoc-3.6.4/bin/pandoc
```

### Tectonic（LaTeX → PDF 编译）

Tectonic 是 XeTeX 引擎的现代替代品，自动按需下载 LaTeX 宏包，无需安装 texlive（几个 GB）。

```bash
curl -sL "https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz" \
  -o /tmp/tectonic.tar.gz
tar xzf /tmp/tectonic.tar.gz -C /tmp
export TECTONIC=/tmp/tectonic
```

### 系统字体要求

需要中文字体。检查命令：

```bash
fc-list :lang=zh | grep -i "noto\|wqy"
```

常见可用字体：
- `Noto Sans CJK SC`（推荐，无衬线）
- `Noto Serif CJK SC`（衬线）
- `WenQuanYi Micro Hei`（轻量备选）

## 2. LaTeX Header 模板

保存为 `header.tex`，通过 `--include-in-header` 引入：

```latex
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{tabularx}
\usepackage{array}
```

> **要点**：不要在此文件中设置字体——字体会通过 Pandoc 的 `-V` 变量传入，由 Pandoc 自动生成 `\setmainfont`、`\setCJKmainfont` 等命令。

## 3. 标准编译命令

### 主报告（11pt, 2.5cm 边距）

```bash
"$PANDOC" 主报告.md \
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
  --metadata title="报告标题" \
  --metadata author="作者名" \
  --metadata date="2026-05-10" \
  --include-in-header=header.tex \
  -o /tmp/report.tex

cd /tmp && "$TECTONIC" report.tex
```

### 附录/技术文档（10pt, 2cm 边距，表格多时用更小字号）

```bash
"$PANDOC" 技术附录.md \
  -f markdown -t latex --standalone \
  -V mainfont="Noto Sans CJK SC" \
  -V sansfont="Noto Sans CJK SC" \
  -V monofont="DejaVu Sans Mono" \
  -V CJKmainfont="Noto Sans CJK SC" \
  -V CJKsansfont="Noto Sans CJK SC" \
  -V CJKmonofont="WenQuanYi Micro Hei" \
  -V geometry:margin=2cm \
  -V fontsize=10pt \
  --metadata title="技术附录" \
  --include-in-header=header.tex \
  -o /tmp/appendix.tex

cd /tmp && "$TECTONIC" appendix.tex
```

## 4. 字体变量说明

| 变量 | 作用 | 触发效果 |
|------|------|---------|
| `mainfont` | 西文正文 | `\setmainfont`（fontspec） |
| `sansfont` | 西文无衬线 | `\setsansfont`（fontspec） |
| `monofont` | 等宽代码 | `\setmonofont`（fontspec） |
| `CJKmainfont` | **中文正文** | **触发 `\usepackage{xeCJK}`** + `\setCJKmainfont` |
| `CJKsansfont` | 中文无衬线 | `\setCJKsansfont` |
| `CJKmonofont` | 中文等宽 | `\setCJKmonofont` |

> **关键**：设置 `CJKmainfont` 是必须的——它触发 Pandoc 自动加载 `xeCJK` 包。缺少 `xeCJK` 会导致中文无法换行，文字溢出右侧边界。

## 5. 常见问题排查

### 5.1 中文文字被裁切

**现象**：PDF 右侧文字截断，句子不完整。

**原因**：未设置 `-V CJKmainfont`，导致 `xeCJK` 未加载，中文汉字间无合法断行点。

**修复**：确保 `-V CJKmainfont="<任意可用中文字体>"` 已设置。

### 5.2 LaTeX 公式不渲染

**现象**：`\frac`、`\sum`、`\cdot` 等命令显示为原始文本。

**原因**：Markdown 原文中的 LaTeX 未用 `$...$` 或 `$$...$$` 包裹，Pandoc 将其视为普通文本转义输出，`\f`、`\s` 等命令在 LaTeX 文本模式中非法。

**修复**：在 Markdown 中把所有 LaTeX 公式用 `$`（行内）或 `$$`（块级）包裹。

```markdown
# 错误
F(d) = F_0 \cdot e^{-\ln(2) \cdot d / d_{1/2}}

# 正确
$$F(d) = F_0 \cdot e^{-\ln(2) \cdot d / d_{1/2}}$$
```

### 5.3 表格过宽溢出

**现象**：编译警告 `Overfull \hbox (XXXpt too wide)`，表格超出页面。

**轻量修复**：
- 缩小页边距（`geometry:margin=2cm`）
- 减小基础字号（`fontsize=10pt`）
- Header 中已有 `longtable` + `tabularx`，可进一步添加 `\footnotesize`

**重度修复**（如果上述不够）：创建 Lua 过滤器将 `longtable` 替换为 `tabular + adjustbox{max width=\textwidth}`。注意：`adjustbox` 内的表格无法跨页，仅适用于短表。

### 5.4 `Missing $ inserted` 错误

**原因**：Markdown 原文中 `_`、`^` 等 TeX 特殊字符出现在非数学模式中（如表格内的 `N_{metro}`）。

**Pandoc 行为**：Pandoc 在非数学模式中会将 `_` 转义为 `\_`，但若 `_` 出现在 `\text` 等 LaTeX 命令内部则不会被转义，导致 TeX 引擎报错。

**修复**：确保所有含 `_`、`^`、`\frac` 等 TeX 命令的文本都在 `$...$` 内。

## 6. 完整实例

本项目使用的完整命令历史，从零依赖到 PDF 输出：

```bash
# === 1. 获取工具 ===
curl -sL "https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz" | tar xz -C /tmp
curl -sL "https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz" | tar xz -C /tmp

PANDOC=/tmp/pandoc-3.6.4/bin/pandoc
TECTONIC=/tmp/tectonic

# === 2. 写 header ===
cat > /tmp/header.tex << 'EOF'
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{tabularx}
\usepackage{array}
EOF

# === 3. 编译主报告 ===
"$PANDOC" 主报告.md \
  -f markdown -t latex --standalone \
  -V mainfont="Noto Sans CJK SC" \
  -V sansfont="Noto Sans CJK SC" \
  -V monofont="DejaVu Sans Mono" \
  -V CJKmainfont="Noto Sans CJK SC" \
  -V CJKsansfont="Noto Sans CJK SC" \
  -V CJKmonofont="WenQuanYi Micro Hei" \
  -V geometry:margin=2.5cm -V fontsize=11pt -V linkcolor=blue \
  --metadata title="报告标题" \
  --metadata author="作者" \
  --metadata date="2026-05-10" \
  --include-in-header=/tmp/header.tex \
  -o /tmp/report.tex

cd /tmp && "$TECTONIC" report.tex
# 输出: report.pdf
```

## 7. 字体选择指南

| 场景 | 推荐字体 | 说明 |
|------|---------|------|
| 学术论文 | Noto Serif CJK SC | 衬线，正式感强 |
| 商业报告 | Noto Sans CJK SC | 无衬线，现代简洁 |
| 技术文档 | Noto Sans CJK SC | 无衬线，屏幕阅读友好 |
| 轻量备选 | WenQuanYi Micro Hei | 覆盖完整但只有一种字重 |
| 西文等宽 | DejaVu Sans Mono | 代码块用 |

## 8. Pandoc Markdown 中 LaTeX 公式书写规范

**行内公式**：`$E = mc^2$`

**块级公式**：
```markdown
$$
P_{ij} = \frac{S_j / T_{ij}^{\lambda}}{\sum_{k} S_k / T_{ik}^{\lambda}}
$$
```

**表格内的数学符号**：含 `_`、`^` 的变量名必须包裹在 `$...$` 内。

```markdown
| **POI 类型** | **$F_0$ (人/单位)** | **$d_{1/2}$ (km)** |
|-------------|:-------------------:|:------------------:|
| 地铁站 | 10,000 | 0.8 |
```

**常见 LaTeX 命令快查**：

| 命令 | 输出 | 用途 |
|------|------|------|
| `\frac{a}{b}` | 分数 | 比值、概率 |
| `\sqrt{x}` | 根号 | Reilly 断裂点 |
| `\cdot` | 乘法点 | 乘积公式 |
| `\ln`, `\log` | 对数 | 距离衰减 |
| `\max`, `\min` | 最大/最小值 | Huff 效用函数 |
| `\sum` | 求和 | 概率标准化 |
| `\text{abc}` | 公式中插入文本 | 带单位的数值 |
| `\lambda` | λ | Huff 摩擦系数 |
| `\to` | → | 趋近符号 |
| `\times` | × | 乘法 |
