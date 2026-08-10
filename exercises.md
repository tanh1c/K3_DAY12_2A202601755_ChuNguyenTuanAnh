# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng trả lời mẫu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Chu Nguyễn Tuấn Anh  Mã học viên: 2A202601755

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi deploy, nếu tôi quên cấu hình `AGENT_API_KEY` thì process dừng ngay và log báo thiếu trường trước khi nhận traffic. Nếu có mặc định `"changeme"`, service vẫn báo deploy thành công nhưng người biết khóa mặc định có thể gọi `/ask`, làm lộ dữ liệu hoặc tiêu hết ngân sách rồi tôi mới phát hiện.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng tôi thu được: `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T03:13:29.994541+00:00", "user_id": "history-local", "tokens_in": 2, "tokens_out": 34, "cost_usd": 2.07e-05}`. Tôi có thể lọc và cộng `cost_usd` theo `user_id`, đồng thời nhóm theo `timestamp` để tính số request hoặc tỷ lệ lỗi trong từng khoảng thời gian. Một chuỗi `print` chung không có trường ổn định để máy truy vấn hai việc đó.

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
| 1 stage (bản đầu) | 1.17 GB |
| Multi-stage | 183 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build và đo bằng `docker images day12-agent`: bản single-stage dùng `python:3.11` đầy đủ là 1,17 GB, còn runtime multi-stage `python:3.11-slim` là 183 MB. Phần chênh lệch chủ yếu là hệ điều hành/base image đầy đủ, công cụ build, cache cài đặt và các thành phần chỉ cần lúc tạo dependency; runtime chỉ nhận kết quả đã cài từ builder cùng source cần chạy.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tôi đổi riêng `app/main.py`, build log cho thấy `COPY requirements.txt` và `RUN pip install` đều `CACHED`; từ `COPY app ./app` trở đi phải chạy lại, gồm copy source và tạo user ở image hiện tại. Nếu đặt `COPY . .` trước `RUN pip install`, thay đổi một ký tự trong source làm layer copy đổi và vô hiệu toàn bộ cache phía sau, nên pip phải tải/cài lại dependency dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Kẻ tấn công khai thác lỗi Python để chạy lệnh trong container; nếu process là root, lệnh đó có toàn quyền trong container và một lỗi kernel/runtime hoặc mount nhạy cảm có thể giúp tác động host với quyền cao. `USER appuser` cắt chuỗi ngay sau bước thực thi lệnh: process bị chiếm chỉ có UID 10001 với quyền tối thiểu. Cơ chế này không tự vá lỗ hổng thoát container, nhưng giảm quyền và hậu quả của nó.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng gửi được 20 request: 10 request vào giây 59 của phút trước và 10 request vào giây 00 hoặc 01 của phút sau. Bộ đếm cố định reset khi sang phút nên cả hai nhóm đều hợp lệ, dù dồn trong khoảng hai giây. Sliding window vẫn nhìn thấy nhóm cũ trong 60 giây gần nhất nên chặn nhóm thứ hai.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit bảo vệ tốc độ request trong 60 giây, còn cost guard bảo vệ tổng tiền của từng user trong tháng. Một user mới gửi request đầu tiên trong phút vẫn qua rate limit, nhưng phải bị cost guard chặn nếu đã tiêu gần hết ngân sách và chi phí ước tính làm vượt trần. Ngược lại, user còn nguyên ngân sách vẫn bị rate limit chặn ở request thứ 11 trong cùng cửa sổ 60 giây.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối → endpoint chung của cả ba container trả 503 → orchestrator hiểu nhầm cả ba process đã chết và đồng loạt restart chúng → trong lúc restart không còn instance phục vụ nên toàn bộ request thất bại → Redis trở lại nhưng các container vẫn cần khởi động và kiểm tra lại. Khi tách endpoint, `/health` vẫn 200 nên process không bị restart; chỉ `/ready` 503 để load balancer tạm ngừng gửi traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis, hai request local liên tiếp của cùng `X-User-Id` trả `history_length` lần lượt 0 rồi 2, vì message user và assistant đều được lưu dùng chung. Nếu dùng dict riêng trong ba process, request được load balancer chuyển sang instance khác sẽ đọc lịch sử riêng: số có thể nhảy 0, 0, 2, 0... thay vì tăng đều 0, 2, 4, 6..., và restart container còn làm nó về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
