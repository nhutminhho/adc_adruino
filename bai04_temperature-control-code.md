# Giải Thích Chi Tiết Code Điều Khiển Nhiệt Độ

## 1. Khai Báo và Định Nghĩa Ban Đầu

```cpp
// Include thư viện LCD
#include <LiquidCrystal.h>

// Định nghĩa chân kết nối LED
#define LED_1   8      // LED cảnh báo kênh 1
#define LED_2   9      // LED cảnh báo kênh 2

// Khai báo biến toàn cục cho hệ thống
int adc[2];            // Mảng lưu giá trị ADC
byte i;                // Biến đếm cho vòng lặp
byte TenKenh[2]={A0,A1}; // Các kênh analog đọc nhiệt độ
float NhietDo[2];      // Giá trị nhiệt độ sau chuyển đổi
char Chuoi[17];        // Buffer hiển thị LCD (16 ký tự + null)
char strNhietDo[10];   // Chuỗi chứa giá trị nhiệt độ
char TrangThai[2][5]={"TAT","TAT"}; // Trạng thái các LED

// Cấu hình ngưỡng nhiệt độ
float NhietDoMin[2]={25,26}; // Ngưỡng dưới
float NhietDoMax[2]={30,31}; // Ngưỡng trên

// Cấu hình LCD pins
const int rs = 2, en = 3, d4 = 4, d5 = 5, d6 = 6, d7 = 7;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
```

## 2. Các Hàm Chính

### 2.1 Hàm Setup - Khởi Tạo Hệ Thống
```cpp
void setup() {
    // Khởi tạo LCD kích thước 16x2
    lcd.begin(16, 2);
    
    // Cấu hình LED và trạng thái ban đầu
    pinMode(LED_1, OUTPUT);
    pinMode(LED_2, OUTPUT);
    digitalWrite(LED_1, LOW);
    digitalWrite(LED_2, LOW);
}
```

### 2.2 Hàm Loop - Xử Lý Chính
```cpp
void loop() {
    while(1) {  // Vòng lặp vô hạn
        // Quét qua 2 kênh cảm biến
        for(i=0; i<2; i++) {
            // Đọc và xử lý nhiệt độ
            adc[i] = analogRead(TenKenh[i]);
            // Chuyển đổi ADC sang nhiệt độ
            NhietDo[i] = (adc[i] * 500.0) / 1023.0;
            
            // Kiểm tra và điều khiển
            if(NhietDo[i] > NhietDoMax[i]) {
                _DieuKhien(i, 1);    // Bật LED
                sprintf(TrangThai[i], "BAT");
            }
            else if(NhietDo[i] <= NhietDoMin[i]) {
                _DieuKhien(i, 0);    // Tắt LED
                sprintf(TrangThai[i], "TAT");
            }
        }
        
        // Hiển thị thông tin kênh 1
        lcd.setCursor(0, 0);
        lcd.print("N/D:");
        lcd.print(NhietDo[0], 1);
        sprintf(Chuoi, "%cC, %s", 223, TrangThai[0]); 
        lcd.print(Chuoi);
        
        // Hiển thị thông tin kênh 2
        lcd.setCursor(0, 1);
        lcd.print("N/D:");
        lcd.print(NhietDo[1], 1);
        sprintf(Chuoi, "%cC, %s", 223, TrangThai[1]); 
        lcd.print(Chuoi);
    }
}
```

### 2.3 Hàm Điều Khiển LED
```cpp
void _DieuKhien(byte Pin, byte Value) {
    switch(Pin) {
        case 0:  // Điều khiển LED 1
            digitalWrite(LED_1, Value);
            break;
        case 1:  // Điều khiển LED 2
            digitalWrite(LED_2, Value);
            break;
    }
}
```

## 3. Nguyên Lý Hoạt Động

### 3.1 Chuyển Đổi Nhiệt Độ
```
Công thức chuyển đổi:
1. ADC sang điện áp:
   - 1023 (ADC) = 5000mV
   - x (ADC) = (x * 5000)/1023 mV

2. Điện áp sang nhiệt độ:
   - 10mV = 1°C
   - y mV = y/10 °C

3. Công thức tổng hợp:
   Nhiệt độ = (ADC * 500)/1023 °C
```

### 3.2 Quy Trình Xử Lý
1. Đọc giá trị ADC từ cảm biến
2. Chuyển đổi sang nhiệt độ
3. So sánh với ngưỡng
4. Điều khiển LED
5. Hiển thị lên LCD

## 4. Tối Ưu và Cải Tiến

### 4.1 Cấu Trúc Dữ Liệu
- Sử dụng mảng để quản lý nhiều kênh
- Dễ dàng mở rộng số kênh đo
- Code ngắn gọn và hiệu quả

### 4.2 Hiển Thị LCD
- Format nhiệt độ 1 số thập phân
- Hiển thị ký hiệu độ C (°C)
- Trạng thái LED rõ ràng

### 4.3 Xử Lý
- So sánh với 2 ngưỡng
- Hỗ trợ nhiều kênh đo
- Cập nhật liên tục

## 5. Tips và Mở Rộng

### 5.1 Cải Thiện Code
1. Thêm bộ lọc nhiễu cho ADC
2. Sử dụng timer thay vì delay
3. Thêm EEPROM lưu ngưỡng

### 5.2 Tính Năng Mới
1. Nút nhấn điều chỉnh ngưỡng
2. Cảnh báo âm thanh
3. Giao tiếp Serial/Bluetooth

### 5.3 Debug
1. Kiểm tra giá trị ADC
2. Xác nhận chuyển đổi nhiệt độ
3. Theo dõi trạng thái hệ thống

---
Tài liệu này được tạo cho mục đích học tập và tham khảo.