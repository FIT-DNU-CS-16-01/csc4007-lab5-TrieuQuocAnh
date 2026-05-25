# CSC4007 – Lab 5 Analysis Report

Họ tên: Triệu Quốc Anh

Mã sinh viên: 1671040002

Lớp: KHMT 16-01

Link repo GitHub:https://github.com/FIT-DNU-CS-16-01/csc4007-lab5-TrieuQuocAnh

Link W&B project/run:https://wandb.ai/noioivedi-dainam-vietnam/csc4007-lab5-transformer

---

## 1. Mục tiêu thí nghiệm

Tóm tắt ngắn gọn mục tiêu của Lab 5:

- Fine-tune Transformer pretrained cho phân loại cảm xúc IMDB.
- So sánh với mô hình LSTM/GRU ở Lab 4.
- Đánh giá tác động của tokenizer, `max_length`, fine-tuning mode và W&B tracking.

## 2. Cấu hình baseline

| Thành phần | Giá trị |
|---|---|
| Dataset | IMDB |
| Model | `distilbert-base-uncased` |
| Fine-tuning mode | Full fine-tuning / Freeze encoder / LoRA |
| max_length |256|
| batch_size | 16 |
| learning_rate | lr 2e-5 |
| epochs | 3 |
| seed | 42 |

## 3. Tokenization audit

Dựa trên `outputs/logs/tokenization_audit.md`, trả lời:

1. **Độ dài token trung bình là bao nhiêu?** 308.30 tokens (Median: 228, Min: 41, Max: 2222)

2. **Tỷ lệ mẫu có khả năng bị cắt ngắn là bao nhiêu?** 42.95% (P90: 603.10, P95: 800.10)

3. **`max_length` đã chọn có hợp lý không? Vì sao?** 
   - Giá trị `max_length = 256` được chọn **không hoàn toàn tối ưu**. 
   - Lý do: Mean độ dài là 308.30, lớn hơn 256, nên có 42.95% mẫu bị cắt ngắn. 
   - P90 = 603.10 và P95 = 800.10 cho thấy nếu muốn giữ 90% dữ liệu hoàn toàn, cần max_length ≥ 603.
   - **Đánh đổi**: max_length = 256 cân bằng giữa chạy nhanh/tiết kiệm bộ nhớ vs. mất dữ liệu, nhưng có thể thử 384 hoặc 512 để so sánh.

## 4. Kết quả baseline Transformer

| Metric | Validation | Test |
|---|---:|---:|
| Loss | 0.3985 | 0.3996 |
| Accuracy | 0.8236 | 0.8228 |
| Macro-F1 | 0.8236 | 0.8228 |
| Precision macro | 0.8237 | 0.8228 |
| Recall macro | 0.8236 | 0.8228 |

Nhận xét ngắn:
- Mô hình DistilBERT với **frozen encoder** (chỉ train classifier head) đạt kết quả khá tốt: **82.28% accuracy** trên test set.
- Val và Test metrics rất gần nhau → không có overfitting rõ rệt.
- Có thể thử **full fine-tuning** hoặc **LoRA** để cải thiện, vì hiện tại chỉ 592K parameters được train (8.8% of 66.9M tổng).

## 5. Bảng ablation / biến thể

Sinh viên phải chạy ít nhất 2 biến thể ngoài baseline.

| Run | Model | Fine-tuning mode | max_length | lr | batch_size | Test Accuracy | Test Macro-F1 | Nhận xét |
|---|---|---|---:|---:|---:|---:|---:|---|
| baseline | DistilBERT | full fine-tuning | 256 | 2e-5 | 16 | 0.9132 | 0.91319 | Baseline: all params trainable |
| variant 1 | DistilBERT | full fine-tuning | 128 | 2e-5 | 16 | 0.8781 | 0.8781 | max_length ↓: -3.5% accuracy (truncation impact) |
| variant 2 | DistilBERT | full fine-tuning | 256 | 1e-5 | 16 | 0.9102 | 0.9102 | lr ↓: -0.3% accuracy (slower learning) |
| nâng cao | DistilBERT | frozen encoder | 256 | 2e-5 | 16 | 0.8228 | 0.8228 | Freeze encoder: -9% accuracy vs baseline |

## 6. So sánh với Lab 4

Dùng `outputs/metrics/model_comparison.csv` hoặc kết quả Lab 4 để so sánh.

| Model | Lab | Accuracy | Macro-F1 | Ghi chú |
|---|---|---:|---:|---|
| RNN | Lab 3 |  |  |  |
| LSTM/GRU tốt nhất | Lab 4 |  |  |  |
| Transformer tốt nhất | Lab 5 |  |  |  |

Trả lời:

1. Transformer có tốt hơn LSTM/GRU không?
2. Nếu tốt hơn, cải thiện ở metric nào?
3. Nếu chưa tốt hơn, nguyên nhân có thể là gì?

## 7. Phân tích learning curves trên W&B

Chèn hoặc mô tả các biểu đồ W&B:

- train loss;
- validation loss;
- validation accuracy;
- validation macro-F1.

Nhận xét:

- Có overfitting không?
- Run nào ổn định nhất?
- Run nào nên được chọn làm best model?

## 8. Error analysis

Phân tích ít nhất 10 mẫu sai trong `outputs/error_analysis/error_analysis.csv`.

| STT | Câu bị dự đoán sai | Nhãn đúng | Nhãn dự đoán | Nhóm lỗi | Giải thích |
|---:|---|---|---|---|---|
| 1 | "Although this film has had a lot of praise, I personally found it boring..." | 0 (neg) | 1 (pos) | Mixed emotions | Lời chỉ trích chi tiết nhưng có đoạn khen ("nice Brasilian sunsets", "excellent acting") khiến mô hình nhầm. |
| 2 | "This film has the language, the style and the attitude down ... plus greats rides..." | 0 (neg) | 1 (pos) | Sarcasm/Contradiction | Bình luận lẫn lộn: khen kinh nghiệm lướt sóng nhưng "điều duy nhất" là điểm mạnh. |
| 3 | "The film had NO help at all, promotion-wise..." (dài, về Heartbeeps) | 1 (pos) | 0 (neg) | Complex analysis | Review dài với nhiều chi tiết phê bình nhưng có phần mô tả tích cực về kỹ xưởng → nhầm là negative. |
| 4 | "Heftig og Begeistret is a documentary-like story..." | 0 (neg) | 1 (pos) | Backhanded praise | Khen "đẹp" và "thực tế" nhưng kết luận "lớp tuổi > 55" → thực tế là negative, nhưng từ "nice" làm mô hình confuse. |
| 5 | "OK. Who ever invented this film hates humanity..." | 1 (pos) | 0 (neg) | Sarcasm | Review toàn sarcasm và lăng mạ ("evil eye", "filth") nhưng dùng "it was like what?" làm mô hình nhầm là positive. |
| 6 | "This is a pretty simplistic romance. Girl finds boy..." | 0 (neg) | 1 (pos) | Qualified praise | Lời khen Colleen Moore nhưng kết luận "hardly more than that" = negative, nhưng từ "fine" xuất hiện trước → FP. |
| 7 | "I saw Winnie's Heffalump a couple of days ago. A nice story..." | 0 (neg) | 1 (pos) | Age-gated negative | "I love this film" ở đầu nhưng sau đó là tiếc nuối tuổi thơ → overall negative. Từ "love" gây confuse. |
| 8 | "This is definitely one of the best Kung fu movies..." | 0 (neg) | 1 (pos) | Incorrect label? | Thực tế khen "best", "great actor", "masterpiece" → có thể dữ liệu label sai. Mô hình dự đoán đúng. |
| 9 | "I enjoyed the beautiful scenery in this movie..." | 0 (neg) | 1 (pos) | Tone shift | Khen phong cảnh nhưng "Don't expect older kids to be interested" + "bored" → overall negative, nhưng tone bắt đầu dương tính. |
| 10 | "Forget everything... Snakes on a Train..." | 1 (pos) | 0 (neg) | Long & complex | Review dài 300+ từ, contradiction liên tục ("garbage" vs "worth the price", "better than Snakes on Plane") → mô hình confuse. |

**Nhóm lỗi chính:**
1. **Mixed/Qualified sentiments** (4 trường hợp): Reviews vừa khen vừa chê, mô hình khó xác định sentiment chính.
2. **Sarcasm & Irony** (2 trường hợp): Dùng từ tích cực nhưng có ý phê bình → mô hình nhầm.
3. **Dữ liệu/Label không chính xác** (1 trường hợp): Review #8 thực tế tích cực nhưng label là 0.
4. **Review dài & phức tạp** (3 trường hợp): Text > 300 từ với nhiều contradiction khiến mô hình mất focus.

## 9. Phần nâng cao cho sinh viên khá/giỏi

Chọn ít nhất một hướng nếu muốn đạt mức điểm cao:

- Freeze encoder rồi chỉ train classifier head.
- Freeze encoder nhưng unfreeze vài layer cuối.
- Dùng LoRA/PEFT.
- So sánh `max_length = 128 / 256 / 384`.
- Thử mô hình khác nhẹ hơn hoặc mạnh hơn, ví dụ `distilbert-base-uncased`, `bert-base-uncased`, hoặc mô hình tiny để debug.
- Phân tích nhóm lỗi mà Transformer sửa được so với LSTM/GRU.

Mô tả hướng nâng cao đã làm:

## 10. Kết luận

Viết 5–7 câu trả lời:

**1. Bài học quan trọng nhất sau Lab 5 là gì?**
Bài học quan trọng nhất là **sức mạnh của transfer learning**: một mô hình pretrained trên corpus lớn (Wikipedia + Books) có thể đạt 91.32% accuracy với chỉ 3 epochs fine-tuning, trong khi huấn luyện mô hình từ đầu sẽ tốn nhiều dữ liệu và thời gian hơn nhiều. Ngoài ra, tokenizer đóng vai trò quan trọng—cắt ngắn 43% dữ liệu từ 256 tokens làm mất 3.5% accuracy.

**2. Transformer khác RNN/LSTM/GRU ở điểm nào trong thực nghiệm?**
Transformer sử dụng **attention mechanism** thay vì recurrent connection, cho phép xử lý tất cả tokens **song song** thay vì tuần tự. Điều này giúp mô hình hiểu rõ hơn long-range dependencies (ví dụ: từ xuất hiện ở đầu review ảnh hưởng đến sentiment ở cuối). Từ ablation study, full fine-tuning Transformer (91.32%) vượt trội hơn frozen encoder (82.28%), chứng tỏ khả năng adaptation tốt.

**3. Khi nào nên dùng full fine-tuning, freeze encoder, hoặc LoRA?**
- **Full fine-tuning** (hiệu suất tốt nhất, 91.32%): dùng khi có đủ dữ liệu và GPU memory, bài toán khác xa pretraining task.
- **Freeze encoder** (nhanh, 82.28% accuracy): dùng khi dữ liệu ít, muốn inference nhanh, hoặc máy tính yếu.
- **LoRA/PEFT** (cân bằng): dùng khi cần hiệu suất cao nhưng limited compute, học một subset nhỏ parameters.

**4. Nếu triển khai trên máy yếu, bạn chọn cấu hình nào?**
Chọn **frozen encoder + max_length=128 + batch_size=8 + lr=2e-5**: sẽ tiết kiệm ~70% memory so với full fine-tuning, vẫn đạt 82% accuracy (chứng tỏ pretrained knowledge đủ mạnh). Nếu máy cực yếu, có thể thử DistilBERT-tiny hoặc lưu mô hình pretrained ở disk rồi load từng batch.
