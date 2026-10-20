# PHÂN TÍCH CHI TIẾT ĐỀ TÀI 03: COLD-CHAIN FRESH FOOD E-COMMERCE
## (Sàn Bán Thực Phẩm Tươi Sống & Giám Sát Chuỗi Cung Ứng Lạnh IoT + AI Dự Báo Hạn Dùng)

---

## 1. TỔNG QUAN & BÀI TOÁN KINH DOANH THỰC TẾ

### 1.1. Bối cảnh
Khi mua sắm thực phẩm tươi sống (thịt bò Wagyu, hải sản đông lạnh, hoa quả nhập khẩu) hoặc dược phẩm (vaccine, insulin) qua mạng:
- Khách hàng luôn e ngại: *"Liệu trên đường vận chuyển, shipper có tắt máy lạnh để tiết kiệm xăng làm thực phẩm bị ôi thiu không?"*
- Doanh nghiệp thiếu công cụ giám sát chất lượng bảo quản theo thời gian thực xuyên suốt chuỗi cung ứng (từ kho đến xe tải và đến tay khách).

### 1.2. Giải pháp của dự án
1. **Sàn thương mại điện tử chuyên thực phẩm chất lượng cao (Web & Mobile):**
   - Đặt hàng, thanh toán, chọn khung giờ giao hàng (Delivery Slot).
   - **Tính năng minh bạch chất lượng (Quality Transparency):** Trên chi tiết mỗi đơn hàng, khách hàng có thể xem **Biểu đồ nhiệt độ & độ ẩm lịch sử** của chính kiện hàng đó từ khi đóng gói cho đến lúc giao.
2. **Hệ thống giám sát IoT (Cold-Chain Telemetry):**
   - Thiết bị IoT nhỏ gọn đặt bên trong thùng bảo ôn/kho lạnh: đo nhiệt độ, độ ẩm liên tục và cảnh báo tức thời nếu phát hiện phá vỡ ngưỡng bảo quản an toàn.
3. **Mô hình AI dự báo:**
   - Dự báo độ tươi sống còn lại (Remaining Shelf-Life) và dự báo nhu cầu nhập hàng (Demand Forecasting).

---

## 2. SƠ ĐỒ KIẾN TRÚC HỆ THỐNG

```mermaid
graph TD
    subgraph IoT_Sensors["Lớp Cảm Biến IoT Trong Kho & Thùng Hàng"]
        SHT30["Cảm biến Nhiệt độ / Độ ẩm SHT30 / DHT22"]
        MQ135["Cảm biến Khí Amoniac / Hôi thối MQ-135"]
        ESP32["Vi điều khiển ESP32 (Gửi dữ liệu 10s/lần)"]
        SHT30 --> ESP32
        MQ135 --> ESP32
    end

    subgraph Communication_Data["Tầng Thu Nhận Dữ Liệu Thời Gian Thực"]
        MQTT["MQTT Broker (Mosquitto / EMQX)"]
        Telegraf["Telegraf / Ingestion Worker"]
        TimescaleDB[("Time-Series DB (TimescaleDB / InfluxDB)")]
        Postgres[("Relational DB (PostgreSQL - Đơn hàng, SP)")]
        ESP32 -->|MQTT Payload JSON| MQTT
        MQTT --> Telegraf
        Telegraf --> TimescaleDB
    end

    subgraph AI_Analytics["Lớp Phân Tích & AI (Python FastAPI)"]
        Anomaly["Anomaly Detection (Phát hiện nhiệt độ tăng đột biến)"]
        ShelfLife["Shelf-Life Degradation Model (Công thức Arrhenius & ML)"]
        TimescaleDB --> Anomaly
        TimescaleDB --> ShelfLife
    end

    subgraph Presentation["Tầng Ứng Dụng (Next.js / Flutter)"]
        CustomerWeb["Web Khách Hàng (Xem biểu đồ nhiệt độ đơn hàng)"]
        AdminWarehouse["Dashboard Giám sát kho lạnh thời gian thực"]
        AlertService["Hệ thống cảnh báo khẩn cấp (Telegram Bot / SMS)"]
        Anomaly --> AlertService
        TimescaleDB --> AdminWarehouse
        Postgres --> CustomerWeb
    end
```

---

## 3. BẢNG LINH KIỆN IOT & SƠ ĐỒ KẾT NỐI

| Linh kiện | Vai trò kỹ thuật | Giá ước lượng |
| :--- | :--- | :---: |
| **ESP32 NodeMCU** | Thu thập dữ liệu cảm biến, kết nối Wi-Fi/4G gửi MQTT | ~80.000 VNĐ |
| **Cảm biến SHT30 (hoặc DHT22)** | Đo nhiệt độ (-40°C đến 125°C) và độ ẩm (0-100% RH) độ chính xác cao | ~45.000 VNĐ |
| **Cảm biến khí MQ-135** | Đo nồng độ các chất khí bay hơi hữu cơ (NH3, Alcohol) khi thịt cá bị ôi thiu | ~35.000 VNĐ |
| **Pin sạc 18650 + Mạch sạc TP4056** | Cấp nguồn độc lập cho hộp bảo quản không cần cắm dây | ~40.000 VNĐ |
| **Hộp xốp hoặc Hộp nhựa bảo ôn mini** | Mô phỏng thùng hàng giao thực phẩm lạnh | ~20.000 VNĐ |
| **TỔNG CHI PHÍ** | | **~220.000 VNĐ** |

---

## 4. MÔ HÌNH TOÁN HỌC & AI TRONG DỰ ÁN

### 4.1. Thuật toán phân tích suy giảm độ tươi sống (Kinetic Degradation Model)
Chất lượng thực phẩm suy giảm theo hàm nhiệt độ dựa trên **Định luật Arrhenius**:
$$k = A \cdot e^{-\frac{E_a}{R \cdot T}}$$
- Trong đó $T$ là nhiệt độ thực tế đo từ cảm biến IoT theo từng phút.
- Hệ thống tích phân tỷ lệ suy giảm $k$ theo chuỗi thời gian để tính ra **"Chỉ số tươi ngon (Freshness Index %)"**:
  - Khi bắt đầu đóng gói: $100\%$.
  - Nếu nhiệt độ duy trì chuẩn dưới 4°C: Tươi ngon giữ được 5 ngày.
  - Nếu bị gián đoạn nhiệt độ (lên 20°C trong 2 tiếng): Hệ thống lập tức giảm điểm tươi ngon xuống $65\%$ và cảnh báo trên giao diện khách hàng.

### 4.2. Cảnh báo tự động đa kênh (Instant Alerts)
Khi cảm biến vượt ngưỡng nguy hiểm (ví dụ: nhiệt độ tủ đông > -10°C trong 15 phút liên tục), backend tự động gửi tin nhắn báo động qua **Telegram Bot / Webhook** để nhân viên kho kiểm tra ngay hệ thống làm lạnh.
