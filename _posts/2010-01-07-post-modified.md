---
title: "Agent Skills là gì? Khả năng mở rộng mạnh mẽ của Claude"
date: 2026-03-27T09:00:00+07:00
categories:
  - AI
tags:
  - claude
  - agent-skills
  - anthropic
  - deep-dive
---

Khi làm việc với Claude ngày càng nhiều, tôi bắt đầu thắc mắc: làm thế nào để Claude có thể xử lý các tác vụ chuyên biệt như tạo file PowerPoint hay đọc PDF mà không cần lập trình thêm gì? Câu trả lời nằm ở **Agent Skills** — một cơ chế mở rộng khả năng của Claude theo từng domain cụ thể.

Bài này ghi lại những gì tôi học được từ [tài liệu chính thức của Anthropic](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

## Agent Skills là gì?

Agent Skills là các **module khả năng có thể tái sử dụng**, đóng gói workflow, context và best practices cho một domain cụ thể. Nếu prompt thông thường là hướng dẫn cho một tác vụ đơn lẻ, thì Skill là kiến thức chuyên sâu được nạp **tự động, đúng lúc** mỗi khi cần.

Ba lợi ích chính:
- **Chuyên hoá Claude** cho từng domain: viết code, xử lý PDF, tạo báo cáo Excel...
- **Không lặp lại hướng dẫn** mỗi lần chat — tạo một lần, dùng mãi
- **Kết hợp nhiều Skill** để xây dựng workflow phức tạp

## Cách Skills hoạt động — 3 tầng nạp dữ liệu

Điều thú vị nhất về Skills là kiến trúc **progressive disclosure** — Claude không nạp toàn bộ nội dung Skill vào context ngay từ đầu, mà nạp theo từng tầng khi cần.

### Tầng 1: Metadata (luôn nạp khi khởi động)

Mỗi Skill có một file `SKILL.md` với YAML frontmatter:

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

Chỉ phần `name` và `description` này được nạp vào system prompt khi Claude khởi động. Chi phí rất nhỏ (~100 token/Skill), nên bạn có thể cài nhiều Skills mà không lo tốn context.

### Tầng 2: Instructions (nạp khi Skill được kích hoạt)

Khi request của bạn khớp với `description` của một Skill, Claude đọc toàn bộ thân file `SKILL.md` qua lệnh bash:

```bash
bash: read pdf-skill/SKILL.md
```

Phần thân này chứa workflow, best practices, ví dụ code — dưới 5k token. Chỉ đến lúc này nội dung mới vào context window.

### Tầng 3: Resources và code (nạp khi được tham chiếu)

Một Skill có thể đi kèm nhiều file phụ:

```
pdf-skill/
├── SKILL.md          ← instructions chính
├── FORMS.md          ← hướng dẫn điền form
├── REFERENCE.md      ← API reference chi tiết
└── scripts/
    └── fill_form.py  ← script thực thi
```

Claude chỉ đọc `FORMS.md` nếu task liên quan đến form. Khi chạy `fill_form.py`, **code của script không vào context** — chỉ có output (kết quả) mới được nhìn thấy. Điều này giúp Skills có thể đóng gói tài liệu, dataset, ví dụ lớn tùy ý mà không tốn token.

## Ví dụ thực tế: Skill xử lý PDF

```
1. Khởi động  → System prompt: "PDF Processing — Extract text and tables..."
2. User nhắn  → "Extract text from this PDF and summarize it"
3. Claude đọc → bash: read pdf-skill/SKILL.md  (instructions vào context)
4. Nhận định  → Task không cần fill form → KHÔNG đọc FORMS.md
5. Thực thi   → Dùng instructions để hoàn thành tác vụ
```

## Skills có ở đâu?

| Platform | Loại Skill hỗ trợ |
|---|---|
| **Claude.ai** | Pre-built + Custom (upload zip, plan Pro/Max/Team/Enterprise) |
| **Claude API** | Pre-built + Custom (upload qua `/v1/skills`, chia sẻ cả workspace) |
| **Claude Code** | Custom only (filesystem-based, tự động phát hiện) |
| **Agent SDK** | Custom only (đặt trong `.claude/skills/`) |

### Pre-built Skills có sẵn

Anthropic cung cấp 4 Skills dựng sẵn dùng được ngay:
- **PowerPoint** (`pptx`) — tạo và chỉnh sửa slides
- **Excel** (`xlsx`) — phân tích dữ liệu, tạo chart
- **Word** (`docx`) — tạo và format tài liệu
- **PDF** (`pdf`) — tạo file PDF có định dạng

## Viết Custom Skill

Cấu trúc tối thiểu của một Skill:

```markdown
---
name: my-skill-name
description: Mô tả ngắn gọn Skill làm gì và khi nào dùng nó.
---

# My Skill Name

## Instructions
Hướng dẫn step-by-step cho Claude.

## Examples
Ví dụ cụ thể về cách dùng Skill này.
```

Quy tắc đặt tên:
- `name`: tối đa 64 ký tự, chỉ gồm chữ thường, số, dấu gạch ngang
- `description`: tối đa 1024 ký tự — **đây là bề mặt khám phá**, viết rõ *khi nào* nên dùng Skill

## Lưu ý quan trọng

**Skills không đồng bộ giữa các platform.** Nếu upload Skill lên Claude.ai, nó không tự có mặt trên API và ngược lại. Bạn phải quản lý riêng từng nơi.

**Chỉ dùng Skills từ nguồn tin cậy.** Skills chạy trong môi trường có quyền truy cập filesystem và bash — một Skill độc hại có thể đọc dữ liệu nhạy cảm hoặc thực thi lệnh không mong muốn. Hãy audit kỹ trước khi dùng bất kỳ Skill nào từ bên ngoài.

## Tài nguyên thêm

- [Quickstart: dùng Skills với Claude API](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart)
- [Authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Skills Cookbook (notebook examples)](https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction)
- [Dùng Skills trong Claude Code](https://code.claude.com/docs/en/skills)