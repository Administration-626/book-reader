# Book Reader

一个面向 Codex 的本地读书技能仓库，用于解析 PDF、EPUB、MOBI、TXT 等书籍文件，提取正文内容，并生成结构化读书解释、导读或 Markdown 笔记。

## 功能特性

- 支持 PDF、EPUB、MOBI、TXT、Markdown 书稿解析
- 将书籍内容提取为结构化 JSON，便于后续分批阅读和总结
- 自动过滤目录、版权页、参考文献、空白页等非正文内容
- EPUB 优先按 spine 顺序读取，避免章节文件名自然排序错误
- TXT/Markdown 支持按章节标题或固定行数切分
- 适合生成读书笔记、章节解释、概念表和全书导读

## 项目结构

```text
.
├── SKILL.md                  # Codex 读书技能说明
├── scripts/
│   └── extract_book.py       # 多格式书籍正文提取脚本
└── workspace/                # 本地读书笔记和提取结果目录
```

## 支持格式

| 格式 | 说明 |
| --- | --- |
| PDF | 优先使用 PyMuPDF，缺失时回退到 pypdf |
| EPUB | 优先使用 ebooklib 和 BeautifulSoup，缺失时使用 zip fallback |
| MOBI | 通过 mobi 库提取，必要时建议先转换为 EPUB |
| TXT/MD | 按章节标记或固定行数切分 |

## 安装依赖

脚本包含部分 fallback 逻辑，但为了获得更稳定的提取效果，建议安装可选依赖：

```bash
python3 -m pip install pymupdf pypdf ebooklib mobi beautifulsoup4
```

## 使用方法

提取一本书：

```bash
python3 scripts/extract_book.py <book_path> -o workspace/<book_slug>_extracted.json
```

例如：

```bash
python3 scripts/extract_book.py example.epub -o workspace/example_extracted.json
```

调试时只处理前 20 个页面或章节：

```bash
python3 scripts/extract_book.py <book_path> --max-pages 20
```

查看命令帮助：

```bash
python3 scripts/extract_book.py --help
```

## 输出格式

提取结果保存为 JSON，主要包含：

```json
{
  "metadata": {
    "filename": "example.epub",
    "format": "epub",
    "total_pages": 12,
    "processed_pages": 12,
    "content_pages": 10,
    "skipped_pages": 2
  },
  "pages": [
    {
      "page_number": 1,
      "title": "Chapter 1",
      "text": "..."
    }
  ]
}
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `total_pages` | 原始页面、章节或内容单元数量 |
| `processed_pages` | 实际处理的页面、章节或内容单元数量 |
| `content_pages` | 被识别为正文并保留的数量 |
| `skipped_pages` | 被过滤的目录、版权页、空白页等数量 |
| `pages` | 提取后的正文内容列表 |

## Codex 技能用法

本仓库的核心技能定义在 `SKILL.md`。当用户提出读书、总结、导读、解释、提取知识等请求时，可以按以下流程工作：

1. 确认书籍来源、格式和目标粒度
2. 使用 `scripts/extract_book.py` 提取正文
3. 检查 `metadata` 和前几个内容单元，确认提取质量
4. 按章节或内容块分批理解
5. 生成 Markdown 读书笔记并保存到 `workspace/`

默认笔记建议包含：

- 书籍概览
- 一句话总结
- 全书主线
- 章节解释
- 关键概念表
- 全书总结
- 可继续追问的问题

## 开发与验证

修改 Python 脚本后，至少运行语法检查：

```bash
python3 -m py_compile scripts/extract_book.py
```

如果修改了某种格式的提取逻辑，建议用对应格式的小样本做一次提取验证：

```bash
python3 scripts/extract_book.py <sample_book> --max-pages 20 -o workspace/sample_extracted.json
```

重点检查：

- `processed_pages` 是否符合预期
- `content_pages` 是否明显过少
- `skipped_pages` 是否异常偏高
- 前几个 `pages` 的标题和正文是否像真实章节
- EPUB 章节顺序是否符合目录或正文顺序

## 注意事项

- 不要提交提取后的 JSON、缓存或临时文件
- 不要把大段受版权保护的原文写入仓库
- 扫描版 PDF 可能无法直接提取正文，需要先做 OCR
- MOBI 解析失败时，优先考虑转换为 EPUB 后再处理
- 生成文件建议使用短 slug，例如 `workspace/nietzsche_extracted.json`

