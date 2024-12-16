# Phân Tích Code Điều Khiển LED 7 Đoạn

## 1. Khai Báo và Định Nghĩa

### 1.1 Thư Viện và Định Nghĩa Chân
```cpp
#include "PORT.h"
//khai bao ket noi cac cot led 7 doan
#define PIN_COT_1   10
#define PIN_COT_2   11
#define PIN_COT_3   12
#define PIN_COT_4   13
```
**Giải thích**:
- Sử dụng thư viện PORT.h để quản lý các chân I/O
- Định nghĩa 4 chân điều khiển các cột (digit) từ 10-13
- Mỗi cột điều khiển một chữ số của LED 7 đoạn

### 1.2 Biến Toàn Cục
```cpp
PORT _PORT;
int adc0=1289, adc5;
byte H1,H2,H3,H4;
```
**Chi tiết**:
- Tạo đối tượng _PORT để điều khiển các chân
- adc0, adc5: Lưu giá trị đọc từ ADC (0-1023)
- H1-H4: Các chữ số được tách ra để hiển thị

## 2. Các Hàm Chính

### 2.1 Hàm Khởi Tạo
```cpp
void setup() {
    // Khởi động các chân điều khiển đoạn
    _PORT._KhoiDongHang(2,3,4,5,6,7,8,9);
    
    // Cấu hình các chân cột
    pinMode(PIN_COT_1,OUTPUT);
    pinMode(PIN_COT_2,OUTPUT);
    pinMode(PIN_COT_3,OUTPUT);
    pinMode(PIN_COT_4,OUTPUT);
    
    // Tắt tất cả các cột
    digitalWrite(PIN_COT_1,HIGH);
    digitalWrite(PIN_COT_2,HIGH);
    digitalWrite(PIN_COT_3,HIGH);
    digitalWrite(PIN_COT_4,HIGH);
}
```
**Chức năng**:
- Khởi tạo các chân điều khiển đoạn (a-g)
- Cấu hình chân cột là OUTPUT
- Đặt các cột ở mức HIGH (tắt)

### 2.2 Hàm Tách Số
```cpp
void _TachSo(int adc, byte *H1, byte *H2, byte *H3, byte *H4) {
    *H4 = adc%10;
    adc = adc/10;
    *H3 = adc%10;
    adc = adc/10;
    *H2 = adc%10;
    adc = adc/10;
    *H1 = adc%10;
    adc = adc/10;      
}
```
**Nguyên lý**:
1. Nhận giá trị ADC và 4 con trỏ
2. Tách từng chữ số bằng phép chia và lấy dư
3. Lưu vào các biến H1-H4 thông qua con trỏ

**Ví dụ**:
```
Với adc = 1289:
H4 = 1289 % 10 = 9
adc = 1289 / 10 = 128
H3 = 128 % 10 = 8
adc = 128 / 10 = 12
H2 = 12 % 10 = 2
adc = 12 / 10 = 1
H1 = 1 % 10 = 1
```

### 2.3 Hàm Hiển Thị LED
```cpp
void _HienThiLed(byte H1, byte H2, byte H3, byte H4) {
    unsigned char MaLed[] = {0xC0,0xF9,0xA4,0xB0,0x99,0x92,
                            0x82,0xF8,0x80,0x98,0xFF,0xBF};
    
    // Hiển thị H1
    _PORT._XuatHang(MaLed[H1]);
    digitalWrite(PIN_COT_1,LOW);
    delay(1);
    digitalWrite(PIN_COT_1,HIGH);
    
    // Tương tự cho H2, H3, H4
    // ...
}
```
**Quy trình hiển thị**:
1. Xuất mã LED cho số cần hiển thị
2. Bật cột tương ứng (LOW)
3. Duy trì trong 1ms
4. Tắt cột (HIGH)
5. Lặp lại cho số tiếp theo

### 2.4 Hàm Delay Quét
```cpp
void _DelayScan(byte H1, byte H2, byte H3, byte H4, int t) {
    t = t/4;
    while(t>0) {
        t = t-1;
        _HienThiLed(H1,H2,H3,H4); 
    }
}
```
**Chức năng**:
- Tạo delay với duy trì hiển thị
- Chia thời gian cho 4 (4 chữ số)
- Quét liên tục trong thời gian delay

## 3. Mã LED 7 Đoạn

### 3.1 Bảng Mã
```cpp
unsigned char MaLed[] = {
    0xC0,  // Số 0: 1100 0000
    0xF9,  // Số 1: 1111 1001
    0xA4,  // Số 2: 1010 0100
    0xB0,  // Số 3: 1011 0000
    0x99,  // Số 4: 1001 1001
    0x92,  // Số 5: 1001 0010
    0x82,  // Số 6: 1000 0010
    0xF8,  // Số 7: 1111 1000
    0x80,  // Số 8: 1000 0000
    0x98   // Số 9: 1001 1000
};
```
**Quy ước**:
- Bit 0: LED sáng
- Bit 1: LED tắt
- 7 bit thấp điều khiển các đoạn a-g

## 4. Vòng Lặp Chính

```cpp
void loop() {
    // Đọc ADC
    adc0 = analogRead(A0);
    adc5 = analogRead(A5);
    
    // Hiển thị giá trị ADC0
    _TachSo(adc0, &H1,&H2,&H3,&H4);
    _DelayScan(H1,H2,H3,H4,500);
    
    // Hiển thị giá trị ADC5
    _TachSo(adc5, &H1,&H2,&H3,&H4);
    _DelayScan(H1,H2,H3,H4,500);
}
```
**Quá trình**:
1. Đọc giá trị từ 2 kênh ADC
2. Tách số thành các chữ số
3. Hiển thị lần lượt 2 giá trị
4. Mỗi giá trị hiển thị trong 500ms

## 5. Tính Toán Timing

### 5.1 Thời gian quét
```
Thời gian cho một số: 1ms
Tổng thời gian một chu kỳ: 4ms
Tần số quét = 1000/4 = 250Hz
```

### 5.2 Tính LED
```
Điện áp LED: 2V
Dòng LED: 20mA
Điện trở hạn dòng = (5V - 2V) / 20mA = 150Ω
```

## 6. Tips và Lưu Ý

1. **Tối ưu hiển thị**:
   - Tăng/giảm thời gian quét tùy độ sáng
   - Điều chỉnh tần số quét > 60Hz
   - Cân bằng độ sáng giữa các số

2. **Xử lý số 0 đầu**:
   ```cpp
   if(H1 == 0) MaLed[11];  // Tắt số 0 đầu
   ```

3. **Debug**:
   - Kiểm tra từng số riêng biệt
   - Đo tần số quét thực tế
   - Kiểm tra dòng điện qua LED

4. **Mở rộng**:
   - Thêm ký tự đặc biệt
   - Điều chỉnh độ sáng
   - Tạo hiệu ứng hiển thị

---
Tài liệu này được tạo cho mục đích học tập và tham khảo.