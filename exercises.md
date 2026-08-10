# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay các dòng giữ chỗ trong từng câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Thị Liên  Mã học viên: 2A202601795

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu quên set `AGENT_API_KEY` mà app vẫn dùng mặc định `"changeme"` thì service public có thể chạy với một key ai cũng đoán được. Việc fail fast làm deploy lỗi ngay từ đầu, mình nhìn log biết thiếu biến môi trường và sửa trong dashboard, thay vì để app chạy sai rồi bị người khác gọi `/ask`.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ log JSON: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T03:00:00+00:00","user_id":"sv-test","tokens_in":5,"tokens_out":42,"cost_usd":0.000026}`. Với log này mình lọc được theo `event` hoặc `user_id`, và cộng/đếm được `cost_usd`, `tokens_in`, `tokens_out` bằng công cụ log. Một câu `print("đã trả lời xong")` không cho biết user nào gọi, tốn bao nhiêu token hay chi phí bao nhiêu.

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
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản ban đầu dùng `python:3.11` full và copy cả repo vào image nên thường rất nặng, khoảng gần 1GB. Bản multi-stage dùng `python:3.11-slim` và chỉ copy dependency đã cài, `app/`, `utils/` nên nhỏ hơn nhiều, test yêu cầu dưới 500MB. Phần chênh lệch chủ yếu là compiler, build cache, file `.git`, `.venv`, test/cache và các package hệ điều hành không cần ở runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại, `COPY requirements.txt` và `RUN pip install` nằm trước khi copy source. Nếu chỉ sửa một ký tự trong `app/main.py`, các layer base image và layer cài dependency được dùng lại từ cache; chỉ layer `COPY app ./app` và các layer sau nó phải chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa code Docker sẽ coi context thay đổi và có thể cài lại toàn bộ dependency, build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app Python có lỗ hổng cho phép ghi file hoặc chạy lệnh trong container, attacker sẽ chạy lệnh với quyền user hiện tại. Nếu container chạy root, khi có lỗi mount volume, cấu hình Docker yếu hoặc escape container, quyền root đó có thể làm thiệt hại lớn hơn trên host. Lệnh `USER appuser` cắt chuỗi này ở bước trong container: code bị khai thác chỉ có quyền user thường, không phải root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Nếu đếm theo phút đồng hồ, user có thể gửi tối đa 20 request trong 2 giây khi limit là 10/phút: gửi 10 request lúc 10:00:59, sau đó sang 10:01:00 bộ đếm reset và gửi tiếp 10 request lúc 10:01:01. Sliding window 60 giây tránh lỗ hổng này vì nó luôn nhìn lại 60 giây gần nhất, không phụ thuộc mốc giây 00.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số lượng request trong một cửa sổ thời gian, còn cost guard giới hạn tổng tiền đã tiêu trong tháng. Ví dụ user gửi ít request, vẫn dưới 10/phút, nhưng mỗi request rất dài và chi phí tháng đã gần hết thì rate limit cho qua nhưng cost guard phải chặn. Ngược lại, user mới chưa tốn tiền nhưng spam 11 request trong vài giây thì cost guard vẫn cho qua về ngân sách, còn rate limit chặn 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp `/health` và `/ready` rồi bắt endpoint đó kiểm tra Redis, khi Redis mất kết nối 30 giây thì cả 3 container sẽ bắt đầu trả health check lỗi. Platform tưởng cả 3 process chết nên restart container hàng loạt. Trong lúc restart, request đang xử lý bị cắt và cụm không còn instance ổn định để nhận traffic. Đúng ra `/health` chỉ kiểm tra process còn sống, còn `/ready` mới kiểm tra Redis để load balancer tạm ngừng gửi request.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi lưu history trong Redis, các container khác nhau vẫn đọc cùng key `history:<user_id>`, nên request thứ hai với cùng `X-User-Id` thấy `history_length` tăng lên 2 sau lượt hỏi đầu tiên. Nếu dùng dict Python trong RAM, mỗi container có dict riêng; request đi vào container khác sẽ thấy history rỗng hoặc lúc tăng lúc không, làm `history_length` không ổn định và agent giống như mất trí nhớ.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi mình gặp khi deploy là Railway báo health check failure. Mình kiểm tra lại cấu hình và thấy `railway.toml` đang override `startCommand` bằng lệnh dùng `$PORT`; có khả năng biến port không được expand đúng nên app không listen đúng cổng Railway cấp. Mình bỏ `startCommand` để dùng `CMD` trong Dockerfile với `sh -c "uvicorn ... --port ${PORT:-8000}"`, tăng `healthcheckTimeout`, rồi kiểm tra local bằng container chạy `PORT=9090`. Sau khi redeploy và set đúng `REDIS_URL` từ Railway Redis service, `/health` và `/ready` đều trả 200.
