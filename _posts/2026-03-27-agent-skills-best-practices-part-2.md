---
title: "Agent Skills Best Practices (Phần 2): Workflow, Feedback Loop và Vòng lặp Cải tiến"
date: 2026-03-27T11:00:00+07:00
categories:
  - AI
tags:
  - claude
  - agent-skills
  - anthropic
  - best-practices
---

[Phần 1](/ai/agent-skills-best-practices-phan-1-viet-skill-ngan-gon-dung-cau-truc/) đã đề cập nguyên tắc viết Skill ngắn gọn và tổ chức cấu trúc. Phần này tập trung vào **vận hành**: thiết kế workflow, tạo feedback loop, đánh giá và cải tiến Skill theo thời gian — và những anti-pattern cần tránh.

Nguồn: [tài liệu best practices chính thức của Anthropic](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

## Thiết kế Workflow cho tác vụ phức tạp

Với tác vụ nhiều bước, hãy phân rã thành các bước rõ ràng và cung cấp checklist để Claude (và bạn) theo dõi tiến trình.

### Ví dụ: Workflow nghiên cứu và tổng hợp (không cần code)

```markdown
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**
Review each document in the `sources/` directory.
Note the main arguments and supporting evidence.

**Step 2: Identify key themes**
Look for patterns across sources. Where do sources agree or disagree?
...
```

### Ví dụ: Workflow có code (PDF form filling)

```markdown
## PDF form filling workflow

Copy this checklist:
```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1:** `python scripts/analyze_form.py input.pdf`
**Step 2:** Edit `fields.json` to add values for each field.
**Step 3:** `python scripts/validate_fields.py fields.json` — fix errors before continuing.
**Step 4:** `python scripts/fill_form.py input.pdf fields.json output.pdf`
**Step 5:** `python scripts/verify_output.py output.pdf` — if fails, return to Step 2.
```

> Checklist rõ ràng ngăn Claude bỏ qua bước validate quan trọng, và giúp bạn biết đang ở đâu trong quá trình.

## Feedback Loop — Pattern cải thiện chất lượng đầu ra

Pattern phổ biến nhất: **chạy validator → sửa lỗi → lặp lại**. Cực kỳ hiệu quả cho các tác vụ yêu cầu chính xác cao.

### Ví dụ: Feedback loop với tài liệu tham chiếu (không code)

```markdown
## Content review process

1. Draft content following guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
```

Ở đây "validator" là `STYLE_GUIDE.md` — Claude tự so sánh, không cần script.

### Ví dụ: Feedback loop với code

```markdown
## Document editing process

1. Make edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
```

Nguyên tắc "validate trước khi tiếp tục" bắt lỗi sớm trước khi ảnh hưởng đến file gốc.

## Tránh thông tin lỗi thời

```markdown
# Tệ — sẽ trở nên sai:
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.

# Tốt — dùng section "old patterns":
## Current method
Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns
<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>
The v1 API used: `api.example.com/v1/messages`
This endpoint is no longer supported.
</details>
```

## Đánh giá và Cải tiến — Vòng lặp thực tế

### Xây dựng evaluations TRƯỚC khi viết documentation

Đây là điểm nhiều người bỏ qua: hãy tạo test scenarios trước, để đảm bảo Skill giải quyết vấn đề thực tế thay vì vấn đề tưởng tượng.

**Quy trình evaluation-driven development:**

1. **Xác định gaps:** Chạy Claude trên các tác vụ mẫu *không có* Skill. Ghi lại điểm fail cụ thể
2. **Tạo evaluations:** Tạo 3 scenarios test những gaps đó
3. **Đo baseline:** Đo performance của Claude *không có* Skill
4. **Viết instructions tối thiểu:** Chỉ đủ để pass các evaluations
5. **Lặp:** Chạy evaluations, so với baseline, tinh chỉnh

Cấu trúc evaluation:

```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF using an appropriate library",
    "Extracts text from all pages without missing any",
    "Saves extracted text to output.txt in a readable format"
  ]
}
```

### Phát triển Skill với chính Claude

Cách hiệu quả nhất: dùng **hai phiên Claude** — một để tạo/cải tiến Skill (Claude A), một để test Skill trong tác vụ thực tế (Claude B).

**Tạo Skill mới — 7 bước:**

1. **Hoàn thành một tác vụ không có Skill:** Làm việc với Claude A qua prompting thông thường. Chú ý những context bạn liên tục cung cấp thủ công
2. **Nhận ra pattern tái sử dụng:** Sau khi xong, tự hỏi: "Context nào tôi cung cấp sẽ hữu ích cho tác vụ tương tự trong tương lai?"
3. **Nhờ Claude A tạo Skill:** *"Tạo một Skill dựa trên pattern BigQuery chúng ta vừa dùng. Bao gồm table schemas, naming conventions, và quy tắc lọc test accounts."*
4. **Review tính ngắn gọn:** Nhờ Claude A loại bỏ giải thích thừa: *"Bỏ phần giải thích win rate là gì — Claude đã biết điều đó rồi."*
5. **Cải thiện information architecture:** *"Tổ chức lại để table schema nằm trong file reference riêng — có thể thêm bảng mới sau."*
6. **Test với Claude B:** Chạy Skill trên Claude mới (fresh instance) với các tác vụ liên quan. Quan sát Claude B tìm đúng thông tin, áp dụng đúng rule, hoàn thành tác vụ không?
7. **Lặp dựa trên quan sát:** *"Khi dùng Skill này, Claude quên lọc theo ngày cho Q4. Nên thêm section về date filtering patterns không?"*

**Cải tiến Skill hiện có — vòng lặp liên tục:**

```
Dùng Skill trong công việc thực tế (Claude B)
  ↓
Quan sát behavior: Claude bỏ sót gì? Làm sai chỗ nào?
  ↓
Chia sẻ observation với Claude A + SKILL.md hiện tại
  ↓
Claude A đề xuất: "Dùng 'MUST filter' thay vì 'always filter'? Hay thêm vào đầu workflow?"
  ↓
Áp dụng thay đổi → test lại với Claude B
  ↓
Lặp lại theo từng scenario mới
```

> **Tại sao cách này hiệu quả:** Claude A hiểu nhu cầu của agent, bạn cung cấp domain expertise, Claude B để lộ gap qua usage thực tế, và cải tiến dựa trên behavior quan sát được — không phải giả định.

### Quan sát cách Claude điều hướng Skills

Khi iterate, chú ý:

| Dấu hiệu | Có thể có nghĩa |
|---|---|
| Claude đọc files theo thứ tự không ngờ | Cấu trúc không trực quan như bạn nghĩ |
| Claude không follow reference đến file quan trọng | Link cần hiển thị hơn / rõ ràng hơn |
| Claude liên tục đọc cùng một file | Nội dung đó nên nằm trong SKILL.md chính |
| Claude không bao giờ truy cập một file bundled | File đó có thể không cần thiết, hoặc chưa được signal đúng |

`name` và `description` trong frontmatter đặc biệt quan trọng — chúng quyết định việc Skill có được chọn hay không. Nếu Skill không kích hoạt đúng lúc, đây là nơi đầu tiên cần kiểm tra.

## Anti-patterns cần tránh

### Dùng Windows-style paths

```
✓ scripts/helper.py
✓ reference/guide.md

✗ scripts\helper.py
✗ reference\guide.md
```

Unix-style paths hoạt động trên tất cả platforms; Windows-style gây lỗi trên Unix.

### Đưa ra quá nhiều lựa chọn

```markdown
# Tệ:
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

# Tốt — cung cấp default, thêm escape hatch khi cần:
"Use pdfplumber for text extraction:
```python
import pdfplumber
```
For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
```

### Script xử lý lỗi kém (punt to Claude)

```python
# Tệ — để Claude tự xử lý khi thất bại:
def process_file(path):
    return open(path).read()

# Tốt — xử lý rõ ràng:
def process_file(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        print(f"File {path} not found, creating default")
        with open(path, "w") as f:
            f.write("")
        return ""
```

### Magic numbers không giải thích

```python
# Tệ:
TIMEOUT = 47
RETRIES = 5

# Tốt — self-documenting:
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

### Quên khai báo dependencies

```markdown
# Tệ:
"Use the pdf library to process the file."

# Tốt:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
```

## Checklist trước khi chia sẻ Skill

### Chất lượng cốt lõi
- [ ] Description cụ thể, bao gồm cả *cái gì* và *khi nào*
- [ ] Body của SKILL.md dưới 500 dòng
- [ ] Không có thông tin lỗi thời (hoặc trong section "old patterns")
- [ ] Terminology nhất quán xuyên suốt
- [ ] File references chỉ một cấp sâu từ SKILL.md
- [ ] Workflow có các bước rõ ràng

### Code và scripts
- [ ] Scripts xử lý lỗi thay vì punt cho Claude
- [ ] Không có magic numbers
- [ ] Packages required được khai báo và đã verify
- [ ] Không dùng Windows-style paths
- [ ] Validation/verification steps cho tác vụ quan trọng

### Testing
- [ ] Ít nhất 3 evaluations đã được tạo
- [ ] Test với các models định dùng
- [ ] Test với scenarios thực tế
- [ ] Thu thập feedback từ người khác (nếu áp dụng)

---

Nếu bạn chưa đọc phần giải thích architecture của Skills, xem lại [Phần 1: Agent Skills là gì?](/ai/agent-skills-la-gi-kha-nang-mo-rong-manh-me-cua-claude/) và [Phần 2: Cấu trúc và nguyên tắc viết Skill](/ai/agent-skills-best-practices-phan-1-viet-skill-ngan-gon-dung-cau-truc/).
