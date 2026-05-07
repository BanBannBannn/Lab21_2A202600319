# REPORT.md

## 1. Setup
- **Base Model**: `unsloth/Qwen2.5-3B-bnb-4bit` (Dựa trên MODEL_NAME trong notebook).
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`. Quy mô: 200 samples (Subset).
- **GPU**: Tesla T4 (16 GB VRAM).
- **Training Cost ước tính**: $0.07 (Tổng ~12.1 phút training @ $0.35/hr).

## 2. Rank Experiment Results
Dưới đây là bảng so sánh hiệu năng giữa các rank LoRA khác nhau:

| Rank | Alpha | Trainable Params | Time (min) | Peak VRAM (GB) | Eval Perplexity |
|------|-------|------------------|------------|----------------|-----------------|
| 8    | 16    | 1,843,200        | 4.16       | 12.70          | 4.75            |
| 16   | 32    | 3,686,400        | 4.03       | 10.62          | 4.55            |
| 64   | 128   | 14,745,600       | 3.87       | 13.48          | 4.38            |

## 3. Loss Curve Analysis
- **Overfitting**: Dựa trên biểu đồ training loss, đường loss giảm đều và ổn định qua các bước (69 steps). Không có dấu hiệu overfitting rõ rệt (như loss tăng ngược trở lại).
- **Tại sao**: Số lượng mẫu (200) và số epoch (3) được giữ ở mức nhỏ, kết hợp với kỹ thuật weight decay (0.01) và rank thấp giúp mô hình không bị 'học vẹt' dữ liệu đào tạo. Do tắt eval-during-training để tiết kiệm VRAM, việc theo dõi chủ yếu dựa trên xu hướng của train loss.

## 4. Qualitative Comparison

| Prompt | Base Model Response | Fine-tuned (r=16) Response | Nhận xét |
|--------|---------------------|----------------------------|----------|
| Giải thích Machine Learning | Giải thích mang tính lý thuyết, câu chữ chưa mượt | Câu trả lời mạch lạc hơn, định nghĩa rõ ràng | Cải thiện độ tự nhiên |
| Code Fibonacci | Code đúng nhưng giải thích dài dòng | Trình bày code sạch sẽ, giải thích súc tích | Cấu trúc phản hồi tốt |
| 5 nguyên tắc UI/UX | Liệt kê các điểm chung chung | Phân loại rõ ràng, có tính chuyên môn | Kiến thức chuyên sâu hơn |
| So sánh LoRA vs QLoRA | Trả lời đúng các khái niệm cơ bản | Đi sâu vào cơ chế nén và tối ưu hóa | Hiểu sâu về kỹ thuật |
| Phân biệt RAG vs FT | Phân tích cơ bản, đôi khi trùng lặp | Chỉ ra rõ sự khác biệt về retrieval vs training | Phân định rạch ròi |

## 5. Conclusion về Rank Trade-off
Qua thực nghiệm, có thể thấy rõ sự đánh đổi giữa tài nguyên và hiệu suất. **Rank r=64** mang lại chỉ số Perplexity thấp nhất (4.38), cho thấy khả năng dự đoán ngôn ngữ tốt nhất trong ba phương án. Tuy nhiên, nó tiêu tốn nhiều VRAM nhất (13.48 GB) và có số lượng tham số đào tạo lớn nhất. **Rank r=16** (baseline) tỏ ra là một điểm cân bằng cực tốt cho GPU T4, khi nó sử dụng ít VRAM nhất (10.62 GB) trong khi vẫn duy trì Perplexity ở mức cạnh tranh (4.55). Rank r=8 tuy nhanh nhưng Perplexity cao nhất (4.75), cho thấy mô hình chưa học được tối đa thông tin từ dữ liệu. Đối với dự án này, r=16 là lựa chọn tối ưu vì nó đảm bảo mô hình chạy an toàn trên 16GB VRAM của T4 mà không gặp rủi ro OOM (Out of Memory) trong khi vẫn cải thiện rõ rệt so với mô hình gốc. Nếu tài nguyên dư dả, r=64 sẽ được ưu tiên để đạt chất lượng cao nhất.

## 6. What I learned
- Hiểu rõ cách LoRA/QLoRA tác động đến việc giảm tải tài nguyên tính toán nhưng vẫn giữ được độ chính xác của mô hình 3B.
- Biết cách sử dụng thư viện Unsloth để tăng tốc độ fine-tuning và quản lý VRAM hiệu quả trên GPU cấu hình thấp như T4.
- Nhận ra rằng tăng Rank không phải lúc nào cũng làm tăng thời gian training đáng kể trên dataset nhỏ, nhưng lại ảnh hưởng trực tiếp đến bộ nhớ đồ họa.