# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng giữ chỗ trong từng câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Dao Gia Bao  Mã học viên: 2A202602793

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu quên đặt `AGENT_API_KEY` trên môi trường deploy, cấu hình bắt buộc sẽ làm
ứng dụng dừng ngay khi khởi động. Tôi sẽ phát hiện lỗi ở log deploy trước khi
service nhận request; giá trị mặc định `changeme` có thể khiến service chạy
với một khóa ai cũng đoán được.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Một log thực tế sau khi gọi `/ask`:

```json
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T02:56:49.812532+00:00", "user_id": "sv-test", "tokens_in": 4, "tokens_out": 35, "cost_usd": 2.16e-05}
```

Từ các trường này có thể lọc chi phí theo user và đếm số request theo loại sự
kiện. Một dòng `print("đã trả lời xong")` không có trường để lọc hay tổng hợp.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 297 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Image một stage mang theo base Python đầy đủ và cài dependency trong cùng image.
Bản multi-stage chỉ chép thư viện đã cài sang `python:3.11-slim`, bỏ phần nền
đầy đủ của base image. Lần build tôi đo được 1.73 GB và 297 MB.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Tôi thử build một bản sao tạm có một dòng comment mới trong `app/main.py`.
Build log cho thấy `pip install` vẫn dùng cache, còn `COPY app` chạy lại. Nếu
`COPY . .` đứng trước `pip install`, sửa source sẽ làm mất cache từ bước copy
và cài dependency lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu lỗ hổng cho phép thoát khỏi tiến trình ứng dụng, quyền của tiến trình có
thể bị dùng để đọc hoặc sửa file và tiến trình khác trong container. Nếu app
chạy root thì kẻ tấn công có quyền cao ngay trong container. `USER appuser`
chạy ứng dụng với tài khoản thường và giới hạn thiệt hại trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Với cách đếm theo phút đồng hồ, có thể gửi 10 request ngay trước giây 00 rồi
thêm 10 request ngay sau khi bộ đếm reset: tổng cộng 20 request trong khoảng
hai giây. Sliding window luôn đếm đủ 60 giây gần nhất nên không có ranh giới
reset này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit chặn số lần gọi trong một khoảng thời gian; cost guard chặn tổng
chi tiêu theo user trong tháng. Mười request ngắn trong ngân sách vẫn có thể
được rate limit cho qua nhưng cost guard sẽ chặn nếu số dư tháng đã hết.
Ngược lại, user còn ngân sách có thể bị rate limit chặn sau khi gửi quá nhiều
request trong một phút.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Nếu health cũng kiểm tra Redis, Redis mất kết nối thì cả ba container sẽ báo
không khỏe; orchestrator có thể restart chúng dù tiến trình vẫn chạy. Khi
Redis trở lại, những lần restart đồng thời còn làm tăng tải. `/health` chỉ
kiểm tra tiến trình, còn `/ready` báo instance tạm ngừng nhận traffic khi
Redis chưa sẵn sàng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Trong test CP4, hai `ConversationStore` dùng chung Redis giả đọc được cùng một
lịch sử. Nếu mỗi instance dùng dict riêng, request thứ hai vào instance khác
không thấy lượt trước nên `history_length` sẽ thấp hơn hoặc quay về 0; Redis
giữ history bên ngoài từng process.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Tôi chưa triển khai lên cloud nên không gặp lỗi build hay health check của một
platform. Lệnh kiểm tra Railway CLI trả `zsh: command not found: railway`, và
môi trường cũng không có token Railway/Render. Vì vậy tôi dùng `LOCAL_FALLBACK`
với Docker Compose, nơi cả `agent` và Redis đều lên trạng thái healthy; ảnh
`/health` đã lưu ở `screenshots/health.png`.
