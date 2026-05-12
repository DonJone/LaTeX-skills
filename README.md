# LaTeX PDF Skills

零系统依赖的 Markdown → PDF 工具链，专为中文 + LaTeX 数学公式场景设计。

## 适用场景

- 学术论文、商业报告、技术文档中的 LaTeX 公式需要渲染为 PDF
- 无法安装 texlive（几个 GB）或没有 root 权限的环境
- 需要批量将 Markdown 转为排版良好的 PDF（含 CJK 中文支持）

## 工具链

| 工具 | 用途 | 大小 |
|------|------|------|
| [Pandoc](https://github.com/jgm/pandoc) | Markdown → LaTeX 转换 | ~50MB 静态二进制 |
| [Tectonic](https://github.com/tectonic-typesetting/tectonic) | LaTeX → PDF 编译（XeTeX 引擎） | ~57MB 静态二进制 |

两个工具均为独立静态二进制，无需安装任何依赖，下载即用。

## 快速开始

```bash
# 1. 获取工具
curl -sL "https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz" | tar xz -C /tmp
curl -sL "https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz" | tar xz -C /tmp

PANDOC=/tmp/pandoc-3.6.4/bin/pandoc
TECTONIC=/tmp/tectonic

# 2. 编译
"$PANDOC" report.md -f markdown -t latex --standalone \
  -V mainfont="Noto Sans CJK SC" -V CJKmainfont="Noto Sans CJK SC" \
  -V geometry:margin=2.5cm -V fontsize=11pt \
  --metadata title="标题" --metadata author="作者" \
  -o /tmp/report.tex

cd /tmp && "$TECTONIC" report.tex
# 输出: report.pdf
```

## 核心要点

- **`-V CJKmainfont`** 是中文 PDF 的生命线——Pandoc 靠它触发 `xeCJK` 包，缺少则中文无法换行
- Markdown 中所有 LaTeX 公式必须用 `$...$`（行内）或 `$$...$$`（块级）包裹
- 系统需要中文字体（Noto Sans CJK SC / WenQuanYi Micro Hei 等）

## 文件说明

| 文件 | 内容 |
|------|------|
| `LaTeXPDF.md` | 完整工作流参考（工具获取、编译命令、常见问题排查、字体指南） |
| `skill-latex-pdf.md` | Claude Code Skill 格式，可被 AI 助手直接调用 |

## 参考

- Pandoc: https://pandoc.org
- Tectonic: https://tectonic-typesetting.github.io
