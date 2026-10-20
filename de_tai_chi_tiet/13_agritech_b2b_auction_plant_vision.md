# PHÂN TÍCH CHI TIẾT ĐỀ TÀI: AGRITECH DATA-DRIVEN PLATFORM
## (Sàn Đấu Giá Nông Sản Trực Tuyến Thời Gian Thực & AI Chẩn Đoán Bệnh Cây Trồng Qua Ảnh)

---

## 1. TỔNG QUAN & BÀI TOÁN THỰC TẾ

### 1.1. Bối cảnh
Nông nghiệp Việt Nam (sầu riêng, thanh long, cà phê, lúa gạo) đối mặt với 2 rủi ro lớn nhất:
- **Nông dân bị thương lái ép giá:** Do thiếu kênh kết nối trực tiếp với chuỗi siêu thị, nhà thu mua xuất khẩu và sàn đấu giá công khai. Nông sản thường xuyên rơi vào cảnh *"được mùa mất giá"*.
- **Thiệt hại mùa màng do phát hiện sâu bệnh muộn:** Khi cây bị nấm lá, đốm mắt cua, rệp sáp, nông dân không có chuyên gia nông học bên cạnh để chẩn đoán chính xác, thường mua sai thuốc bảo vệ thực vật khiến cây chết hoặc tồn dư hóa chất vượt ngưỡng xuất khẩu.

### 1.2. Giải pháp của dự án
1. **Sàn đấu giá nông sản trực tuyến (Real-Time B2B Live Auction):**
   - Hợp tác xã/nông dân đăng tải lô hàng (ví dụ: 5 tấn Sầu riêng Ri6 chuẩn VietGAP).
   - Phiên đấu giá diễn ra theo thời gian thực (Real-time WebSocket): Nhà mua sỉ vào trả giá trực tiếp, có đồng hồ đếm ngược (Countdown Timer) và cơ chế khóa tiền cọc đảm bảo giao dịch.
2. **AI Computer Vision chẩn đoán bệnh cây trồng:**
   - Người dùng chỉ cần cầm điện thoại chụp ảnh lá cây hoặc quả bị bệnh -> Mô hình Deep Learning nhận diện chính xác tên bệnh trong 0.3 giây, đưa ra nguyên nhân và phác đồ điều trị an toàn sinh học.
3. **Cảnh báo thời tiết nông vụ (Weather Integration):**
   - Kết nối API khí tượng để cảnh báo sương muối, mưa đá, hạn mặn và gợi ý lịch phun thuốc tối ưu.

---

## 2. SƠ ĐỒ KIẾN TRÚC HỆ THỐNG

```mermaid
graph TD
    subgraph Client_Applications["1. Giao Diện Người Dùng (Next.js + Tailwind + Mobile Web)"]
        AuctionRoom["Phòng Đấu Giá Trực Tiếp (Live Bidding Room)"]
        PlantDoctor["Bác Sĩ Cây Trồng (Chụp ảnh & Nhận diện bệnh)"]
        Marketplace["Chợ Nông Sản B2B & Quản Lý Đơn Hàng"]
    end

    subgraph Realtime_Auction_Engine["2. Bộ Máy Đấu Giá Thời Gian Thực (Go / Node.js WebSocket)"]
        RoomManager["Quản Lý Phòng Đấu Giá & Đồng Hồ Đếm Ngược"]
        BidHandler["Bộ Xử Lý Bước Giá & Khóa Đặt Cược (Atomic Bidding)"]
        RedisCluster[("Redis - Lưu Giá Hiện Tại & Pub/Sub")]
        RoomManager <--> RedisCluster
        BidHandler <--> RedisCluster
    end

    subgraph Core_Backend["3. Backend API & Quản Lý Giao Dịch (PostgreSQL)"]
        OrderService["Quản Lý Đơn Hàng & Ký Quỹ Đấu Giá (Escrow Payment)"]
        Traceability["Truy Xuất Nguồn Gốc Lô Hàng VietGAP (Mã QR Động)"]
        Postgres[(PostgreSQL)]
        OrderService <--> Postgres
        Traceability <--> Postgres
        BidHandler -->|Chốt phiên đấu giá| OrderService
    end

    subgraph AI_Plant_Doctor["4. AI Computer Vision Service (PyTorch / FastAPI)"]
        VisionModel["Mô Hình Phân Loại Bệnh Lá (YOLOv8 / EfficientNet)"]
        DiseaseKB[("Cơ Sở Tri Thức Bệnh Nông Nghiệp & Phác Đồ Xử Lý")]
        PlantDoctor -->|Tải ảnh lá bệnh| VisionModel
        VisionModel <--> DiseaseKB
    end

    AuctionRoom <-->|WebSocket Realtime| Realtime_Auction_Engine
    Marketplace <--> Core_Backend
```

---

## 3. THIẾT KẾ ĐỘNG CƠ ĐẤU GIÁ REAL-TIME (AUCTION ENGINE)

Đấu giá trực tuyến đòi hỏi tính nhất quán tuyệt đối về thời gian và giá trị đặt cược:

```mermaid
sequenceDiagram
    autonumber
    actor BidderA as Người Mua A (Siêu thị)
    actor BidderB as Người Mua B (Xuất khẩu)
    participant WS as WebSocket Server
    participant Redis as Redis (Atomic State)
    participant DB as PostgreSQL (Lưu lịch sử)

    Note over WS,Redis: Lô hàng: 5 tấn Sầu riêng | Giá khởi điểm: 80k/kg | Bước giá: 2k
    BidderA->>WS: Trả giá 82.000 đ/kg
    WS->>Redis: Thực thi lệnh kiểm tra: NewBid > CurrentBid ?
    Redis-->>WS: Hợp lệ -> Cập nhật CurrentPrice = 82k, Leader = A
    WS-->>BidderA: Phát thanh WebSocket (Broadcast): "A dẫn đầu với 82k"
    WS-->>BidderB: Cập nhật giá trên màn hình của B
    WS->>WS: Reset đồng hồ đếm ngược về 30 giây

    BidderB->>WS: Trả giá 84.000 đ/kg (ở giây thứ 28)
    WS->>Redis: Cập nhật CurrentPrice = 84k, Leader = B
    WS-->>BidderA: "B đã trả 84k! Bạn có muốn trả giá cao hơn?"
    WS->>WS: Reset đồng hồ đếm ngược lại 30 giây
    Note over WS: Hết 30 giây không ai trả thêm -> Chốt đơn cho B!
    WS->>DB: Ghi nhận người thắng cuộc B với giá 84k/kg
```

---

## 4. PIPELINE NHẬN DIỆN BỆNH CÂY TRỒNG (COMPUTER VISION)

1. **Bộ dữ liệu (Dataset):**
   - Sử dụng bộ dữ liệu mở **PlantVillage Dataset** (hơn 54.000 ảnh chụp lá cây của 14 loài cây trồng khác nhau với 38 lớp bệnh lý: bệnh đạo ôn, đốm mắt cua, rỉ sắt, thán thư, xoăn lá do virus...).
2. **Huấn luyện mô hình:**
   - Fine-tune trên kiến trúc mạng **EfficientNet-B0** hoặc **MobileNetV3** (nhẹ, tối ưu để chạy trực tiếp trên web hoặc điện thoại di động).
   - Áp dụng các kỹ thuật tăng cường dữ liệu (Data Augmentation): Lật ảnh, xoay góc, chỉnh độ sáng/tương phản để mô hình nhận diện chính xác kể cả khi chụp ngoài trời nắng gắt hoặc trong bóng râm.
3. **Độ chính xác (Accuracy):**
   - Đạt độ chính xác Top-1 $> 94\%$ trên tập kiểm thử (Test set).
   - Tốc độ suy luận (Inference Time): **dưới 100ms**.
