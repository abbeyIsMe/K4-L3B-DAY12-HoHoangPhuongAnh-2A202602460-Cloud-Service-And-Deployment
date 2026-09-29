# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: HỒ HOÀNG PHƯƠNG ANH  Mã học viên: 2A202602460

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên AGENT_API_KEY khi deploy, app sẽ báo lỗi khi khởi động thay vì chạy với một key mặc định. Điều này giúp phát hiện cấu hình bị thiếu sớm và tránh việc service chạy với một API key không an toàn

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T09:36:36.422241+00:00", "user_id": "anonymous", "tokens_in": 40, "tokens_out": 52, "cost_usd": 3.72e-05} Log dạng JSON lọc và đếm request theo user hoặc event, đồng thời theo dõi được số token và chi phí của mỗi request. Với print("đã trả lời xong") thì không có các thông tin này để máy có thể xử lý và thống kê

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
| 1 stage (bản đầu) | 289 MB |
| Multi-stage | 273 MB |

(IMAGE                         ID             DISK USAGE   CONTENT SIZE   EXTRA
day12-agent-onestage:latest   61c8520ab273        289MB         68.5MB        
    
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
day12-agent-multistage:latest   23f4557d2d95        273MB         64.5MB       )

Giải thích: phần dung lượng chênh lệch đó là những gì?

> One-stage có kích thước 289 MB, còn multi-stage là 273 MB. Multi-stage nhỏ hơn vì các phần và dependencies chỉ dùng để build được giữ ở stage builder, còn image cuối chỉ chứa những phần cần thiết để chạy ứng dụng


---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa một ký tự trong app/main.py, Docker vẫn dùng cache cho COPY requirements.txt và RUN pip install, còn COPY . . và các bước phía sau phải chạy lại. Nếu đặt COPY . . trước RUN pip install thì khi source code thay đổi, bước pip install cũng chạy lại, làm thời gian build lâu hơn

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu có chỗ yếu, attacker sẽ tấn công vào chỗ yếu đó để truy cạp và tiếp tục tấn công vào các phần khác. Nếu app chạy bằng root thì attacker có thể có quyền cao hơn bìh thường trong container và ảnh hưởng nhiều hơn. USER appuser giúp cắt chuỗi này bằng cách cho app chạy với user thường thay vì root

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Sliding window kiểm tra số request trong 60 giây gần nhất nên giúp hạn chế việc user, bot hoặc AI gửi quá nhiều request trong thời gian ngắn. Với giới hạn 10 request trong 60 giây, trong 2 giây liên tiếp tối đa có 10 request. Cách này kiểm soát request chặt hơn fixed window nhưng lại tốn tài nguyên hơn

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit là giới hạn request trong 1 khảong thời gian, còn cost guard là limit số token/ hay là chi phí mà requets có thể sử dụng. Nếu chỉ có ít request,mỗi request lại khá dài-rate limit sẽ cho qua, việc này sẽ tốn token, cost guard sẽ chặn lại. Ngược lại, người dùng có thể gửi đủ 10 request ngắn và chưa vượt rate limit, nhưng nếu tổng token hoặc chi phí vượt giới hạn thì cost guard vẫn chặn

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp /health và /ready, cho endpoint kiểm tra Redis, khi Redis mất kết nối thì cả 3 agent có thể bị đánh dấu là không khỏe và bị restart, dù app (health) vẫn đang chạy. Nếu tách hai endpoint, /health vẫn trả về OK vì app còn sống, còn /ready sẽ không OK vì Redis không hoạt động. Khi Redis kết nối lại, agent có thể trở lại trạng thái Ready mà không cần restart app

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi chạy 3 agent và lưu history trong Redis, các agent có thể dùng chung history nên history_length tăng theo các request của cùng một user. Nếu lưu history bằng Python dict thì mỗi container có một dữ liệu riêng, nên khi request chuyển sang container khác, history_length có thể không tăng liên tục mà thay đổi tùy container nhận request


---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy, mình gặp lỗi do public URL trong DEPLOYMENT.md có thêm ký tự "|" (do lúc fill vàpo bảng) ở cuối nên các test tạo URL không hợp lệ. Kiểm tra log và test output để tìm ra URL được tạo sai. Sau đó mình xóa ký tự "|" khỏi URL và chạy lại test để kiểm tra