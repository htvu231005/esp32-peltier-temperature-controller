# Hệ thống Điều khiển Nhiệt độ Peltier (Peltier PID Temperature Controller)

Dự án giám sát và điều khiển nhiệt độ vòng kín (Closed-loop) sử dụng vi điều khiển ESP32 và framework ESP-IDF, tích hợp thuật toán PID và giao thức MQTT để điều khiển qua Web.

## 1. Yêu cầu Hệ thống (System Requirements)

* **Vi điều khiển:** ESP32 (sử dụng ESP-IDF framework)
* **Cảm biến:** SHT31 (giao tiếp I2C) để lấy dữ liệu nhiệt độ phản hồi.
* **Cơ cấu chấp hành:** Sò nóng lạnh Peltier, được băm xung (PWM) thông qua module cầu H (DRV8833).
* **Giao thức mạng:** Wi-Fi & MQTT Client.

## 2. Sơ đồ kết nối phần cứng (Pinout)

| Thiết bị / Module | Chân ESP32                         | Ghi chú cấu hình                        |
| :---------------- | :--------------------------------- | :-------------------------------------- |
| **SHT31 (I2C)**   | `SDA: GPIO 8` <br> `SCL: GPIO 9`   | I2C Master, tốc độ 100 kHz, Pull-up nội |
| **DRV8833 (PWM)** | `Lạnh: GPIO 4` <br> `Nóng: GPIO 5` | Tần số 18 kHz, độ phân giải 12-bit      |

## 3. Kiến trúc Phần mềm & Thuật toán

Dự án được thiết kế xoay quanh FreeRTOS và các cơ chế bảo vệ phần cứng:

* **Bộ điều khiển PID:** Vận hành với tập thông số cấu hình sẵn (`K_p = 2000`, `K_i = 25`, `K_d = 5`). Tích hợp logic chống tích lũy sai số (Anti-windup) khi vượt quá giới hạn ±2.0 °C và giới hạn giá trị integral trong khoảng ±200.

* **Cơ chế bảo vệ Sò Peltier (Dead-time):** Thuật toán tự động chèn khoảng trễ 5 giây (`vTaskDelay(5000)`) khi đảo chiều dòng điện giữa chế độ LÀM LẠNH và LÀM NÓNG để tránh sốc nhiệt và hỏng hóc phần cứng.

* **Fail-safe Mode:** Nếu giao tiếp I2C với SHT31 thất bại, hệ thống tự động đưa nhiệt độ về -45 °C, ngắt xung PWM cấp cho hệ thống và reset sai số tích lũy để bảo vệ an toàn.

* **Giao tiếp MQTT:** Gửi chuỗi định dạng JSON chứa trạng thái thực (Nhiệt độ, Setpoint, PWM, Chế độ) với tần số 1 Hz tới topic `/hethong/temp_control`. Lắng nghe và cập nhật Setpoint từ topic `/hethong/set_from_web`.

## 4. Hướng dẫn Biên dịch (Build & Flash)

Dự án sử dụng thư viện kết nối Wi-Fi mẫu của ESP-IDF (`protocol_examples_common.h`).

```bash
# 1. Cấu hình Wi-Fi (SSID/Password) trong Example Configuration
idf.py menuconfig

# 2. Biên dịch và nạp firmware
idf.py build
idf.py -p [PORT] flash monitor
```
