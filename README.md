# multi-agent-research

Một bộ skill nghiên cứu học thuật cho Claude, xây theo kiến trúc **multi-agent chuyên môn hóa** — mỗi skill không phải một agent duy nhất mà là một đội agent, mỗi agent chịu trách nhiệm cho một phần công việc.

> Triết lý cốt lõi: *Agent kiểm chứng nguồn không viết bài. Agent viết bài không tự quyết định citation. Agent phản biện không chỉ khen mà phải chỉ ra điểm yếu, giả định ẩn và rủi ro lập luận.*

## Skill đầu tiên: `deep-research` (13 agent)

`deep-research/` điều phối **1 Lead Researcher + 12 subagent chuyên môn** qua 7 pha, với 3 checkpoint kiểm soát chất lượng.

| Pha | Agent | Trách nhiệm |
|-----|-------|-------------|
| 1 | clarifier | Làm rõ câu hỏi, xác định scope/ngôn ngữ/độ sâu |
| 2 | methodology-designer + ethics-checker | Thiết kế chiến lược tìm kiếm + kiểm tra bias/đạo đức *(song song)* |
| 3 | web-scout ×2 + document-scout + academic-scout | Tìm nguồn web (góc rộng + phản biện), tài liệu người dùng, học thuật *(song song)* |
| 4 | source-verifier → relevance-filter | Kiểm chứng độ tin cậy → sàng lọc & xếp hạng |
| 5 | evidence-synthesizer + gap-detector | Tổng hợp bằng chứng + phát hiện khoảng trống *(song song)* |
| 6 | devils-advocate | Phản biện ngược: điểm yếu, giả định ẩn, rủi ro lập luận |
| 7 | report-writer | Viết báo cáo có trích dẫn, theo ngôn ngữ câu hỏi |

**Checkpoints:** A (dừng nếu có vấn đề đạo đức) · B (cảnh báo nếu thiếu nguồn đã kiểm chứng) · C (loop lại nếu phản biện phát hiện lỗi nghiêm trọng).

### Mỗi agent định nghĩa theo công thức
**Vai trò + Nhiệm vụ + Đầu vào + Đầu ra + Tiêu chuẩn chất lượng + Anti-Patterns**

### Nguồn dữ liệu
Web (search + fetch) · tài liệu người dùng cung cấp · Google Drive.

### Đầu ra
Báo cáo nghiên cứu có cấu trúc + trích dẫn, ngôn ngữ tự động theo ngôn ngữ câu hỏi.

## Cách dùng

Cài như một Claude skill (Claude Code / claude.ai). Skill tự kích hoạt khi bạn yêu cầu nghiên cứu một chủ đề, điều tra một câu hỏi, hoặc cần một báo cáo có trích dẫn.

- **Claude Code:** dispatch subagent thật, chạy song song trong từng pha (Mode A).
- **claude.ai / môi trường không có subagent:** Lead Researcher đóng từng vai tuần tự theo đúng quy trình (Mode B fallback).

## Cấu trúc

```
deep-research/
├── SKILL.md              # Lead Researcher + orchestration 7 pha
├── subagents/            # 12 agent chuyên môn (mỗi file 1 agent)
└── references/
    ├── quality-gates.md  # Rubric tin cậy + logic checkpoint
    └── output-format.md  # Template báo cáo cuối
```

## Lộ trình (framework)

`deep-research` là skill nền tảng. Các skill tiếp theo trong framework:
- `academic-paper` — viết bài học thuật từ kết quả research
- `academic-paper-reviewer` — peer review
- `academic-pipeline` — điều phối cả 3 skill trên theo trình tự

## Tài liệu thiết kế

Spec và implementation plan đầy đủ trong [docs/superpowers/](docs/superpowers/).
