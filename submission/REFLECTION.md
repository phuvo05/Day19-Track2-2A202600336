# Reflection — Lab 19

**Tên:** _<Họ Tên>_
**Cohort:** _<A20-K1 / A20-K2 / ...>_
**Path đã chạy:** lite (Windows + WSL, fastembed ONNX CPU + Qdrant in-memory + Feast SQLite)

---

## Câu hỏi (≤ 200 chữ)

**Mode nào thắng ở loại query nào?**

- **`exact` (15 queries):** BM25 thắng đơn độc ở 96.7% vs hybrid 96.7% và semantic 88.7%. Exact queries chứa từ kỹ thuật verbatim (PWA, service worker, Kubernetes) — BM25 đánh chính xác theo tần suất term, không cần semantic. Hybrid không cải thiện vì signal keyword đã quá mạnh.

- **`paraphrase` (15 queries):** Cả ba đều thất bại nặng (24-33%). Câu hỏi không chứa từ trong doc, và model `BAAI/bge-small-en-v1.5` được train trên tiếng Anh — semantic similarity giữa tiếng Việt paraphrase và tiếng Anh corpus bị suy giảm nghiêm trọng. BM25 nhỉnh hơn một chút (33.3%) vì vẫn bắt được partial token overlap. Trên production với `bge-m3` multilingual, semantic và hybrid sẽ thắng rõ ràng hơn.

- **`mixed` (20 queries):** Hybrid thắng áp đảo ở 100.0% vs keyword 97.0% và semantic 98.5%. Đây là pattern thực tế nhất (user thật viết vừa có từ kỹ thuật vừa có ý tưởng), và RRF k=60 kết hợp cả hai signal một cách mạnh mẽ.

**Hybrid không phải lúc nào cũng đúng:**
- Khi recall đã ~100% với BM25 thuần (exact queries), hybrid chỉ tăng latency và compute mà không cải thiện đáng kể.
- Khi corpus đơn ngữ và embedding model mạnh, paraphrase queries sẽ nghiêng về semantic — lúc đó pure vector search đủ tốt mà không cần BM25 overhead.
- Trong hệ thống latency-critical với budget P99 cứng (vd. <10ms), hybrid lookup đắt gấp đôi pure modes có thể không khả thi.

## Điều ngạc nhiên nhất khi làm lab này

Điều bất ngờ nhất là **BM25 thắng semantic trên cả paraphrase queries** (33.3% vs 24.0%) trên corpus tiếng Việt với embedding model English-trained. Điều này cho thấy embedding model choice không phải boilerplate mà là architectural decision có tính quyết định — sai model, toàn bộ semantic retrieval pipeline sụp đổ, và RRF cũng không cứu được. Lab này củng cố rất rõ ràng: **luôn cho AI biết ngôn ngữ, corpus size, và latency budget trước khi chọn embedding model**.

---

## Bonus challenge

- [ ] Đã làm bonus (xem `bonus/`)
- [ ] Pair work với: _<tên đồng đội nếu có>_
