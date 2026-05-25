# Tokenization audit

File này giúp kiểm tra tokenizer đang biến văn bản thành chuỗi token như thế nào.

- Tokenizer/model: `distilbert-base-uncased`
- `max_length`: `256`
- Số dòng train/val/test: `22500` / `2500` / `25000`
- Phân bố nhãn train: `{'0': 11250, '1': 11250}`

## Độ dài chuỗi token trên mẫu train

- Min: `41`
- Mean: `308.30`
- Median: `228.00`
- P90: `603.10`
- P95: `800.10`
- Max: `2222`
- Tỷ lệ mẫu có khả năng bị cắt ngắn: `42.95%`

## Gợi ý đọc kết quả

- Nếu tỷ lệ bị cắt ngắn quá cao, hãy thử tăng `--max_length`, nhưng cần quan sát thời gian chạy và bộ nhớ.
- Nếu `max_length` quá lớn, mô hình có thể chạy chậm hơn nhiều mà metric chưa chắc tăng.
- Khi so sánh mô hình, cần giữ split dữ liệu và metric giống nhau.
