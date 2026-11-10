# 🛡️ Hệ Thống Cảnh Báo An Ninh Thông Minh IoT (ESP32 + Telegram + ThingSpeak MQTT)

Dự án xây dựng hệ thống giám sát an ninh và phát hiện xâm nhập tự động dựa trên vi điều khiển **ESP32**. Hệ thống sử dụng cảm biến hồng ngoại PIR để phát hiện chuyển động, áp dụng thuật toán lọc nhiễu thời gian 10 giây (`millis()`), đồng thời phát cảnh báo đa kênh: màn hình hiển thị tại chỗ, còi hú, đèn LED, gửi tin nhắn điện thoại qua **Telegram Bot API** và lưu trữ trạng thái thời gian thực lên **ThingSpeak MQTT Broker**.

---

## 🌟 Tính Năng Nổi Bật

- **Lọc nhiễu thời gian thông minh:** Chỉ kích hoạt cảnh báo khi có chuyển động duy trì liên tục từ 10 giây trở lên, giúp loại bỏ tối đa báo động giả (thú cưng, lá cây rung,...).
- **Cảnh báo tại chỗ (Local UI & Alarm):**
  - Màn hình **LCD 20x4** (hoặc tùy chọn **OLED SSD1306**) hiển thị trạng thái hệ thống chi tiết.
  - Hệ thống đèn LED chỉ thị: **LED Xanh** (An toàn), **LED Đỏ** (Cảnh báo).
  - Còi hú **Buzzer 1kHz** phát tín hiệu âm thanh khi xảy ra sự cố.
- **Cảnh báo từ xa (Remote Push Notification):** Gửi tin nhắn tức thời về ứng dụng **Telegram** trên điện thoại cá nhân.
- **Đồng bộ dữ liệu đám mây (Cloud Logging):** Kết nối tới **ThingSpeak MQTT Broker** qua giao thức MQTT để vẽ đồ thị lịch sử cảnh báo thời gian thực.
- **Mô phỏng dễ dàng:** Sẵn sàng chạy mô phỏng trực tuyến trên nền tảng **Wokwi Simulator**.

---

## 📐 Sơ Đồ Nối Dây (Pinout Connection)

| Linh kiện | Chân linh kiện | Chân ESP32 | Chức năng / Ghi chú |
| :--- | :--- | :--- | :--- |
| **Cảm biến PIR (HC-SR501)** | `VCC` | `5V` / `VIN` | Nguồn cấp 5V |
| | `OUT` (Signal) | `GPIO 13` | Đọc tín hiệu chuyển động (HIGH/LOW) |
| | `GND` | `GND` | Chân mát chung |
| **Màn hình LCD 20x4 (I2C)** | `VCC` | `5V` | Nguồn cấp màn hình |
| | `GND` | `GND` | Chân mát chung |
| | `SDA` | `GPIO 21` | Dữ liệu I2C |
| | `SCL` | `GPIO 22` | Xung nhịp I2C |
| **Đèn LED Xanh** | Anode (`+`) | `GPIO 25` (qua trở 220Ω) | Báo trạng thái **AN TOÀN** |
| **Đèn LED Đỏ** | Anode (`+`) | `GPIO 26` (qua trở 220Ω) | Báo trạng thái **CẢNH BÁO** |
| **Còi Cảnh Báo (Buzzer)** | Positive (`+`) | `GPIO 27` | Phát âm thanh báo động |

---

## 📚 Thư Viện Cần Thiết (Arduino IDE / Wokwi)

Vào **Library Manager** để thêm các thư viện sau:
1. `WiFi.h` & `HTTPClient.h` *(Có sẵn trong ESP32 Core)*
2. `Wire.h` *(Có sẵn trong ESP32 Core)*
3. **`LiquidCrystal I2C`** *(Nếu dùng LCD 20x4)* hoặc **`Adafruit SSD1306`** & **`Adafruit GFX`** *(Nếu dùng OLED)*
4. **`PubSubClient`** *(Dùng cho giao thức MQTT)*

---

## ⚙️ Hướng Dẫn Cấu Hình Thông Số

Trong file `sketch.ino`, cập nhật các thông tin cá nhân của bạn tại các dòng sau:

### 1. Cấu hình Telegram Bot
```cpp
String botToken = "YOUR_TELEGRAM_BOT_TOKEN"; // Lấy từ @BotFather
String chatId   = "YOUR_TELEGRAM_CHAT_ID";   // Lấy từ @userinfobot
```

### 2. Cấu hình ThingSpeak MQTT
```cpp
const char* clientID  = "YOUR_MQTT_CLIENT_ID"; // Lấy từ Tab MQTT Devices trên ThingSpeak
const char* mqttUser  = "YOUR_MQTT_USERNAME";  
const char* mqttPass  = "YOUR_MQTT_PASSWORD";  
const char* channelID = "YOUR_CHANNEL_ID";      // ID Kênh ThingSpeak của bạn
```

---

## 🔄 Luồng Hoạt Động Của Chương Trình

```
[Chờ Chuyển Động]
       │
       ▼ (PIR = HIGH)
[Bắt đầu đếm thời gian: motionStartTime = millis()]
       │
       ├──── (Người rời đi < 10s) ───► [Reset biến đếm, giữ An toàn]
       │
       ▼ (Duy trì PIR = HIGH >= 10s)
[KÍCH HOẠT CHUỖI CẢNH BÁO]
       ├── 1. Màn hình LCD/OLED: Chuyển sang "CANH BAO! CO NGUOI BEN NGOAI"
       ├── 2. Phần cứng: Đèn Đỏ SÁNG, Đèn Xanh TẮT, Còi hú kêu 1kHz
       ├── 3. Telegram: Gửi tin nhắn "CANH BAO: Phat hien co nguoi..."
       └── 4. ThingSpeak MQTT: Publish giá trị `field1=1`
       │
       ▼ (Người rời đi, PIR = LOW)
[KHÔI PHỤC TRẠNG THÁI]
       ├── Màn hình & Đèn LED trở về "AN TOÀN"
       └── ThingSpeak MQTT: Publish giá trị `field1=0`
```

---

## 🚀 Cách Chạy Mô Phỏng Trên Wokwi

1. Tạo dự án mới trên [Wokwi.com](https://wokwi.com) chọn bo mạch **ESP32 DevKit V1**.
2. Sao chép nội dung cấu hình phần cứng vào tab `diagram.json`.
3. Cài đặt các thư viện bắt buộc trong **Library Manager**.
4. Dán mã nguồn vào `sketch.ino`.
5. Nhấn nút **Play (Mô phỏng)**.
6. Click vào cảm biến PIR, bấm **Simulate motion** và giữ quá 10 giây để kiểm tra kết quả cảnh báo gửi về Telegram và ThingSpeak.

---

## 📝 Giấy Phép & Tác Giả

- **Tác giả:** Đồ án IoT Cá nhân
- **Môi trường thử nghiệm:** Wokwi Simulator / Arduino IDE
- **License:** MIT License
