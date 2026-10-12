# Phân tích lỗi Phase A

Nhóm: K4-Track3B-ChungVanDuy

Người phụ trách: Chung Văn Duy - 2A202602854

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T07:58:07.086164+00:00.
Generation mode/fallback và điều kiện chạy: Chạy live qua answers chuẩn bị từ pipeline kết hợp retriever BAAI/bge-m3 và flashrank reranker.
Models generation/evaluator: Evaluator models gồm BAAI/bge-m3 và gpt-4o-mini theo adapter m4.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.7000 | 0.7000 | 0.7000 |
| answer_relevancy | 0.6799 | 0.6292 | 0.6777 |
| context_precision | 0.6373 | 0.5696 | 0.5778 |
| context_recall | 0.6518 | 0.5261 | 0.5031 |
| avg_score | 0.6672 | 0.6062 | 0.6146 |

## Bottom 10 và nguyên nhân

| Rank | ID | Câu hỏi | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 31 | Nhân viên đi công tác trong nước, ở khách sạn 2 đêm với giá 1.500.000 VNĐ/đêm. Công ty thanh toán tối đa bao nhiêu cho tiền khách sạn? | multi_hop | 0.5600 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 2 | 39 | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | multi_hop | 0.5600 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 3 | 40 | Nhân viên cấp Manager có thời hạn thử việc 90 ngày, đang trong tháng thứ 3, phát hiện vi phạm bảo mật. Họ nên và không nên làm gì theo chính sách? | multi_hop | 0.5600 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 4 | 34 | Nhân viên tạm ứng 4 triệu và một nhân viên khác tạm ứng 7 triệu: quy trình phê duyệt khác nhau thế nào? | multi_hop | 0.5686 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 5 | 47 | Khi phát hiện malware trên máy tính công ty, nhân viên có nên tự xử lý không? | adversarial | 0.5752 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 6 | 35 | Nhân viên Junior P1 có lương cơ bản 12 triệu vừa vào thử việc: nhận lương bao nhiêu và phụ cấp gì tháng đầu tiên? | multi_hop | 0.5778 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |\n| 7 | 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | multi_hop | 0.5825 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 8 | 44 | Bao lâu phải đổi mật khẩu một lần? | adversarial | 0.5825 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 9 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.5840 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |\n| 10 | 37 | Nhân viên tự ý xóa malware khỏi máy và chia sẻ thông tin sự cố này trên Slack nội bộ. Vi phạm những chính sách nào cụ thể? | multi_hop | 0.5878 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |

Đối chiếu câu hỏi đại diện ID 31 và ID 39 cho thấy nhóm lỗi retrieval xuất phát từ việc ngữ cảnh tìm kiếm bị nhiễu do chứa nhiều đoạn trích không trực tiếp liên quan hoặc trích nhầm phiên bản chính sách cũ (v1.0 thay vì v2.0). Việc reranking chưa đủ lọc bỏ các chunk trùng lặp về từ khóa nhưng khác ngữ nghĩa.

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 0 | 0 | 0 |
| context_recall | 4 | 11 | 8 |
| context_precision | 11 | 8 | 2 |
| answer_relevancy | 5 | 1 | 0 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.6062.
Worst metric xuất hiện nhiều nhất: context_recall; số câu: 23.

Nhóm multi_hop yếu nhất vì các câu hỏi yêu cầu tổng hợp thông tin từ nhiều nguồn hoặc so sánh nhiều điều kiện, khiến bước truy xuất dễ bỏ sót các đoạn văn bản quan trọng (kéo giảm context_recall). Đề xuất cải thiện là tăng cường kích thước chunk cha-con (hierarchical chunking) và áp dụng truy vấn mở rộng query expansion để thu thập đầy đủ ngữ cảnh liên quan.

## Bẫy adversarial và giới hạn

So sánh với factual (0.6672) và multi_hop (0.6062), nhóm adversarial có điểm trung bình là 0.6146. Điểm số của nhóm adversarial thấp hơn factual phản ánh độ phức tạp cao do xung đột phiên bản chính sách và các điều kiện phủ định. Để giải quyết triệt để, hệ thống cần bổ sung siêu dữ liệu phiên bản tài liệu (metadata version filtering) và cơ chế guardrail phát hiện mâu thuẫn thời gian.
