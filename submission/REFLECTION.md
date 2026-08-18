# Lakehouse Reflection: Top Anti-Pattern Risk

Trong **Top 5 Lakehouse Anti-Patterns**, team chúng tôi dễ vướng nhất vào **Small Files Problem do Ingestion streaming/micro-batch mà thiếu Compaction & VACUUM định kỳ**.

### 1. Nguyên nhân & Rủi ro:
- **Luồng dữ liệu realtime/CDC:** Dữ liệu streaming và log ghi liên tục theo chu kỳ ngắn (1–5 phút), sinh ra hàng loạt file Parquet rất nhỏ (< vài MB) kèm commit log mới.
- **Hệ quả:**
  - **Query latency tăng cao:** Query engine tốn phần lớn I/O để list metadata và mở file thay vì đọc dữ liệu thật.
  - **Storage & Metadata bùng nổ:** Thiếu `VACUUM` / `expire_snapshots` khiến snapshot cũ và orphan files tích lũy vô hạn, vừa tốn chi phí vừa vi phạm compliance (Nghị định 13/GDPR).

### 2. Giải pháp:
- Lên lịch job tự động chạy `OPTIMIZE ... ZORDER BY` gom file về kích thước tối ưu (128MB–1GB).
- Chạy `VACUUM` định kỳ với retention threshold hợp lý (ví dụ: 7 ngày) để dọn dẹp snapshot và orphan files.
