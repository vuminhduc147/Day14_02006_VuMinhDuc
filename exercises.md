# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 09:15–12:00

**Domain:** Northstar University Student Services

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 09:15–09:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (09:30–09:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời có diễn đạt lại hoặc suy luận nhẹ nên overlap từ vựng với context thấp, nhưng mọi claim vẫn kiểm chứng được. | Câu trả lời chứa thông tin, con số, thời hạn hoặc chính sách không được context hỗ trợ; đặc biệt nghiêm trọng với học phí, quyền riêng tư và thủ tục học vụ. | Kiểm tra từng claim với evidence, cải thiện prompt grounding/citation và chặn phát hành nếu có hallucination nghiêm trọng. |
| Answer Relevance | Câu hỏi rộng hoặc hội thoại cần một đoạn giải thích/định hướng bổ sung, nên một phần câu trả lời không khớp trực tiếp từ khóa câu hỏi. | Câu trả lời né tránh ý chính, trả lời sai intent hoặc đưa phần lớn nội dung không liên quan khiến sinh viên không thực hiện được bước tiếp theo. | Phân tích intent, chỉnh prompt để trả lời trực tiếp trước, loại nội dung thừa và bổ sung test cho các cách diễn đạt khác nhau. |
| Context Recall | Expected answer chứa chi tiết tùy chọn hoặc kiến thức nền không cần cho câu trả lời tối thiểu, trong khi retrieved context vẫn đủ trả lời đúng câu hỏi. | Retrieval bỏ sót điều kiện, ngoại lệ, deadline hoặc document quyết định, làm câu trả lời sai hay thiếu thông tin quan trọng. | Kiểm tra chunking/query, tăng hoặc điều chỉnh top-k, bổ sung metadata filtering/query expansion và test lại retrieval. |
| Context Precision | Nhiều chunk cùng chủ đề được lấy để tăng recall; chunk đúng vẫn nằm ở các vị trí đầu và nhiễu không ảnh hưởng câu trả lời. | Chunk không liên quan chiếm các rank đầu, đẩy evidence đúng xuống thấp hoặc làm model dựa vào policy sai/đã lỗi thời. | Cải thiện retriever và metadata filter, thêm reranking, rồi theo dõi precision theo rank cùng với recall. |
| Completeness | Câu trả lời cố ý ngắn gọn, bỏ phần phụ hoặc ví dụ nhưng vẫn có đủ thông tin bắt buộc để người dùng hành động an toàn. | Thiếu một bước bắt buộc, điều kiện đủ, ngoại lệ, deadline, khoản phí hoặc kênh escalation làm câu trả lời gây hiểu nhầm. | Ánh xạ expected answer thành các required claims, chỉnh prompt/checklist và thêm regression case cho claim bị thiếu. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Tạo một tập câu hỏi có hai câu trả lời A và B đã được human đánh giá, gồm cả cặp chất lượng tương đương và cặp có đáp án tốt hơn rõ ràng. Condition 1 đưa A trước B; condition 2 đảo thứ tự B trước A nhưng giữ nguyên prompt, rubric, model và tham số. Chạy nhiều lần với thứ tự mẫu được random hóa và so sánh tỷ lệ thắng của từng answer. Nếu cùng một answer được chọn nhiều hơn đáng kể khi đứng đầu (ví dụ chênh lệch vượt ngưỡng thống kê đã định), judge có position bias. Có thể thêm condition 3 chấm từng answer độc lập để làm baseline không có vị trí tương đối.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Rubric phải chấm theo các claim/tiêu chí quan sát được như correctness, coverage, relevance và safety, không dùng độ dài hay mức độ chi tiết như tín hiệu chất lượng. Yêu cầu judge bỏ qua văn phong và phần lặp, không cộng điểm cho giải thích ngoài câu hỏi, đồng thời phạt nội dung thừa hoặc không liên quan. Cung cấp anchor examples trong đó câu trả lời ngắn nhưng đủ ý đạt điểm cao hơn câu dài có lặp/nhiễu, và có thể chuẩn hóa hoặc giới hạn độ dài trước khi chấm.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Human labels là chuẩn tham chiếu để kiểm tra judge có thực sự phản ánh rubric và yêu cầu domain hay chỉ tạo điểm số nhất quán với bias của model. Calibration giúp đo agreement, phát hiện systematic bias, chọn prompt/threshold phù hợp và xác định các nhóm câu hỏi judge không đáng tin. Cần dùng nhiều người chấm, giải quyết bất đồng và giữ một calibration set riêng; các case bất đồng cao hoặc rủi ro cao phải chuyển sang human review.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | ≥ 0.85 | Đây là safety gate quan trọng nhất: claim không có căn cứ có thể khiến sinh viên làm sai thủ tục; ngoài average, không cho phép regression nghiêm trọng ở từng case critical. |
| Answer Relevance | ≥ 0.75 | Cho phép một ít nội dung hướng dẫn bổ sung nhưng vẫn yêu cầu phần lớn câu trả lời giải quyết đúng intent; đồng thời không được giảm quá 0.05 so với baseline. |
| Completeness | ≥ 0.80 | Các bước, điều kiện và ngoại lệ chính phải được bao phủ; threshold cao giúp tránh câu trả lời đúng một phần nhưng không thể hành động. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Dùng offline evaluation trong mỗi pull request và trước release trên golden/regression dataset vì kết quả lặp lại được, không ảnh hưởng người dùng và phù hợp để block deployment. Dùng online evaluation sau khi phát hành bằng telemetry, feedback, A/B hoặc shadow testing để phát hiện distribution shift và hành vi thực tế mà bộ test chưa bao phủ; chỉ thu thập dữ liệu phù hợp chính sách riêng tư và có rollback/alert. Dùng human review để tạo và hiệu chỉnh gold labels, xử lý case mơ hồ hoặc judge bất đồng, audit định kỳ, và phê duyệt các thay đổi/câu trả lời rủi ro cao như học phí, quyền riêng tư, khiếu nại hay chính sách học vụ. Ba lớp bổ sung cho nhau: offline là gate, online là monitor, human là chuẩn kiểm định và escalation.

---

## Part 2 — Core Coding (09:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho Student Services. Mỗi mức phải đủ cụ thể để
hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
