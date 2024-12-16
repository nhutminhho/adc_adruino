# Quá Trình Chuyển Đổi ADC Trong Arduino

## 1. Cấu Trúc ADC Trong Arduino

### 1.1 Đặc điểm kỹ thuật
- Kiểu: SAR (Successive Approximation Register)
- Độ phân giải: 10-bit
- Thời gian chuyển đổi: 13 chu kỳ ADC clock
- Tần số lấy mẫu tối đa: 15kSPS (15,000 samples per second)
- Điện áp tham chiếu mặc định: 5V (có thể thay đổi)

### 1.2 Các thanh ghi chính
```
ADMUX: ADC Multiplexer Selection Register
ADCSRA: ADC Control and Status Register A
ADCH/ADCL: ADC Data Registers
```

## 2. Quá Trình Chuyển Đổi Chi Tiết

### 2.1 Bước 1: Khởi tạo ADC
```cpp
// Các bit cấu hình trong ADCSRA
ADCSRA = (1 << ADEN)    // Enable ADC
       | (1 << ADPS2)   // Prescaler = 128
       | (1 << ADPS1)   // for 16MHz Arduino
       | (1 << ADPS0);  // ADC clock = 125kHz
```

### 2.2 Bước 2: Chọn kênh và điện áp tham chiếu
```cpp
// ADMUX configuration
ADMUX = (1 << REFS0);    // AVCC with external capacitor
ADMUX |= channel & 0x07; // Select ADC channel
```

### 2.3 Quá trình chuyển đổi SAR
```
Ví dụ cho điện áp 3.3V:
3.3V/5V × 1023 = 675 (0b1010100011)

Bước    Bit Test    Giá trị thử    So sánh
1       Bit 9       512 (2.5V)     3.3V > 2.5V → 1
2       Bit 8       768 (3.75V)    3.3V < 3.75V → 0
3       Bit 7       640 (3.125V)   3.3V > 3.125V → 1
...và tiếp tục cho đến bit 0
```

### 2.4 Timing Diagram
```
           ┌─────┐     ┌─────┐     ┌─────┐
ADSC       │     └─────┘     └─────┘     └─────
           │     │     │     │     │     │
           │     │     │     │     │     │
Sample     ┌─┐   │     ┌─┐   │     ┌─┐   │
& Hold     │ └───┘     │ └───┘     │ └───┘
           │           │           │
ADC Clock  ┊◀─13 cycles─▶┊◀─13 cycles─▶┊
```

## 3. Công Thức Và Tính Toán

### 3.1 Công thức cơ bản
```
ADC = (Vin × 1024) / Vref

Trong đó:
- Vin: Điện áp đầu vào
- Vref: Điện áp tham chiếu (5V mặc định)
- 1024: 2^10 (10-bit resolution)
```

### 3.2 Ví dụ tính toán
```
Với Vin = 3.3V, Vref = 5V:
ADC = (3.3 × 1024) / 5 = 675.84 ≈ 676

Ngược lại:
Voltage = (ADC × Vref) / 1024
Voltage = (676 × 5) / 1024 = 3.30078125V
```

### 3.3 Độ phân giải
```
Resolution = Vref / 2^10
          = 5V / 1024
          = 4.88mV per step
```

## 4. Code Thực Hành

### 4.1 Đọc ADC cơ bản
```cpp
int readADC(uint8_t channel) {
    // Chọn kênh ADC
    ADMUX = (ADMUX & 0xF0) | (channel & 0x0F);
    
    // Bắt đầu chuyển đổi
    ADCSRA |= (1 << ADSC);
    
    // Đợi chuyển đổi hoàn thành
    while(ADCSRA & (1 << ADSC));
    
    // Đọc kết quả
    return ADC;
}
```

### 4.2 Đọc ADC với trung bình hóa
```cpp
int readADC_Averaged(uint8_t channel, uint8_t samples = 8) {
    long sum = 0;
    
    for(uint8_t i = 0; i < samples; i++) {
        // Bắt đầu chuyển đổi
        ADMUX = (ADMUX & 0xF0) | (channel & 0x0F);
        ADCSRA |= (1 << ADSC);
        
        // Đợi hoàn thành
        while(ADCSRA & (1 << ADSC));
        
        // Cộng dồn kết quả
        sum += ADC;
        
        // Đợi ADC ổn định
        delayMicroseconds(100);
    }
    
    // Trả về giá trị trung bình
    return sum / samples;
}
```

## 5. Tối Ưu Và Cải Thiện

### 5.1 Lọc nhiễu phần cứng
```
1. Thêm tụ bypass:
   - 100nF gần chân AVCC
   - 10µF electrolytic cho nguồn

2. Layout PCB:
   - Đường analog ngắn
   - Ground plane riêng
   - Tách biệt phần analog/digital
```

### 5.2 Lọc nhiễu phần mềm
```cpp
class ADCFilter {
private:
    static const int WINDOW_SIZE = 8;
    int values[WINDOW_SIZE];
    int index = 0;
    
public:
    int update(int newValue) {
        // Thêm giá trị mới
        values[index] = newValue;
        index = (index + 1) % WINDOW_SIZE;
        
        // Sắp xếp và lấy trung vị
        int temp[WINDOW_SIZE];
        memcpy(temp, values, sizeof(values));
        sort(temp, temp + WINDOW_SIZE);
        
        return temp[WINDOW_SIZE/2];
    }
};
```

## 6. Vấn Đề Thường Gặp

### 6.1 Nhiễu ADC
1. **Nguyên nhân**:
   - EMI từ mạch số
   - Ripple nguồn
   - Ground bounce

2. **Giải pháp**:
   ```cpp
   // Đọc và bỏ qua lần đầu
   analogRead(ADC_0);
   delayMicroseconds(100);
   
   // Đọc giá trị thật
   int value = analogRead(ADC_0);
   ```

### 6.2 Sai số đo
1. **Nguyên nhân**:
   - Điện áp tham chiếu không chính xác
   - Nhiệt độ thay đổi
   - Input impedance

2. **Giải pháp**:
   ```cpp
   // Calibration với điện áp chuẩn
   float calibration_factor = 1.0;
   int raw = analogRead(ADC_0);
   float voltage = (raw * 5.0 * calibration_factor) / 1024.0;
   ```

## 7. Ứng Dụng Thực Tế

### 7.1 Đo điện áp chính xác
```cpp
float readVoltage(uint8_t pin) {
    const float cal_factor = 1.0234; // Từ calibration
    const int samples = 16;
    long sum = 0;
    
    for(int i = 0; i < samples; i++) {
        sum += analogRead(pin);
        delayMicroseconds(100);
    }
    
    int avg = sum / samples;
    return (avg * 5.0 * cal_factor) / 1024.0;
}
```

### 7.2 Auto-ranging voltmeter
```cpp
float autoRangeRead(uint8_t pin) {
    int raw = analogRead(pin);
    float voltage = (raw * 5.0) / 1024.0;
    
    // Chọn thang đo phù hợp
    if(voltage < 0.1) {
        // Chuyển sang thang mV
        return voltage * 1000;
    }
    return voltage;
}
```

Bạn cần thêm thông tin về phần nào của quá trình chuyển đổi ADC?