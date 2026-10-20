# LỘ TRÌNH & TIÊU CHUẨN KỸ THUẬT PHÁT TRIỂN SẢN PHẨM THƯƠNG MẠI (COMMERCIAL-GRADE SOFTWARE)
> **Mục tiêu:** Xây dựng phần mềm thực tế phục vụ khách hàng trả tiền (SaaS / B2B Enterprise / B2C Production), đạt tiêu chuẩn vận hành 24/7/365, chịu lỗi cao, bảo mật pháp lý và có khả năng mở rộng.  
> **Thời gian triển khai:** 6 Tháng (24 Tuần) chia theo 6 giai đoạn chuẩn Product Engineering.

---

## 📌 BẢNG SO SÁNH: "ĐỒ ÁN SINH VIÊN" VS "SẢN PHẨM THƯƠNG MẠI THỰC TẾ"

| Tiêu chuẩn | Đồ án trường học (Làm để chấm điểm) | Sản phẩm thương mại (Khách hàng trả tiền) |
| :--- | :--- | :--- |
| **Mục tiêu hệ thống** | Chạy kịch bản demo 15 phút cho giám khảo | Vận hành liên tục $24/7/365$, Uptime $\ge 99.9\%$, MTTR (thời gian khôi phục sự cố) $< 15$ phút |
| **Kiến trúc dữ liệu** | Single-tenant (dữ liệu các bên nằm chung một bảng sơ sài) | **Multi-Tenant SaaS**: Dữ liệu công ty A tuyệt đối cô lập với công ty B (Schema-per-tenant hoặc Row-level Security) |
| **Xử lý ngoại lệ** | Giả định người dùng luôn nhập đúng | Bắt mọi tình huống: mất mạng giữa chừng, người dùng bấm đúp liên tục, nhập ký tự đặc biệt, tấn công SQLi/XSS |
| **Thanh toán & Đối soát** | Chỉ gọi sandbox tạo mã QR thanh toán thử | Đầy đủ: Webhook có chữ ký bảo mật (HMAC SHA-256), Idempotency Key, Đối soát tự động (Reconciliation), Xuất hóa đơn VAT |
| **Bảo mật & Pháp lý** | Hash mật khẩu cơ bản | Tuân thủ Nghị định 13/2023/NĐ-CP (Bảo vệ dữ liệu cá nhân), mã hóa dữ liệu nhạy cảm AES-256-GCM, Rate Limiting, CORS |
| **Quy trình kiểm thử** | Thử nghiệm bằng tay trên giao diện | **CI/CD Tự động hóa**: Unit Test (Coverage $> 80\%$), Integration Test, E2E Test (Playwright), Stress Test (k6) |
| **Giám sát vận hành** | Chạy `npm run dev` trên localhost, xem log console | **Observability**: Prometheus + Grafana theo dõi tài nguyên, Sentry cảnh báo lỗi tức thì về Telegram/Slack, Loki lưu log tập trung |

---

## 🗺️ LỘ TRÌNH 6 THÁNG (24 TUẦN) TRIỂN KHAI CHI TIẾT

```
Tháng 1: Khảo sát thực tế & Dựng Nền Móng Kiến Trúc (Architecture Foundation)
├── Phỏng vấn 3 - 5 khách hàng tiềm năng để xác định đúng "nỗi đau" (Pain Points)
├── Viết Product Requirements Document (PRD) & Data Flow Diagram (DFD)
├── Thiết kế Clean Architecture / Modular Monolith chuẩn Domain-Driven Design (DDD)
├── Thiết kế CSDL Multi-Tenancy (Row-Level Security trong PostgreSQL)
└── Dựng hạ tầng CI/CD (GitHub Actions: Lint, Format, Build check)

Tháng 2: Xây Dựng Lõi Nghiệp Vụ Chuyên Sâu (Core Business Engine)
├── Lập trình các module nghiệp vụ phức tạp nhất (State Machine, Kế toán kép, Mật mã PKI...)
├── Phân quyền chi tiết RBAC/ABAC (Tenant Owner, Admin, Manager, Staff, Customer)
├── Xây dựng Authentication an toàn: JWT Access Token (15p) + Refresh Token Rotation (lưu Redis)
└── Viết Unit Test & Integration Test cho 100% các API nghiệp vụ cốt lõi

Tháng 3: Tích Hợp AI / Queue / Real-time & Xử Lý Ngoại Lệ
├── Xây dựng cụm Worker xử lý tác vụ nền bất đồng bộ (RabbitMQ / BullMQ)
├── Tích hợp mô hình AI (RAG, Computer Vision, OCR) với cơ chế Fallback an toàn khi AI lỗi
├── Xây dựng hạ tầng Real-time (WebSocket / WebRTC) có cơ chế Auto-Reconnect khi rớt mạng
└── Thiết kế hệ thống Audit Trail bất biến ghi vết toàn bộ hành vi sửa đổi dữ liệu

Tháng 4: Cổng Thanh Toán Thương Mại, Billing & Khách Hàng Thử Nghiệm (Pilot Beta)
├── Tích hợp cổng thanh toán sản phẩm thật (VNPay, MoMo, PayOS, Stripe)
├── Xây dựng module Quản lý Gói cước (SaaS Subscription: Free, Starter, Pro, Enterprise)
├── Triển khai bản Private Beta lên VPS / Cloud (AWS / DigitalOcean / Cloudflare)
└── Mời 2 - 3 khách hàng thực tế vào dùng thử (Pilot Testing) để thu thập phản hồi thực tế

Tháng 5: Tối Ưu Hiệu Năng, Bảo Mật & Hệ Thống Giám Sát (Observability)
├── Cài đặt hệ thống giám sát Prometheus + Grafana theo dõi CPU/RAM/Network
├── Cài đặt Sentry bắt lỗi crash ứng dụng và bắn cảnh báo ngay về Telegram/Slack
├── Tối ưu hóa Database: Đánh chỉ mục (Index tuning), Connection Pooling (PgBouncer)
└── Chạy kiểm thử chịu tải k6 mô phỏng hàng ngàn người dùng đồng thời

Tháng 6: Hoàn Thiện UI/UX, Đóng Gói, Viết Tài Liệu & Go-To-Market
├── Tinh chỉnh UI/UX chỉn chu, chuyên nghiệp, responsive đa thiết bị (Desktop, Tablet, Mobile)
├── Viết tài liệu hướng dẫn sử dụng (User Guide) & Cổng hỗ trợ khách hàng (Help Center)
├── Chuẩn bị hợp đồng dịch vụ SLA (Cam kết chất lượng dịch vụ)
└── Chính thức Open Beta / Ra mắt thương mại phục vụ khách hàng
```

---

## 🛡️ CÁC TIÊU CHUẨN KỸ THUẬT BẮT BUỘC CỦA SẢN PHẨM THẬT

### 1. Kiến trúc Multi-Tenancy (Dành cho nền tảng SaaS B2B)
Không để lẫn lộn dữ liệu giữa các công ty khách hàng. Sử dụng **Row-Level Security (RLS)** trên PostgreSQL:

```sql
-- Kích hoạt Row-Level Security trên bảng orders
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Tạo chính sách: Mỗi tenant chỉ được xem và sửa dữ liệu của chính mình
CREATE POLICY tenant_isolation_policy ON orders
    FOR ALL
    USING (tenant_id = NULLIF(current_setting('app.current_tenant_id', true), '')::UUID);
```

### 2. Cơ chế Idempotency Key khi xử lý giao dịch tiền bạc
Đảm bảo khi khách hàng bấm nút thanh toán nhiều lần hoặc đường truyền mạng bị gửi lặp request (Retry), hệ thống không bao giờ trừ tiền 2 lần:

```typescript
// Middleware kiểm tra Idempotency Key trên Redis
async function idempotencyMiddleware(req, res, next) {
    const idempotencyKey = req.headers['x-idempotency-key'];
    if (!idempotencyKey) return next();

    const cachedResponse = await redis.get(`idempotency:${idempotencyKey}`);
    if (cachedResponse) {
        return res.status(200).json(JSON.parse(cachedResponse));
    }
    // Ghi nhận key đang được xử lý trong 5 phút
    await redis.set(`idempotency:${idempotencyKey}`, 'PROCESSING', 'EX', 300);
    next();
}
```

### 3. Hệ thống Giám sát & Báo động (Observability & Alerting)
- **Sentry Error Tracking:** Mọi lỗi HTTP 500 hoặc ngoại lệ không bắt được (Unhandled Exception) trên Backend/Frontend được tự động chụp lại: Stack trace, thiết bị người dùng, URL xảy ra lỗi, và gửi tin nhắn cảnh báo đỏ về **Telegram Bot** trong vòng 2 giây.
- **Health Check Endpoint:** Expose endpoint `/healthz` kiểm tra kết nối tới: Database, Redis, RabbitMQ. Kubernetes hoặc Cloud Server sẽ ping endpoint này 30 giây/lần; nếu fail 3 lần liên tiếp, hệ thống tự động khởi động lại container mới.

### 4. Quy trình Backup & Khôi phục dữ liệu sau thảm họa (Disaster Recovery)
- **Sao lưu tự động (Automated Backup):** Snapshot database định kỳ mỗi 6 tiếng và lưu trữ mã hóa lên Object Storage khác vùng địa lý (AWS S3 / Cloudflare R2).
- **RPO (Recovery Point Objective):** Tối đa mất mát dữ liệu $\le 1$ giờ.
- **RTO (Recovery Time Objective):** Thời gian phục dựng toàn bộ hệ thống từ file backup $\le 30$ phút.

---

## 📈 CHIẾN LƯỢC TRIỂN KHAI VỚI KHÁCH HÀNG THỬ NGHIỆM (PILOT PROGRAM)

1. **Tuần 14 - 16 (Giai đoạn Alpha):** Mời **1 khách hàng thân thiết nhất** dùng thử phiên bản lõi đầu tiên. Ngồi cạnh khách hàng quan sát họ thao tác để ghi nhận các điểm gây bối rối (UX friction).
2. **Tuần 17 - 20 (Giai đoạn Beta):** Mở rộng cho **3 - 5 khách hàng thật**. Cho phép họ sử dụng miễn phí 3 tháng để đổi lại: Họ đồng ý dùng thử với dữ liệu thật và cung cấp phản hồi cải tiến hàng tuần.
3. **Tuần 21 - 24 (Thương mại hóa):** Chuyển đổi các khách hàng Beta thành khách hàng trả tiền đầu tiên với chính sách ưu đãi Early-bird.
