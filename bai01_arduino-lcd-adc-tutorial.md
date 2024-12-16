# Hướng dẫn Chi Tiết: Arduino với LCD và ADC

## Mục lục
1. [Giới thiệu](#1-giới-thiệu)
2. [Lý thuyết cơ bản](#2-lý-thuyết-cơ-bản)
3. [Phần cứng](#3-phần-cứng)
4. [Phân tích code](#4-phân-tích-code)
5. [Thực hành](#5-thực-hành)
6. [Bài tập và ứng dụng](#6-bài-tập-và-ứng-dụng)
7. [Troubleshooting](#7-troubleshooting)

## 1. Giới thiệu

### 1.1 Mục tiêu bài học
- Hiểu về ADC trong Arduino
- Sử dụng LCD hiển thị thông tin
- Xử lý tín hiệu analog
- Điều khiển LED theo giá trị analog

### 1.2 Yêu cầu kiến thức
- Kiến thức cơ bản về Arduino
- Hiểu về điện áp analog và digital
- Biết cách đấu nối cơ bản

## 2. Lý thuyết cơ bản

### 2.1 ADC (Analog to Digital Converter)
#### a) Nguyên lý hoạt động
- ADC chuyển đổi điện áp analog thành giá trị số
- Arduino UNO sử dụng ADC 10-bit
- Công thức: 
```
Digital = (Analog × 1024) / 5V
```

#### b) Đặc điểm ADC trong Arduino
- Độ phân giải: 10 bit (0-1023)
- Thời gian chuyển đổi: ~100μs
- Điện áp tham chiếu: 5V
- Độ phân giải điện áp: 4.88mV/bit

### 2.2 LCD 16x2
#### a) Đặc điểm
- 16 ký tự × 2 dòng
- Giao tiếp 4-bit hoặc 8-bit
- Điện áp hoạt động: 5V
- Có đèn nền

#### b) Các chân LCD
1. Chân điều khiển:
   - RS: Chọn thanh ghi (0:lệnh, 1:dữ liệu)
   - E: Enable (xung clock)
   - RW: Read/Write (thường nối GND)

2. Chân dữ liệu:
   - Mode 4-bit: Sử dụng D4-D7
   - Mode 8-bit: Sử dụng D0-D7

## 3. Phần cứng

### 3.1 Danh sách thiết bị
1. Arduino UNO R3
2. LCD 16x2
3. Biến trở 10kΩ (×3)
   - 2 cái cho đọc ADC
   - 1 cái cho điều chỉnh độ tương phản LCD
4. LED và điện trở 220Ω
5. Dây nối

### 3.2 Sơ đồ kết nối chi tiết
```
LCD -> Arduino
VSS -> GND
VDD -> 5V
VEE -> Biến trở 10kΩ
RS  -> D2
RW  -> GND
E   -> D3
D4  -> D4
D5  -> D5
D6  -> D6
D7  -> D7

Biến trở 1 -> A0
Biến trở 2 -> A5
LED (+ qua 220Ω) -> D8
```

## 4. Phân tích code

### 4.1 Khai báo và cấu hình
```cpp
#include <LiquidCrystal.h>
#define ADC_0   A0
#define ADC_5   A5

// Khai báo chân LCD
const int rs = 2, en = 3, d4 = 4, d5 = 5, d6 = 6, d7 = 7;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

// Biến toàn cục
int adc0, adc5;        // Lưu giá trị ADC
char Chuoi[17];        // Buffer hiển thị LCD
```

### 4.2 Hàm thiết lập
```cpp
void setup() {
  lcd.begin(16, 2);    // Khởi tạo LCD
  pinMode(8, OUTPUT);   // Cấu hình LED
  digitalWrite(8, LOW); // Tắt LED
}
```

### 4.3 Chương trình chính
```cpp
void loop() {
  // Đọc ADC
  adc0 = analogRead(ADC_0);
  adc5 = analogRead(ADC_5);

  // Hiển thị LCD
  lcd.setCursor(0,0);
  lcd.print("DOC ADC");
  lcd.setCursor(0,1);
  sprintf(Chuoi,"A0=%04d, A5=%04d",adc0,adc5);
  lcd.print(Chuoi);

  // Điều khiển LED
  digitalWrite(8, HIGH);
  delay(adc0);
  digitalWrite(8, LOW);
  delay(adc5);
}
```

## 5. Thực hành

### 5.1 Cải thiện đọc ADC
```cpp
// Hàm đọc ADC có lọc nhiễu
int readADC_filtered(uint8_t pin) {
    long sum = 0;
    for(int i = 0; i < 10; i++) {
        sum += analogRead(pin);
        delay(1);
    }
    return sum / 10;
}

// Chuyển đổi sang điện áp
float convertToVoltage(int adcValue) {
    return (adcValue * 5.0) / 1024.0;
}
```

### 5.2 Cải thiện hiển thị LCD
```cpp
void displayADCValues(int adc0, int adc5) {
    float v0 = convertToVoltage(adc0);
    float v5 = convertToVoltage(adc5);
    
    lcd.clear();
    lcd.setCursor(0,0);
    sprintf(Chuoi, "V0=%.2fV", v0);
    lcd.print(Chuoi);
    
    lcd.setCursor(0,1);
    sprintf(Chuoi, "V5=%.2fV", v5);
    lcd.print(Chuoi);
}
```

## 6. Bài tập và ứng dụng

### 6.1 Bài tập cơ bản
1. Hiển thị điện áp thay vì giá trị ADC
2. Thêm đơn vị đo
3. Tối ưu code hiển thị

### 6.2 Bài tập nâng cao
1. Tạo voltmeter đơn giản
2. Thêm chức năng cảnh báo điện áp
3. Lưu giá trị max/min

### 6.3 Ví dụ mở rộng
```cpp
// Cảnh báo điện áp cao
void voltageWarning() {
    float v0 = convertToVoltage(adc0);
    if(v0 > 4.0) {
        lcd.clear();
        lcd.print("CANH BAO!");
        lcd.setCursor(0,1);
        lcd.print("DIEN AP CAO");
        delay(1000);
    }
}
```

## 7. Troubleshooting

### 7.1 Vấn đề thường gặp
1. LCD không hiển thị
   - Kiểm tra kết nối
   - Điều chỉnh độ tương phản
   - Kiểm tra code khởi tạo

2. Giá trị ADC không ổn định
   - Thêm tụ lọc nhiễu
   - Sử dụng hàm lọc trong code
   - Kiểm tra nguồn điện

3. LED không nhấp nháy
   - Kiểm tra điện trở
   - Kiểm tra chân kết nối
   - Đảm bảo thời gian delay phù hợp

### 7.2 Tips and Tricks
1. Hardware
   - Dùng dây ngắn cho tín hiệu analog
   - Thêm tụ 100nF gần chân VCC
   - Tránh nhiễu từ nguồn điện

2. Software
   - Sử dụng hàm millis() thay delay()
   - Thêm bộ lọc số cho ADC
   - Tối ưu code hiển thị LCD

## Phụ lục: Mã nguồn hoàn chỉnh

```cpp
#include <LiquidCrystal.h>

// Định nghĩa chân
#define ADC_0   A0
#define ADC_5   A5
#define LED_PIN 8

// Khởi tạo LCD
const int rs = 2, en = 3, d4 = 4, d5 = 5, d6 = 6, d7 = 7;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

// Biến toàn cục
int adc0, adc5;
char Chuoi[17];
unsigned long lastUpdate = 0;

// Hàm đọc ADC có lọc nhiễu
int readADC_filtered(uint8_t pin) {
    long sum = 0;
    for(int i = 0; i < 10; i++) {
        sum += analogRead(pin);
        delay(1);
    }
    return sum / 10;
}

// Chuyển đổi sang điện áp
float convertToVoltage(int adcValue) {
    return (adcValue * 5.0) / 1024.0;
}

void setup() {
    lcd.begin(16, 2);
    pinMode(LED_PIN, OUTPUT);
    digitalWrite(LED_PIN, LOW);
}

void loop() {
    // Đọc và lọc ADC
    adc0 = readADC_filtered(ADC_0);
    adc5 = readADC_filtered(ADC_5);
    
    // Hiển thị LCD
    float v0 = convertToVoltage(adc0);
    float v5 = convertToVoltage(adc5);
    
    lcd.clear();
    lcd.setCursor(0,0);
    sprintf(Chuoi, "V0=%.2fV", v0);
    lcd.print(Chuoi);
    
    lcd.setCursor(0,1);
    sprintf(Chuoi, "V5=%.2fV", v5);
    lcd.print(Chuoi);
    
    // Điều khiển LED
    digitalWrite(LED_PIN, HIGH);
    delay(adc0);
    digitalWrite(LED_PIN, LOW);
    delay(adc5);
}
```

---
Tài liệu này được tạo cho mục đích giáo dục.