# Blueprint CI/CD cho RAG và guardrails

Nhóm: K4-Track3B-ChungVanDuy

Người phụ trách: Chung Văn Duy - 2A202602854

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T08:37:25.543098+00:00; guard models: qwen/qwen3.8-27b.
Models generation/evaluator/judge từ A/B: Evaluator dùng BAAI/bge-m3 và gpt-4o-mini; Judge dùng openai/gpt-oss-120b; Guard dùng qwen/qwen3.8-27b.

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Kiến trúc phòng thủ đa tầng hoạt động theo nguyên tắc Defense-in-depth:
1. Tầng Presidio PII: Quét nhanh bằng biểu thức chính quy và NER cho các thực thể CCCD, điện thoại VN, email. Khi phát hiện PII nhạy cảm, request bị chặn ngay hoặc được ẩn danh hóa trước khi đi tiếp.
2. Tầng NeMo Input Rails: Kiểm tra jailbreak, prompt injection và yêu cầu ngoài phạm vi chính sách công ty. Nếu phát hiện vi phạm, hệ thống dừng sớm và trả về thông điệp từ chối tiêu chuẩn.
3. Tầng RAG Pipeline: Thực hiện truy xuất Hybrid (Dense + BM25) kết hợp Reranker và sinh câu trả lời dựa trên ngữ cảnh đã xác thực.
4. Tầng NeMo Output Rails: Kiểm tra câu trả lời trước khi gửi đến người dùng, đảm bảo không rò rỉ thông tin mật khẩu, thông tin cá nhân hay dữ liệu nhạy cảm.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2;
phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 14.07 | 19.89 | 22.29 | Đạt chuẩn hiệu năng cao dưới 25ms |
| NeMo input | 296.43 | 2466.51 | 2478.94 | Vượt ngưỡng ngân sách do cloud API call |
| Input guard sequential | 313.90 | 2473.71 | 2492.26 | Vượt ngưỡng ngân sách 500ms tổng thể |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế P95 = 2473.71ms; budget_ok: False.
Nguyên nhân nghẽn cổ chai: Thời gian xử lý của Presidio rất tối ưu (P95 chỉ đạt 19.89ms), tuy nhiên NeMo Input Rails phụ thuộc vào lời gọi LLM qua mạng Internet nên chịu độ trễ RTT đáng kể (2466.51ms). Để đưa P95 về dưới 500ms trong môi trường thực tế, giải pháp là thay thế LLM cloud bằng mô hình Guardrail chuyên dụng triển khai local (như Llama-Prompt-Guard hoặc SLM chạy trên GPU nội bộ).

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.
Kế hoạch triển khai CI: Đề xuất một job GitHub Actions chạy `python scripts/run_experiments.py --phase all` và `python scripts/check_lab.py --with-tests` trong pull request. Nếu bất kỳ chỉ số an toàn hoặc chất lượng nào không đạt ngưỡng, script sẽ trả mã thoát nonzero để chặn merge vào nhánh chính.

## Monitoring và cải tiến

Hệ thống giám sát vận hành sẽ theo dõi các chỉ số quan trọng sau:
1. Tỷ lệ chặn đầu vào theo danh mục (Jailbreak, PII, Off-topic) với ngưỡng cảnh báo khi tỷ lệ vi phạm tăng đột biến trên 15%.
2. Giám sát False Positive Rate thông qua phản hồi của người dùng, phân tích drift dữ liệu theo chu kỳ tuần.
3. Bảo mật thông tin: Mọi dữ liệu log truy vấn đều phải được redact PII trước khi ghi vào kho log tập trung.

RAGAS mean/worst distribution: multi_hop với điểm trung bình 0.6062.
Cohen’s κ: 1.0000.
Attacks passed: 20/20; clean false_positive_rate: 0.0000.

Các giải pháp cải tiến kiểm chứng được:
1. Nâng cấp bộ chunking sang kiến trúc Parent-Child phân cấp để tăng context_recall cho các câu hỏi phức tạp.
2. Triển khai mô hình SLM guardrail cục bộ để giảm độ trễ P95 của tầng bảo vệ input xuống dưới 200ms.
3. Mở rộng bộ luật PII recognizer cho các định dạng số định danh thuế và bảo hiểm xã hội Việt Nam.
