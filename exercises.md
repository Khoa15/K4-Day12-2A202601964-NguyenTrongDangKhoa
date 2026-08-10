# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ, khi deploy lên Railway, tôi quên khai báo `API_TOKEN`. Với cấu hình
> fail fast, Pydantic báo lỗi ngay lúc khởi động và container không được xem là
> healthy. Nhờ vậy tôi biết ngay secret đang thiếu và không vô tình chạy API
> với token công khai như `changeme`. Nếu có giá trị mặc định, app vẫn chạy,
> đến khi bị gọi ngoài Internet tôi mới phát hiện vấn đề và có thể đã phát
> sinh chi phí hoặc bị truy cập trái phép.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi quan sát được có dạng:
>
> ```json
> {"event": "chat_completed", "severity": "INFO", "ts": "2026-08-10T10:15:20.123456+00:00", "client_id": "sv01", "prompt_tokens": 12, "completion_tokens": 18, "usd_cost": 0.0001}
> ```
>
> Thứ nhất, tôi có thể lọc và đếm tự động số sự kiện `chat_completed` theo
> `client_id`, thời gian hoặc mức `severity` để làm dashboard/cảnh báo. Thứ
> hai, tôi có thể cộng trường `usd_cost` để theo dõi chi phí và phát hiện một
> client gọi bất thường. Một câu `print` chỉ là chuỗi cho người đọc, không có
> các trường ổn định để hệ thống log truy vấn và tổng hợp chính xác.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng 1.8 GB |
| Multi-stage | dưới 400 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage chứa cả base image đầy đủ, bộ công cụ build, cache/package
> tạm và toàn bộ dependency dùng trong quá trình cài đặt. Multi-stage chỉ
> mang phần dependency đã cài cùng source sang runtime image `python:3.11-slim`,
> nên loại bỏ compiler, cache và file trung gian. Vì Docker Desktop đang ở
> trạng thái paused nên tôi chưa đo được số MB thực tế; các số trên là mức đích
> được ghi trong đề bài, không phải số đo local.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa một ký tự trong `app/main.py`, layer cài dependency vẫn được
> dùng lại vì `requirements.txt` chưa thay đổi. Layer `COPY app ./app` phải
> chạy lại, sau đó các layer phía sau nó cũng được tạo lại. Nếu đặt `COPY . .`
> trước `RUN pip install`, mọi thay đổi source sẽ làm layer COPY thay đổi và
> Docker phải chạy lại `pip install` dù requirements không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro là: lỗ hổng cho phép kẻ tấn công thực thi lệnh trong process
> Python; process đó đang chạy bằng root nên lệnh có quyền đọc/sửa nhiều file
> trong container; nếu container runtime có cấu hình nguy hiểm hoặc có thêm
> lỗ hổng escape, quyền root trong container có thể dẫn tới quyền cao trên
> host. Lệnh `USER appuser` cắt chuỗi ngay ở bước process bị chiếm quyền:
> process chỉ có quyền của user thường, không thể tùy ý sửa file hệ thống,
> cài package hay quản trị dịch vụ. Đây không thay thế việc vá lỗ hổng nhưng
> làm giảm đáng kể hậu quả khi lỗ hổng bị khai thác.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> `WWW-Authenticate: Bearer` là yêu cầu của chuẩn HTTP cho response 401 và
> cho client biết resource yêu cầu cơ chế xác thực Bearer; nhờ đó client hoặc
> thư viện chuẩn biết cách gửi lại header `Authorization: Bearer <token>`.
> Tôi trả cùng một lỗi cho thiếu header, sai scheme và sai token để không làm
> lộ thông tin cho người đang dò API. Nếu trả lời “scheme đúng nhưng token
> sai” hoặc “token gần đúng”, kẻ tấn công có thể dùng các phản hồi khác nhau
> để dò cấu trúc và trạng thái secret.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Sau 10 phút, xô vẫn chỉ có tối đa 10 token vì `available()` luôn giới hạn
> bằng `min(capacity, ...)`. Vì vậy client gửi liên tiếp được 10 request rồi
> request tiếp theo nhận 429 (giả sử không có request nào khác dùng chung
> client). Nếu bỏ `min(capacity, ...)`, tốc độ nạp là 10 token/phút nên sau
> 10 phút xô có 110 token: 10 token ban đầu cộng 100 token được nạp thêm.
> Khi đó client có thể gửi 110 request liên tiếp, trái với giới hạn burst
> tối đa là 10.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD/tháng, sự cố bắt đầu lúc 2h sáng có thể tiêu gần toàn bộ
> 30 USD trước khi bị chặn; hạn mức chỉ tự đặt lại khi sang tháng mới. Với
> hạn mức 1 USD/ngày, thiệt hại tối đa trong ngày đó là khoảng 1 USD (thực tế
> có thể dừng ở request làm vượt ngưỡng tùy cách ước lượng/ghi nhận chi phí),
> và key chi tiêu mới được dùng khi sang ngày UTC kế tiếp. Vì vậy service tự
> hồi phục vào ngày hôm sau, không cần chờ hết tháng hay xóa thủ công ngân
> sách. Hạn mức ngày hy sinh một phần khả năng dùng dồn ngân sách nhưng cô
> lập sự cố tốt hơn.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint và endpoint chung kiểm tra Redis, khi Redis mất kết nối
> sẽ xảy ra theo thứ tự: (1) health check của cả ba container gọi endpoint;
> (2) cả ba đều nhận lỗi Redis và trả trạng thái không healthy; (3) load
> balancer/orchestrator loại cả ba container khỏi pool hoặc đánh dấu chúng
> cần restart; (4) vì không còn instance healthy, người dùng nhận lỗi 503/502;
> (5) trong 30 giây Redis hồi phục, các container có thể bị restart không cần
> thiết và tạo ra downtime. Tách `/healthz` nhẹ khỏi `/readyz` giúp process
> vẫn được xem là sống, còn từng container chỉ bị rút khỏi traffic cho đến
> khi `/readyz` kiểm tra Redis thành công.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Một lỗi tôi gặp khi deploy cloud là health check timeout vì cloud cấp cổng
> qua biến môi trường `$PORT`, trong khi service ban đầu luôn lắng nghe ở
> cổng 8000. Tôi kiểm tra log deploy và thấy process khởi động thành công
> nhưng probe gọi tới cổng được platform cấp không kết nối được. Tôi sửa
> command khởi động thành `uvicorn app.main:app --host 0.0.0.0 --port
> ${PORT:-8000}`, đồng thời để health check gọi `/healthz`. Sau khi deploy
> lại, log cho thấy app bind đúng cổng và health check chuyển sang healthy.
