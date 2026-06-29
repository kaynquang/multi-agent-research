# Design: deep-research skill (13-agent)

**Date:** 2026-06-29  
**Status:** Approved for implementation  
**Scope:** Skill đầu tiên trong bộ academic-research-skills framework

---

## 1. Bối cảnh & mục tiêu

Build skill `deep-research` theo kiến trúc multi-agent chuyên môn hóa: mỗi agent chịu trách nhiệm một phần công việc, không kiêm nhiệm. Skill được đóng gói thành folder portable, push lên git để chia sẻ và cài đặt.

**Mục tiêu:** Người dùng đặt câu hỏi nghiên cứu → skill tự điều phối 13 agent → trả về báo cáo nghiên cứu có trích dẫn, đã kiểm chứng nguồn, đã phản biện ngược.

**Không phải mục tiêu:** Viết bài báo học thuật (đó là skill `academic-paper`), peer review (skill `academic-paper-reviewer`).

---

## 2. Triết lý thiết kế

- **Cô lập trách nhiệm:** Agent kiểm chứng nguồn không viết bài. Agent viết bài không tự quyết định citation đúng. Agent phản biện không chỉ khen.
- **Interface rõ ràng:** Mỗi agent nhận input có cấu trúc, trả output có cấu trúc. Lead researcher tổng hợp giữa các phase.
- **Fail early:** Có checkpoint gating — nếu ethics-checker phát hiện vấn đề, hoặc verified_sources dưới ngưỡng → dừng sớm và báo cáo thay vì sinh output chất lượng thấp.
- **Portable:** Chạy tốt nhất trong Claude Code (subagent thật), có fallback role-play cho claude.ai.

---

## 3. Folder structure

```
deep-research/
├── SKILL.md                    ← Lead researcher + orchestration logic
├── subagents/
│   ├── clarifier.md
│   ├── methodology-designer.md
│   ├── ethics-checker.md
│   ├── web-scout.md
│   ├── document-scout.md
│   ├── academic-scout.md
│   ├── source-verifier.md
│   ├── relevance-filter.md
│   ├── evidence-synthesizer.md
│   ├── gap-detector.md
│   ├── devils-advocate.md
│   └── report-writer.md
└── references/
    ├── output-format.md        ← Template báo cáo cuối
    └── quality-gates.md        ← Tiêu chuẩn pass/fail mỗi checkpoint
```

**13 agent = 1 lead (SKILL.md) + 12 subagent file.**

---

## 4. 13 Agent — Định nghĩa theo công thức

Công thức: **Vai trò + Nhiệm vụ + Đầu vào + Đầu ra + Tiêu chuẩn chất lượng**

**13 agent instances = 1 Lead (SKILL.md) + 12 subagent files, trong đó web-scout.md được dispatch 2 lần với angle khác nhau.**

| Instance | File | Vai trò | Nhiệm vụ |
|----------|------|---------|----------|
| 1 | SKILL.md | Lead Researcher — điều phối tổng thể | Nhận câu hỏi, dispatch agents theo pha, tổng hợp giữa pha, kiểm tra checkpoint |
| 2 | clarifier.md | Chuyên gia làm rõ vấn đề | Hỏi làm rõ câu hỏi nếu mơ hồ, xác định scope, ngôn ngữ, độ sâu mong đợi |
| 3 | methodology-designer.md | Chuyên gia phương pháp | Thiết kế chiến lược tìm kiếm: keywords, nguồn ưu tiên, angles[] |
| 4 | ethics-checker.md | Giám sát đạo đức & bias | Kiểm tra bias ngay từ đầu, flag vấn đề đạo đức, quyết định proceed/stop |
| 5 | web-scout.md (dispatch #1, angle: broad) | Thám tử web góc rộng | WebSearch + WebFetch; tìm bài tổng quan, tin tức, báo cáo |
| 6 | web-scout.md (dispatch #2, angle: critical) | Thám tử web góc phản biện | WebSearch + WebFetch; tìm phản biện, góc nhìn thiểu số, recent criticism |
| 7 | document-scout.md | Chuyên gia phân tích tài liệu | Đọc file người dùng cung cấp + Google Drive; trích xuất luận điểm & dữ liệu |
| 8 | academic-scout.md | Chuyên gia tài liệu học thuật | Tìm preprint, open-access papers (arXiv, PubMed mở, SSRN); ưu tiên peer-reviewed |
| 9 | source-verifier.md | Chuyên gia kiểm chứng | Đánh giá credibility từng nguồn (1-5), flag nguồn đáng ngờ, loại nguồn yếu |
| 10 | relevance-filter.md | Chuyên gia sàng lọc | Xếp hạng nguồn theo độ liên quan với câu hỏi, loại trùng lặp |
| 11 | evidence-synthesizer.md | Chuyên gia tổng hợp | Tìm patterns, contradictions, consensus; map bằng chứng vào câu hỏi |
| 12 | gap-detector.md | Chuyên gia phân tích khoảng trống | Phát hiện điều chưa được trả lời, góc chưa khám phá, câu hỏi follow-up |
| 13 | devils-advocate.md | Phản biện viên | Tìm điểm yếu trong tổng hợp, giả định ẩn, rủi ro lập luận — KHÔNG chỉ khen |
| — | report-writer.md | Nhà văn học thuật | Viết báo cáo cuối: có cấu trúc, citations, theo ngôn ngữ câu hỏi, theo template |

> `report-writer.md` là file thứ 12 trong `subagents/` — không được đánh số instance riêng vì Lead Researcher gọi nó trực tiếp ở pha cuối, không song song. Tổng agent instances trong một run đầy đủ: 13.

---

## 5. Orchestration flow

```
[PHASE 0] User đặt câu hỏi → Lead Researcher nhận
    ↓
[PHASE 1] dispatch clarifier
    OUTPUT: {research_question, scope, constraints, language, depth}
    ↓
[PHASE 2] fan-out SONG SONG:
    ├── methodology-designer → {search_strategy, keywords[], sources_priority, angles[]}
    └── ethics-checker       → {bias_flags[], ethical_concerns[], proceed: bool, notes}
    ↓
    [CHECKPOINT A] — nếu proceed: false → Lead Researcher báo cáo vấn đề, dừng
    ↓
[PHASE 3] fan-out SONG SONG (scouts):
    ├── web-scout-A    → {sources[]}  (góc rộng theo strategy.angles[0])
    ├── web-scout-B    → {sources[]}  (góc sâu/phản biện theo strategy.angles[1])
    ├── document-scout → {docs[]}     (file người dùng + Drive nếu có)
    └── academic-scout → {papers[]}   (open-access academic)
    ↓
[PHASE 4] tuần tự:
    source-verifier    → {verified[], rejected[], overall_confidence}
    relevance-filter   → {ranked_sources[], dropped[]}
    ↓
    [CHECKPOINT B] — nếu verified < 3 hoặc confidence: low → cảnh báo, offer re-search
    ↓
[PHASE 5] fan-out SONG SONG:
    ├── evidence-synthesizer → {themes[], evidence_map, contradictions[], confidence_level}
    └── gap-detector         → {gaps[], unanswered[], recommended_followup[]}
    ↓
[PHASE 6] devils-advocate
    INPUT: toàn bộ output Phase 5
    OUTPUT: {weak_points[], hidden_assumptions[], counter_arguments[], severity[]}
    ↓
[PHASE 7] report-writer
    INPUT: tất cả output từ Phase 1-6
    OUTPUT: Final report (markdown, ngôn ngữ theo research_question.language)
```

---

## 6. Interface schema (mỗi agent)

Mỗi file subagent mô tả:
1. **Role statement** — 1 câu: tôi là ai, tôi chịu trách nhiệm gì
2. **Input contract** — các trường input tôi nhận (tên + mô tả)
3. **Task** — hướng dẫn chi tiết làm gì
4. **Output contract** — các trường output tôi trả về (tên + kiểu + mô tả)
5. **Quality standards** — tiêu chuẩn tôi tự đánh giá trước khi trả kết quả
6. **Anti-patterns** — những gì tôi KHÔNG làm (giữ cô lập vai)

Ví dụ output schema của source-verifier:
```
verified_sources:
  - source: url/title
    credibility_score: 1-5
    flags: [outdated | anonymous | no_peer_review | conflict_of_interest | ...]
    verdict: accept | accept_with_caution | reject
rejected_sources:
  - source: url/title
    reason: string
overall_confidence: high | medium | low
```

---

## 7. Nguồn dữ liệu

| Scout | Tools | Chiến lược |
|-------|-------|-----------|
| Web Scout A | WebSearch, WebFetch | Tìm broad: overview, official sources, reports |
| Web Scout B | WebSearch, WebFetch | Tìm counter: criticism, minority view, recent updates |
| Document Scout | Read (local files), Google Drive MCP | Đọc tài liệu người dùng cung cấp |
| Academic Scout | WebSearch (site:arxiv.org, site:pubmed.ncbi.nlm.nih.gov, etc.), WebFetch | Ưu tiên peer-reviewed, preprint chấp nhận được nếu có preprint server uy tín |

---

## 8. Fallback cho môi trường không có Agent tool

SKILL.md có section "Environment Check" ở đầu:
- **Claude Code / môi trường có Agent tool:** dispatch subagent thật, đọc file subagent làm system prompt của subagent
- **Claude.ai / môi trường không có Agent tool:** Lead Researcher lần lượt "mặc" từng vai, đọc file subagent tương ứng (progressive disclosure), thực hiện đúng thứ tự phases, tự kiểm soát bằng checklist

Fallback kém cô lập hơn nhưng vẫn theo đúng quy trình và quality gates.

---

## 9. Quality gates (tóm tắt)

| Checkpoint | Điều kiện dừng sớm |
|-----------|-------------------|
| A (sau Phase 2) | ethics-checker: proceed = false |
| B (sau Phase 4) | verified_sources < 3 HOẶC overall_confidence = low HOẶC tất cả scouts trống |
| C (sau Phase 6) | devils-advocate tìm thấy fatal flaw trong tổng hợp (severity: critical) → loop lại Phase 5 hoặc cảnh báo |

Tiêu chuẩn chi tiết hơn trong `references/quality-gates.md`.

---

## 10. Đầu ra cuối (báo cáo)

Ngôn ngữ: theo ngôn ngữ câu hỏi nghiên cứu (auto-detect từ clarifier).

Cấu trúc báo cáo (references/output-format.md):
1. **Tóm tắt điều hành** (3-5 câu)
2. **Câu hỏi nghiên cứu đã làm rõ**
3. **Phương pháp** (cách tìm, nguồn nào được kiểm tra)
4. **Bằng chứng chính** (themes, grouped by confidence)
5. **Contradictions & debates**
6. **Khoảng trống nghiên cứu**
7. **Phản biện & giới hạn**
8. **Câu hỏi follow-up đề xuất**
9. **Tài liệu tham khảo** (với credibility score)

---

## 11. Phạm vi ngoài thiết kế này

Các skill tiếp theo sau khi deep-research hoàn thành:
- `academic-paper` (12 agent) — viết bài từ kết quả research
- `academic-paper-reviewer` (7 agent) — peer review
- `academic-pipeline` (5 agent điều phối) — gọi 3 skill trên theo trình tự

---

## 12. Câu hỏi mở (đã quyết định)

| Câu hỏi | Quyết định |
|---------|-----------|
| Mỗi agent = file riêng hay gom nhóm? | Mỗi agent = 1 file trong `subagents/` |
| Môi trường chạy | Claude Code là primary; portable fallback |
| Nguồn dữ liệu | Web + tài liệu người dùng + Google Drive |
| Ngôn ngữ đầu ra | Theo ngôn ngữ câu hỏi |
| Bắt đầu từ skill nào | deep-research trước |
