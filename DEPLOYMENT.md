# Thông Tin Deploy — Checkpoint 5

Service FastAPI của bài lab đã được triển khai công khai trên Railway bằng Dockerfile trong repository. Redis được chạy dưới dạng service riêng trong cùng Railway project và ứng dụng truy cập qua biến tham chiếu nội bộ.

> Tài liệu này chỉ ghi tên và nguồn của biến môi trường. Giá trị API key thật không được lưu trong repository.

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Đào Thị Trang |
| Mã học viên | 2A202601809 |
| Repository | [TrangDaoThi673/K3-DAY12-2A202601809-DaoThiTrang](https://github.com/TrangDaoThi673/K3-DAY12-2A202601809-DaoThiTrang) |

## Thông Tin Service

| Mục | Nội dung |
|---|---|
| Public URL | https://day12-agent-production-9a30.up.railway.app |
| Platform | Railway |
| Môi trường | production |
| Ngày deploy | 10/08/2026 |
| App service | `day12-agent` |
| Data service | `day12-redis` |

## Biến Môi Trường Trên Cloud

| Biến | Trạng thái | Nguồn/Cấu hình |
|---|---|---|
| `PORT` | Đã set | Railway tự cấp tại runtime; ứng dụng đọc qua `$PORT` |
| `AGENT_API_KEY` | Đã set | Railway Variables; giá trị bí mật không nằm trong source code |
| `REDIS_URL` | Đã set | Reference tới `day12-redis.REDIS_URL` trong Railway project |
| `RATE_LIMIT_PER_MINUTE` | Đã set | `10` request/phút/user |
| `MONTHLY_BUDGET_USD` | Đã set | `10.0` USD/user/tháng |
| `LOG_LEVEL` | Đã set | `INFO` |

## Kiểm Tra Bản Deploy

Public URL được kiểm tra từ PowerShell bằng các request sau:

```powershell
$deployUrl = "https://day12-agent-production-9a30.up.railway.app"

curl.exe -i "$deployUrl/health"
curl.exe -i "$deployUrl/ready"

$unauthorizedBody = @{ question = "Hello" } | ConvertTo-Json
try {
    Invoke-WebRequest -UseBasicParsing -Uri "$deployUrl/ask" -Method POST `
      -ContentType "application/json" -Body $unauthorizedBody
} catch {
    $_.Exception.Response.StatusCode.value__  # 401
}

# $deployKey được đọc từ secret cục bộ, không được ghi vào repository.
$headers = @{
    "X-API-Key" = $deployKey
    "X-User-Id" = "sv01"
}
$body = @{ question = "Deploy la gi?" } | ConvertTo-Json
$response = Invoke-WebRequest -UseBasicParsing `
  -Uri "$deployUrl/ask" `
  -Method POST `
  -Headers $headers `
  -ContentType "application/json" `
  -Body $body
```

## Kết Quả Chạy Thật

| Request | HTTP status | Kết quả quan sát |
|---|---:|---|
| `GET /health` | 200 | `{"status":"ok","service":"day12-agent","version":"1.0.0"}` |
| `GET /ready` | 200 | `{"status":"ready","redis":true}` |
| `POST /ask` không có API key | 401 | `{"detail":"invalid or missing API key"}` |
| `POST /ask` có API key hợp lệ | 200 | Response có `answer`, `user_id`, `history_length`, `cost_usd` và `tokens` |

Kết quả trên xác nhận service hoạt động qua HTTPS, Redis đã sẵn sàng, endpoint chính được bảo vệ bằng API key và request hợp lệ nhận được câu trả lời từ agent.

## Bằng Chứng

- [Dashboard Railway: agent và Redis đều Online](screenshots/dashboard.png)
- [Liveness và readiness đều trả 200](screenshots/health-ready.png)
- [`/ask` không có API key trả 401](screenshots/ask-401.png)
- [`/ask` có API key hợp lệ trả 200](screenshots/ask-200.png)

Các ảnh không hiển thị giá trị thật của `AGENT_API_KEY` hoặc `DEPLOY_API_KEY`.
