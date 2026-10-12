# Đánh giá LLM Judge Phase B

Nhóm: K4-Track3B-ChungVanDuy

Người phụ trách: Chung Văn Duy - 2A202602854

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T08:18:59.030222+00:00.
Models generation/judge: Judge model chính thức openai/gpt-oss-120b được dùng cho cả pairwise judge và binary calibration.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Lượt 1 (A/B) | Lượt 2 (B/A quy về A/B) | Final Winner | Position Consistent | Nhận xét lượt 1 |
|---|---|---|---|---|---|
| 1 | Pass1: B (0.20/0.90) | Pass2: B (0.20/0.80) | Final: B | Consistent: True | Answer B matches the reference exactly: 3 paid days, no dedu... |\n| 5 | Pass1: A (0.40/0.20) | Pass2: B (0.00/0.20) | Final: tie | Consistent: False | Answer A mentions the >50M tier but omits the approver, wher... |\n| 12 | Pass1: A (0.95/0.60) | Pass2: B (0.00/1.00) | Final: tie | Consistent: False | Answer A cites the source and includes the exact condition (... |\n| 21 | Pass1: B (0.10/0.90) | Pass2: B (0.00/1.00) | Final: B | Consistent: True | Answer B directly provides the correct number of vacation da... |\n| 41 | Pass1: tie (0.00/0.00) | Pass2: tie (0.00/0.00) | Final: tie | Consistent: True | Cả hai đáp án đều không phản ánh chính sách hiện hành 2024 (... |

Position bias count/rate: 2 câu bị ảnh hưởng / tỷ lệ 0.4000 (40.0%).
Verbosity win rate: 0.0000 (0.0%); decisive cases: 2.

Kết quả cho thấy ở 2/5 câu (ID 5 và 12), kết quả đảo chiều khi hoán đổi vị trí câu trả lời, minh chứng rõ rệt cho hiện tượng Position Bias của mô hình ngôn ngữ lớn khi làm giám khảo. Cơ chế swap-and-average đã trung hòa thành công sự thiên lệch này bằng cách gán nhãn hòa (tie) khi có xung đột giữa hai lượt đánh giá. Về verbosity bias, các câu trả lời dài hơn không tự động chiếm ưu thế (tỷ lệ thắng của câu dài hơn trong các case quyết định là 0.0%), cho thấy prompt đánh giá đã nhấn mạnh đúng tiêu chí súc tích và chính xác.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| ID | Model Answer Gốc | Nhãn Judge | Nhãn Human | Đồng thuận | Giải thích ngắn |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương ... | Judge: 1 | Human: 1 | Agree: True | Answer correctly states 3 days paid leave, matchin... |\n| 5 | Cần Giám đốc phòng ban phê duyệt.... | Judge: 0 | Human: 0 | Agree: True | Answer gives wrong approver (Giám đốc phòng ban) w... |\n| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng l... | Judge: 1 | Human: 1 | Agree: True | Câu trả lời khớp với nội dung tham chiếu: thưởng T... |\n| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm ni... | Judge: 1 | Human: 1 | Agree: True | Answer matches reference exactly: 18 days (15+3) a... |\n| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết.... | Judge: 1 | Human: 1 | Agree: True | Câu trả lời khớp với reference: nhân viên chưa đạt... |\n| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 ... | Judge: 0 | Human: 0 | Agree: True | Answer omits Kế toán trưởng, which the reference r... |\n| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng.... | Judge: 1 | Human: 1 | Agree: True | Answer matches reference: provides 1.500.000 VNĐ/m... |\n| 41 | Nhân viên được nghỉ 12 ngày phép năm.... | Judge: 0 | Human: 0 | Agree: True | Answer uses outdated 2023 policy (12 ngày) while r... |\n| 46 | Nhân viên thử việc không được nghỉ phép năm v... | Judge: 1 | Human: 1 | Agree: True | Answer matches reference: correctly states that pr... |\n| 50 | Được, miễn là đảm bảo kết nối an toàn.... | Judge: 0 | Human: 0 | Agree: True | Answer says 'Được' (allowed) while reference expli... |

Cohen’s κ: 1.0000.
Observed/chance agreement và diễn giải: Độ đồng thuận quan sát được (Po) đạt 1.0 (10/10 câu trùng khớp tuyệt đối giữa Judge và Human), xác suất ngẫu nhiên (Pe) là 0.50, mang lại hệ số Cohen kappa tuyệt đối 1.0000. Điều này chứng minh prompt calibration nhị phân đã định hình chính xác tiêu chí đánh giá khắt khe về độ đúng của chính sách nội bộ.

## Giới hạn và cách dùng judge

Dù đạt kết quả calibration cao trên bộ mẫu 10 câu, LLM-as-Judge vẫn mang các giới hạn cố hữu về độ nhạy cảm vị trí (Position Bias) như đã thấy trong phần pairwise. Do đó, trong thực tế sản xuất không nên xem quyết định của một lượt LLM Judge là chân lý tuyệt đối, mà bắt buộc phải áp dụng swap-and-average, kết hợp kiểm định mẫu ngẫu nhiên từ con người (human-in-the-loop) để liên tục hiệu chỉnh rubric chấm điểm.
