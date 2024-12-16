# Giải Thích Chi Tiết Code LED 7 Đoạn

## 1. Khai Báo Ban Đầu

```cpp
// Include thư viện PORT.h để điều khiển các chân I/O một cách hiệu quả
// Thư viện này giúp quản lý và điều khiển các chân digital dễ dàng hơn
#include "PORT.h"

// Định nghĩa các chân điều khiển cột (digit select) cho LED 7 đoạn
// Mỗi cột điều khiển một chữ số của LED
#define PIN_COT_1   10    // Điều khiển chữ số hàng nghìn
#define PIN_COT_2   11    // Điều khiển chữ số hàng trăm
#define PIN_COT_3   12    // Điều khiển chữ số hàng chục
#define PIN_COT_4   13    // Điều khiển chữ số hàng đơn vị

// Tạo đối tượng _PORT để quản lý các chân
PORT _PORT;

// Khai báo các biến toàn cục
int adc0=1289, adc5;      // Biến lưu giá trị đọc từ ADC (0-1023)
byte H1,H2,H3,H4;         // Biến lưu từng chữ số sau khi tách
```

## 2. Khai Báo Prototype

```cpp
// Khai báo prototype các hàm sẽ sử dụng
void _TachSo(int adc, byte *H1, byte *H2, byte *H3, byte *H4);    
// Hàm tách số thành các chữ số riêng lẻ

void _HienThiLed(byte H1, byte H2, byte H3, byte H4);             
// Hàm điều khiển hiển thị LED

void _DelayScan(byte H1, byte H2, byte H3, byte H4, int t);       
// Hàm tạo delay có quét LED
```

## 3. Hàm Setup

```cpp
void setup() {
    // Khởi tạo các chân điều khiển đoạn LED (a-g)
    // Các chân này điều khiển 7 đoạn của LED, từ chân 2-9
    _PORT._KhoiDongHang(2,3,4,5,6,7,8,9);
    
    // Cấu hình các chân điều khiển cột là OUTPUT
    // Các chân này điều khiển việc chọn chữ số nào sẽ hiển thị
    pinMode(PIN_COT_1,OUTPUT);
    pinMode(PIN_COT_2,OUTPUT);
    pinMode(PIN_COT_3,OUTPUT);
    pinMode(PIN_COT_4,OUTPUT);
    
    // Ban đầu tắt tất cả các cột (mức HIGH)
    // LED 7 đoạn dùng là loại Common Cathode nên HIGH = tắt
    digitalWrite(PIN_COT_1,HIGH);
    digitalWrite(PIN_COT_2,HIGH);
    digitalWrite(PIN_COT_3,HIGH);
    digitalWrite(PIN_COT_4,HIGH);
}
```

## 4. Vòng Lặp Chính

```cpp
void loop() {
    // Đọc giá trị analog từ 2 chân A0 và A5
    // analogRead trả về giá trị từ 0-1023 (10-bit ADC)
    adc0 = analogRead(A0);
    adc5 = analogRead(A5);
    
    // Xử lý và hiển thị giá trị ADC0
    _TachSo(adc0,&H1,&H2,&H3,&H4);     // Tách số thành các chữ số
    _DelayScan(H1,H2,H3,H4,500);        // Hiển thị trong 500ms
    
    // Xử lý và hiển thị giá trị ADC5
    _TachSo(adc5,&H1,&H2,&H3,&H4);     // Tách số thành các chữ số
    _DelayScan(H1,H2,H3,H4,500);        // Hiển thị trong 500ms
}
```

## 5. Hàm Tách Số

```cpp
void _TachSo(int adc, byte *H1, byte *H2, byte *H3, byte *H4) {
    // Tách số thành các chữ số riêng biệt bằng phép chia và lấy dư
    *H4 = adc%10;         // Lấy chữ số hàng đơn vị
    adc = adc/10;         // Loại bỏ chữ số vừa lấy
    
    *H3 = adc%10;         // Lấy chữ số hàng chục
    adc = adc/10;         // Loại bỏ chữ số vừa lấy
    
    *H2 = adc%10;         // Lấy chữ số hàng trăm
    adc = adc/10;         // Loại bỏ chữ số vừa lấy
    
    *H1 = adc%10;         // Lấy chữ số hàng nghìn
    adc = adc/10;         // Loại bỏ chữ số vừa lấy
}
```

## 6. Hàm Hiển Thị LED

```cpp
void _HienThiLed(byte H1, byte H2, byte H3, byte H4) {
    // Mảng mã LED 7 đoạn cho các số từ 0-9 và ký tự đặc biệt
    // Mỗi byte điều khiển 7 đoạn LED (a-g)
    // Bit 0 = LED sáng, Bit 1 = LED tắt
    unsigned char MaLed[]={
        0xC0,  // Số 0: 1100 0000
        0xF9,  // Số 1: 1111 1001
        0xA4,  // Số 2: 1010 0100
        0xB0,  // Số 3: 1011 0000
        0x99,  // Số 4: 1001 1001
        0x92,  // Số 5: 1001 0010
        0x82,  // Số 6: 1000 0010
        0xF8,  // Số 7: 1111 1000
        0x80,  // Số 8: 1000 0000
        0x98,  // Số 9: 1001 1000
        0xFF,  // Tắt hết: 1111 1111
        0xBF   // Dấu trừ: 1011 1111
    };
    
    // Hiển thị chữ số hàng nghìn (H1)
    _PORT._XuatHang(MaLed[H1]);         // Xuất mã điều khiển đoạn
    digitalWrite(PIN_COT_1,LOW);         // Bật cột 1 (cho phép hiển thị)
    delay(1);                           // Duy trì 1ms
    digitalWrite(PIN_COT_1,HIGH);        // Tắt cột 1
    
    // Hiển thị chữ số hàng trăm (H2)
    _PORT._XuatHang(MaLed[H2]);
    digitalWrite(PIN_COT_2,LOW);
    delay(1);
    digitalWrite(PIN_COT_2,HIGH);
    
    // Hiển thị chữ số hàng chục (H3)
    _PORT._XuatHang(MaLed[H3]);
    digitalWrite(PIN_COT_3,LOW);
    delay(1);
    digitalWrite(PIN_COT_3,HIGH);
    
    // Hiển thị chữ số hàng đơn vị (H4)
    _PORT._XuatHang(MaLed[H4]);
    digitalWrite(PIN_COT_4,LOW);
    delay(1);
    digitalWrite(PIN_COT_4,HIGH);
}
```

## 7. Hàm Delay Có Quét

```cpp
void _DelayScan(byte H1, byte H2, byte H3, byte H4, int t) {
    // Chia thời gian delay cho 4 vì có 4 chữ số cần quét
    t = t/4;
    
    // Lặp quét LED trong suốt thời gian delay
    while(t>0) {
        t = t-1;
        _HienThiLed(H1,H2,H3,H4);       // Quét LED liên tục
    }
}
```

## 8. Giải Thích Thêm

### 8.1. Nguyên lý quét LED
- Chỉ một chữ số được hiển thị tại một thời điểm
- Quét nhanh qua 4 chữ số (mỗi số 1ms)
- Tạo ảo giác hiển thị đồng thời nhờ tồn tại hình ảnh
- Tần số quét: 1000Hz/(4 số) = 250Hz > 60Hz (ngưỡng nhấp nháy)

### 8.2. Tối ưu code
1. Sử dụng port manipulation thay vì digitalWrite
2. Giảm thời gian delay nếu cần tăng độ sáng
3. Tăng tần số quét để giảm nhấp nháy

### 8.3. Debug và xử lý lỗi
1. Kiểm tra từng chữ số riêng biệt
2. Đo thời gian quét thực tế
3. Kiểm tra điện áp và dòng điện qua LED

---
Tài liệu này được tạo cho mục đích học tập và tham khảo.
