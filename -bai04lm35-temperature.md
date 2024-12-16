# Giải Thích Chi Tiết Code Đọc Nhiệt Độ LM35

## 1. Khai Báo và Định Nghĩa

```cpp
// Include thư viện điều khiển LCD
#include <LiquidCrystal.h>

// Định nghĩa chân kết nối cảm biến LM35
#define LM35  A1      // Cảm biến kết nối với chân analog A1

// Khai báo biến toàn cục
int adc[2];           // Mảng lưu giá trị ADC từ 2 cảm biến
float Tam;            // Biến tạm để tính toán nhiệt độ
byte NhietDo[2];      // Mảng lưu nhiệt độ sau chuyển đổi
char Chuoi[17];       // Buffer hiển thị LCD (16 ký tự + null)

// Cấu hình chân LCD
const int rs = 2, en = 3, d4 = 4, d5 = 5, d6 = 6, d7 = 7;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
```

## 2. Các Hàm Chính

### 2.1 Hàm Setup
```cpp
void setup() {
    // Khởi tạo LCD kích thước 16x2
    lcd.begin(16, 2);
}
```

### 2.2 Hàm Loop
```cpp
void loop() {
    // Đọc và xử lý cảm biến 1
    adc[0] = analogRead(LM35);             // Đọc ADC
    Tam = (adc[0]*500.0)/1023.0;          // Chuyển sang độ C
    NhietDo[0] = Tam;                      // Lưu giá trị

    // Đọc và xử lý cảm biến 2
    adc[1] = analogRead(A2);               // Đọc ADC
    Tam = (adc[1]*500.0)/1023.0;          // Chuyển sang độ C
    NhietDo[1] = Tam;                      // Lưu giá trị

    // Hiển thị tiêu đề
    lcd.setCursor(0,0);
    lcd.print("DOC C.BIEN LM35");

    // Hiển thị nhiệt độ
    lcd.setCursor(0,1);
    sprintf(Chuoi,"%02d%cC, %02d%cC",
            NhietDo[0],223,NhietDo[1],223);
    lcd.print(Chuoi);
}
```

## 3. Công Thức Chuyển Đổi

### 3.1 Chuyển Đổi ADC sang Điện Áp
```
Công thức:
1023 ADC = 5000mV
x ADC    = (x * 5000)/1023 mV
```

### 3.2 Chuyển Đổi Điện Áp sang Nhiệt Độ
```
Theo datasheet LM35:
10mV = 1°C
y mV  = y/10 °C
```

### 3.3 Công Thức Tổng Hợp
```
Nhiệt độ = (ADC * 5000)/1023/10
         = (ADC * 500)/1023 °C
```

## 4. Format Hiển Thị LCD

### 4.1 Dòng 1 - Tiêu Đề
```cpp
lcd.setCursor(0,0);
lcd.print("DOC C.BIEN LM35");
```
- Vị trí: (0,0) - đầu dòng 1
- Nội dung: Tiêu đề cố định

### 4.2 Dòng 2 - Nhiệt Độ
```cpp
sprintf(Chuoi,"%02d%cC, %02d%cC",
        NhietDo[0],223,NhietDo[1],223);
```
- `%02d`: Số nguyên 2 chữ số, thêm 0 phía trước
- `223`: Mã ASCII của ký tự độ (°)
- Format: xx°C, yy°C

## 5. Tối Ưu và Cải Tiến

### 5.1 Cải Thiện Độ Chính Xác
```cpp
// Đọc nhiều lần và lấy trung bình
float readTemp(byte pin) {
    long sum = 0;
    for(byte i=0; i<8; i++) {
        sum += analogRead(pin);
        delay(1);
    }
    return (sum * 500.0)/(1023.0 * 8);
}
```

### 5.2 Xử Lý Nhiễu
```cpp
// Thêm bộ lọc
float filterTemp(float newTemp) {
    static float oldTemp = 25.0;
    float filteredTemp = 0.9*oldTemp + 0.1*newTemp;
    oldTemp = filteredTemp;
    return filteredTemp;
}
```

## 6. Tips và Lưu Ý

### 6.1 Độ Chính Xác
1. LM35 có độ chính xác ±0.5°C
2. ADC 10-bit cho độ phân giải ~0.5°C
3. Nên hiệu chuẩn cảm biến

### 6.2 Hiển Thị
1. Chỉ cập nhật LCD khi có thay đổi
2. Tránh nhấp nháy màn hình
3. Xem xét thêm đơn vị đo khác (°F)

### 6.3 Cải Tiến
1. Thêm cảnh báo nhiệt độ cao/thấp
2. Lưu lịch sử nhiệt độ
3. Giao tiếp với máy tính/smartphone

---
Tài liệu này được tạo cho mục đích học tập và tham khảo.