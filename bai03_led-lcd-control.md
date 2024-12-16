# Code Điều Khiển LED và LCD - Phân Tích Chi Tiết

## 1. Khai Báo và Định Nghĩa

```cpp
// Include thư viện điều khiển LCD
#include <LiquidCrystal.h>

// Định nghĩa chân kết nối cho các LED
#define LED_1   8      // LED 1 được kết nối với chân digital 8
#define LED_2   9      // LED 2 được kết nối với chân digital 9

// Khai báo các biến toàn cục để lưu trữ và xử lý dữ liệu
int adc0, adc1;        // Biến lưu giá trị đọc từ ADC
char Chuoi[17];        // Buffer cho chuỗi hiển thị LCD (16 ký tự + null)
char TrangThai0[5];    // Lưu trạng thái LED 1 ("BAT"/"TAT")
char TrangThai1[5];    // Lưu trạng thái LED 2

// Mảng và biến cho phiên bản tối ưu
int adc[2];            // Mảng lưu giá trị của 2 kênh ADC
char i;                // Biến đếm cho vòng lặp
char TenKenh[2]={A0,A1}; // Mảng lưu tên các kênh analog cần đọc
char TrangThai[2][5];    // Mảng 2 chiều lưu trạng thái của 2 LED
int GiaTri[2]={500,800}; // Ngưỡng so sánh cho mỗi kênh

// Định nghĩa chân kết nối LCD
const int rs = 2, en = 3, d4 = 4, d5 = 5, d6 = 6, d7 = 7;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
```

## 2. Các Hàm Chính

### 2.1 Hàm Setup
```cpp
void setup() {
    // Khởi tạo LCD với kích thước 16x2
    lcd.begin(16, 2);
    
    // Cấu hình chân điều khiển LED là OUTPUT
    pinMode(LED_1, OUTPUT);
    pinMode(LED_2, OUTPUT);
    
    // Ban đầu tắt cả 2 LED
    digitalWrite(LED_1, LOW);
    digitalWrite(LED_2, LOW);
}
```

### 2.2 Hàm Loop
```cpp
void loop() {
    // Vòng lặp xử lý 2 kênh
    for(i=0; i<2; i++) {
        // Đọc giá trị ADC từ kênh tương ứng
        adc[i] = analogRead(TenKenh[i]);
        
        // So sánh với ngưỡng và điều khiển LED
        if(adc[i] > GiaTri[i]) {
            _DieuKhien(i, 0);          // Tắt LED
            sprintf(TrangThai[i], "TAT");
        }
        else {
            _DieuKhien(i, 1);          // Bật LED
            sprintf(TrangThai[i], "BAT");      
        }
    }
    
    // Hiển thị thông tin kênh 0 lên LCD dòng 1
    sprintf(Chuoi, "A0=%04d - %s", adc[0], TrangThai[0]);
    lcd.setCursor(0,0);
    lcd.print(Chuoi);
    
    // Hiển thị thông tin kênh 1 lên LCD dòng 2
    sprintf(Chuoi, "A1=%04d - %s", adc[1], TrangThai[1]);
    lcd.setCursor(0,1);
    lcd.print(Chuoi);    
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

## 3. Giải Thích Chi Tiết

### 3.1 Cấu Trúc Dữ Liệu
1. **Mảng TenKenh[]**
   - Lưu các chân analog cần đọc (A0, A1)
   - Giúp dễ dàng mở rộng số kênh

2. **Mảng GiaTri[]**
   - Lưu ngưỡng so sánh cho mỗi kênh
   - Kênh 0: 500
   - Kênh 1: 800

3. **Mảng TrangThai[][]**
   - Mảng 2 chiều lưu trạng thái LED
   - Mỗi trạng thái là chuỗi "BAT" hoặc "TAT"

### 3.2 Quy Trình Xử Lý
1. **Đọc ADC**
   ```cpp
   adc[i] = analogRead(TenKenh[i]);
   ```
   - Đọc giá trị analog (0-1023)
   - Lưu vào mảng adc[]

2. **So Sánh và Điều Khiển**
   ```cpp
   if(adc[i] > GiaTri[i]) {
       _DieuKhien(i, 0);  // Tắt
   }
   else {
       _DieuKhien(i, 1);  // Bật
   }
   ```

3. **Hiển Thị LCD**
   ```cpp
   sprintf(Chuoi, "A%d=%04d - %s", i, adc[i], TrangThai[i]);
   ```
   - Định dạng: "A0=xxxx - BAT/TAT"
   - %04d: Hiển thị số 4 chữ số, thêm 0 phía trước

## 4. Cải Tiến So Với Code Ban Đầu

### 4.1 Ưu Điểm
1. Sử dụng mảng thay vì biến riêng lẻ
2. Dùng vòng lặp xử lý đồng nhất
3. Tách riêng hàm điều khiển LED
4. Code ngắn gọn và dễ bảo trì

### 4.2 Khả Năng Mở Rộng
1. Thêm kênh mới: 
   - Thêm phần tử vào TenKenh[]
   - Thêm ngưỡng vào GiaTri[]

2. Thêm chức năng:
   - Cảnh báo khi vượt ngưỡng
   - Lưu trạng thái
   - Điều khiển từ xa

## 5. Tips và Lưu Ý

### 5.1 Tối Ưu Hiển Thị
1. Sử dụng LCD.clear() khi cần
2. Tránh cập nhật LCD quá nhanh
3. Chỉ cập nhật khi có thay đổi

### 5.2 Xử Lý ADC
1. Thêm bộ lọc nhiễu
2. Lấy giá trị trung bình
3. Thêm độ trễ (hysteresis)

### 5.3 Debug
1. Kiểm tra giá trị ADC
2. Theo dõi trạng thái LED
3. Kiểm tra định dạng hiển thị

---
Tài liệu này được tạo cho mục đích học tập và tham khảo.