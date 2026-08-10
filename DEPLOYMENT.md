# Thông Tin Deploy — Checkpoint 5

> Service đã được deploy trên Render. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Trọng Đăng Khoa |
| Mã học viên | 2A202601964 |
| Repo | https://github.com/Khoa15/K4-Day12-2A202601964-NguyenTrongDangKhoa |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-1zke.onrender.com |
| Platform | Render |
| Ngày deploy | 10/08/2026 |
| Health check | `/healthz` |
| Readiness check | `/readyz` |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của Render |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Đặt token trong biến môi trường local trước khi kiểm tra. Không ghi token vào
file này hoặc commit file `.env`:

```bash
export API_TOKEN="<token-da-set-tren-render>"
export BASE_URL="https://day12-chat-1zke.onrender.com"
```

Các lệnh kiểm tra:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i $BASE_URL/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i $BASE_URL/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST $BASE_URL/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST $BASE_URL/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST $BASE_URL/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Kiểm Tra

| Kiểm tra | Kết quả mong đợi |
|----------|------------------|
| `GET /healthz` | `200` — service đang hoạt động |
| `GET /readyz` | `200` — service đã kết nối Redis |
| `POST /chat` không có token | `401` — yêu cầu Bearer token |
| `POST /chat` với token hợp lệ | `200` — trả về câu trả lời |
| Gọi vượt rate limit | `429` — có header `Retry-After` |

Không ghi API token hoặc dữ liệu nhạy cảm vào tài liệu này.

## Ảnh Chụp Màn Hình

Ảnh minh chứng được lưu trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không sử dụng phương án dự phòng. Nếu Render tạm thời không khả dụng, có thể
chạy local bằng Docker với các lệnh sau:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
Không commit `.env` và không đưa secret vào repository.
