---
title: "Agent Skills Best Practices (Phần 1): Viết Skill ngắn gọn, đúng cấu trúc"
date: 2026-03-27T10:00:00+07:00
categories:
  - AI
tags:
  - claude
  - agent-skills
  - anthropic
  - best-practices
---

Sau khi hiểu [Agent Skills là gì và cách hoạt động](/ai/agent-skills-la-gi-kha-nang-mo-rong-manh-me-cua-claude/), câu hỏi tiếp theo là: **làm thế nào để viết một Skill tốt?**

Bài này tổng hợp từ [tài liệu best practices chính thức của Anthropic](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), tập trung vào phần nền tảng: nguyên tắc cốt lõi, đặt tên, viết description, và tổ chức nội dung theo progressive disclosure.

## Nguyên tắc 1: Ngắn gọn là vua

Context window là tài nguyên chung — Skill của bạn chia sẻ nó với system prompt, lịch sử hội thoại, metadata của các Skill khác, và nội dung request thực tế.

**Giả định mặc định: Claude đã rất thông minh.** Chỉ thêm context mà Claude thực sự chưa có. Trước mỗi đoạn, tự hỏi:

- "Claude có thực sự cần giải thích này không?"
- "Tôi có thể giả định Claude đã biết điều này không?"
- "Đoạn này có xứng đáng với chi phí token không?"

**Ví dụ thực tế:**

```markdown
# Tốt (~50 token):
## Extract PDF text
Use pdfplumber for text extraction:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

# Tệ (~150 token):
## Extract PDF text
PDF (Portable Document Format) là định dạng file phổ biến chứa text, ảnh...
Để extract text, bạn cần dùng một thư viện. Có nhiều thư viện,
nhưng pdfplumber được khuyến nghị vì dễ dùng...
```

Phiên bản ngắn giả định Claude biết PDF là gì và cách import thư viện Python — điều hiển nhiên với một AI mạnh.

## Nguyên tắc 2: Mức độ tự do phù hợp với tác vụ

Không phải lúc nào cũng nên ra lệnh cụ thể hoặc để tự do hoàn toàn. Hãy chọn dựa trên độ nguy hiểm của tác vụ:

| Mức độ | Dùng khi | Ví dụ |
|---|---|---|
| **Tự do cao** (hướng dẫn văn bản) | Nhiều cách tiếp cận đều hợp lệ, phụ thuộc context | Code review |
| **Tự do vừa** (pseudocode/script có tham số) | Có pattern ưa thích, nhưng cho phép biến tấu | Tạo report |
| **Tự do thấp** (lệnh cụ thể, ít/không có tham số) | Tác vụ dễ sai, cần thực hiện đúng thứ tự | Database migration |

Hình ảnh dễ nhớ: **Claude như robot đi trên đường**.
- *Cầu hẹp có vách đá hai bên* → Chỉ có một con đường an toàn → Cần guardrail cụ thể (tự do thấp)
- *Cánh đồng trống* → Nhiều đường đều dẫn đến đích → Cho hướng chung, tin Claude tự tìm đường (tự do cao)

## Cấu trúc Skill: Đặt tên

Quy tắc kỹ thuật cho trường `name`:
- Tối đa 64 ký tự
- Chỉ gồm chữ thường, số, dấu gạch ngang
- Không chứa XML tags
- Không dùng từ khóa reserved: `anthropic`, `claude`

**Nên dùng dạng gerund (verb + -ing)** — mô tả rõ hoạt động:

```
✓ processing-pdfs
✓ analyzing-spreadsheets
✓ managing-databases
✓ writing-documentation
```

**Chấp nhận được:**
- Noun phrase: `pdf-processing`, `spreadsheet-analysis`
- Action-oriented: `process-pdfs`, `analyze-spreadsheets`

**Tránh:**
- Mơ hồ: `helper`, `utils`, `tools`
- Quá chung chung: `documents`, `data`, `files`

Tên nhất quán giúp dễ tra cứu, hiểu chức năng ngay, và xây dựng thư viện Skill chuyên nghiệp.

## Cấu trúc Skill: Viết description hiệu quả

`description` là bề mặt khám phá — Claude dùng nó để quyết định **có kích hoạt Skill hay không** trong số hàng trăm Skill có thể có. Đây là trường quan trọng nhất.

**Luôn viết ở ngôi thứ ba.** Description được inject vào system prompt; viết ngôi một gây lỗi khám phá:

```
✓ "Processes Excel files and generates reports"
✗ "I can help you process Excel files"
✗ "You can use this to process Excel files"
```

**Bao gồm cả *cái gì* lẫn *khi nào*:**

```yaml
# Tốt — cụ thể, có trigger rõ ràng:
description: >
  Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDF files or when the user mentions PDFs,
  forms, or document extraction.

# Tốt — Excel:
description: >
  Analyze Excel spreadsheets, create pivot tables, generate charts.
  Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.

# Tệ — quá mơ hồ:
description: "Helps with documents"
description: "Processes data"
```

## Cấu trúc Skill: Progressive disclosure

`SKILL.md` là bản tóm tắt, như mục lục của một tài liệu onboarding — nó chỉ Claude đến thông tin chi tiết khi cần, không dump tất cả vào một chỗ.

**Giới hạn thực hành:** giữ body của `SKILL.md` dưới 500 dòng. Nếu vượt, tách sang file riêng.

### Pattern 1: Hướng dẫn tổng quan + links

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features
**Form filling**: See [FORMS.md](FORMS.md)
**API reference**: See [REFERENCE.md](REFERENCE.md)
**Examples**: See [EXAMPLES.md](EXAMPLES.md)
```

Claude chỉ đọc `FORMS.md` khi task liên quan đến form — token không bị tiêu khi không cần.

### Pattern 2: Tổ chức theo domain

Khi Skill có nhiều lĩnh vực, tổ chức từng domain trong file riêng:

```
bigquery-skill/
├── SKILL.md          ← tổng quan, điều hướng
└── reference/
    ├── finance.md    ← metrics doanh thu, billing
    ├── sales.md      ← pipeline, cơ hội
    ├── product.md    ← API usage, tính năng
    └── marketing.md  ← chiến dịch, attribution
```

Khi user hỏi về doanh thu, Claude chỉ đọc `finance.md` — ba file còn lại không tiêu token nào.

### Pattern 3: Nội dung có điều kiện

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents
For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

### Tránh nested references quá sâu

Claude có thể đọc file không hoàn chỉnh khi gặp tham chiếu lồng nhau quá sâu (dùng `head -100` để preview thay vì đọc toàn bộ).

```markdown
# Tệ — quá sâu:
# SKILL.md → advanced.md → details.md → thông tin thực tế

# Tốt — một cấp từ SKILL.md:
# SKILL.md
**Basic usage**: [hướng dẫn trong SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
```

### Table of contents cho file dài

File reference dài hơn 100 dòng nên có mục lục ở đầu, giúp Claude nắm toàn cảnh ngay cả khi chỉ đọc một phần:

```markdown
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...
```

## Kiểm thử với nhiều model

Skills hoạt động khác nhau trên từng model — hãy test với tất cả model bạn định dùng:

| Model | Câu hỏi cần kiểm tra |
|---|---|
| **Haiku** (nhanh, tiết kiệm) | Skill có cung cấp đủ hướng dẫn không? |
| **Sonnet** (cân bằng) | Skill có rõ ràng và hiệu quả không? |
| **Opus** (reasoning mạnh) | Skill có giải thích thừa không? |

Điều hoàn hảo với Opus có thể cần thêm chi tiết cho Haiku. Nếu dùng nhiều model, hướng đến instructions hoạt động tốt với tất cả.

---

Phần tiếp theo sẽ đi vào **workflows, feedback loops, vòng lặp đánh giá & cải tiến, và các anti-pattern cần tránh** — [đọc Phần 2 tại đây](/ai/agent-skills-best-practices-phan-2-workflow-feedback-loop-va-vong-lap-cai-tien/).
