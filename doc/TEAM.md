# Nhóm thực hiện K4 Track 3B

Tên nhóm: K4-Track3B-ChungVanDuy

Repo chung: https://github.com/CoderNVU/K4-Track-3B-Day24-ChungVanDuy-2A202602854-Eval-Guard

Trưởng nhóm/người nộp: Chung Văn Duy - 2A202602854

Ngày lab/deadline áp dụng: 2026-10-12 18:00:00 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: HEAD

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Chung Văn Duy | 2A202602854 | Toàn bộ Phase A, Phase B, Phase C, tích hợp pipeline Day 18 và hoàn thiện toàn bộ báo cáo | src/phase_a_ragas.py, src/phase_b_judge.py, src/phase_c_guard.py, artifacts, doc/reports |

Người tích hợp/chạy batch/nộp link: Chung Văn Duy - 2A202602854, hoàn thành chạy các script experiments live và submission checker đạt 100%.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11 64-bit, Python 3.12, MSVC Build Tools / pure-python annoy fallback.
Virtualenv/dependencies/imports: Đã cài đặt đầy đủ ragas 0.1.22, langchain 0.2.17, nemoguardrails 0.10.0, presidio-analyzer 2.2.364, spacy 3.8.16 cùng model en_core_web_lg và fastembed 0.9.0.
Models/Qdrant/50 answers đã chuẩn bị: 50 câu hỏi đã được sinh và kiểm tra hợp lệ tại artifacts/answers_50q.json sử dụng retriever BAAI/bge-m3 kết hợp flashrank reranker.
Đã xử lý trọn vẹn tích hợp NeMo Guardrails chạy mượt mà cùng LangChain 0.2 và hoàn tất đo kiểm tra an toàn hệ thống.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### ChungVanDuy-2A202602854

- Phần được giao và phần thực tế đã làm: Hoàn thiện 100% các task từ Task 1 đến Task 13 bao gồm Phase A (RAGAS evaluation & failure clustering), Phase B (Pairwise LLM Judge swap-and-average, Cohen kappa calibration, bias analysis) và Phase C (Presidio PII guard, NeMo input/output rails, adversarial suite, P95 latency measurement).
- Bằng chứng: Các file nguồn src/phase_a_ragas.py, src/phase_b_judge.py, src/phase_c_guard.py không còn TODO nào; artifacts/answers_50q.json, artifacts/ragas_50q.json, artifacts/judge_results.json, artifacts/guard_results.json được sinh từ live runner.
- Cách kiểm tra: Đã chạy thành công 86 unit tests offline (100% pass) và kiểm tra submission check bằng scripts/check_lab.py --with-tests đạt 0 failed checks.
- Điều đã học: Hiểu sâu sắc cơ chế đánh giá RAG đa chiều (Faithfulness, Relevancy, Precision, Recall), hiện tượng Position Bias / Verbosity Bias trong LLM-as-Judge và giải pháp Swap-and-Average, kỹ thuật phòng thủ đa lớp (Defense-in-depth) kết hợp Regex Pattern, Presidio NER và NeMo Guardrails Self-Check Rails.
- Giới hạn/cách xử lý: Độ trễ P95 của tầng bảo vệ input qua cloud API đạt 2473ms (vượt budget 500ms do độ trễ mạng Internet). Trong môi trường production thực tế, giải pháp tối ưu là triển khai local lightweight LLM (như Llama-Guard hoặc SLM local qua vLLM/Ollama) trên cùng mạng nội bộ.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity IDE hỗ trợ pair programming, tự động kiểm thử và rà soát hợp đồng dữ liệu chuẩn xác theo rubric.

## Cập nhật sau khi nộp

Bản nộp chính thức hoàn thiện vào ngày 12/10/2026, đáp ứng đầy đủ yêu cầu và kiểm thử hợp lệ.
