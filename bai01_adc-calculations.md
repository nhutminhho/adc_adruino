# Cách Tính Toán ADC Chi Tiết

## 1. Công Thức Cơ Bản ADC

### 1.1 Chuyển đổi từ Điện áp sang ADC
```
ADC = (Vin × 1024) / Vref

Trong đó:
- ADC: Giá trị số sau chuyển đổi (0-1023)
- Vin: Điện áp đầu vào (0-5V)
- Vref: Điện áp tham chiếu (thường là 5V)
- 1024: 2^10 (độ phân giải 10-bit)
```

### 1.2 Chuyển đổi từ ADC sang Điện áp
```
Vin = (ADC × Vref) / 1024

Ví dụ:
ADC = 512
Vin = (512 × 5V) / 1024 = 2.5V
```

## 2. Độ Phân Giải ADC

### 2.1 Độ phân giải điện áp
```
Resolution = Vref / 2^10
          = 5V / 1024
          = 4.88mV/step

Nghĩa là:
- Mỗi bước ADC = 4.88mV
- Độ chính xác tối đa = ±2.44mV
```

### 2.2 Bảng giá trị điển hình
```
ADC    Voltage
0      0.000V
1      0.004883V
2      0.009766V
...
511    2.495117V
512    2.500000V
...
1022   4.990234V
1023   4.995117V
```

## 3. Ví Dụ Tính Toán Chi Tiết

### 3.1 Ví dụ 1: Đo điện áp 3.3V
```
Bước 1: Chuyển đổi 3.3V sang ADC
ADC = (3.3V × 1024) / 5V
    = 3379.2 / 5
    = 675.84
    ≈ 676 (làm tròn)

Bước 2: Kiểm tra lại điện áp thực
Vin = (676 × 5V) / 1024
    = 3380 / 1024
    = 3.301V
```

### 3.2 Ví dụ 2: Đo điện trở với điện áp phân áp
```
Cấu hình: R1 = 10kΩ (cố định), Rx (cần đo)
Công thức: Rx = R1 × (1024/ADC - 1)

ADC = 512
Rx = 10kΩ × (1024/512 - 1)
   = 10kΩ × (2 - 1)
   = 10kΩ
```

## 4. Code Tính Toán Thực Tế

### 4.1 Tính điện áp đơn giản
```cpp
float calculateVoltage(int adcValue) {
    return (adcValue * 5.0) / 1024.0;
}

void loop() {
    int adc0 = analogRead(ADC_0);
    float voltage = calculateVoltage(adc0);
    
    // Hiển thị trên LCD
    lcd.setCursor(0,0);
    sprintf(Chuoi, "V=%.3fV", voltage);
    lcd.print(Chuoi);
}
```

### 4.2 Tính toán với calibration
```cpp
float calibrateADC(int adcValue) {
    const float CAL_FACTOR = 1.0234;  // Từ đo đạc thực tế
    float voltage = (adcValue * 5.0 * CAL_FACTOR) / 1024.0;
    return voltage;
}
```

## 5. Các Trường Hợp Đặc Biệt

### 5.1 Tính toán với offset
```cpp
float calculateWithOffset(int adcValue) {
    const float OFFSET = 0.05;  // 50mV offset
    float voltage = (adcValue * 5.0) / 1024.0;
    return voltage - OFFSET;
}
```

### 5.2 Tính toán phạm vi đo
```cpp
struct Range {
    float min_voltage;
    float max_voltage;
    int min_adc;
    int max_adc;
};

Range calculateRange(float target_voltage, float tolerance) {
    Range r;
    r.min_voltage = target_voltage * (1 - tolerance);
    r.max_voltage = target_voltage * (1 + tolerance);
    r.min_adc = (r.min_voltage * 1024) / 5.0;
    r.max_adc = (r.max_voltage * 1024) / 5.0;
    return r;
}
```

## 6. Cải Thiện Độ Chính Xác

### 6.1 Oversampling
```cpp
// Tăng độ phân giải lên 12-bit bằng oversampling
int oversample12bit() {
    long sum = 0;
    // Cần 16 mẫu cho mỗi bit thêm
    for(int i = 0; i < 16; i++) {
        sum += analogRead(ADC_0);
    }
    // Chia cho 4 (2^2) để có kết quả 12-bit
    return sum >> 2;
}
```

### 6.2 Moving Average
```cpp
class VoltageFilter {
    static const int WINDOW_SIZE = 16;
    float values[WINDOW_SIZE];
    int index = 0;
    
public:
    float update(float newVoltage) {
        values[index] = newVoltage;
        index = (index + 1) % WINDOW_SIZE;
        
        float sum = 0;
        for(int i = 0; i < WINDOW_SIZE; i++) {
            sum += values[i];
        }
        return sum / WINDOW_SIZE;
    }
};
```

## 7. Ứng Dụng Thực Tế

### 7.1 Đo pin lithium-ion
```cpp
float measureBatteryVoltage() {
    // Pin lithium 3.7V qua điện trở phân áp 2:1
    float voltage = calculateVoltage(analogRead(ADC_0));
    return voltage * 2;  // Nhân 2 vì phân áp
}

String getBatteryStatus(float voltage) {
    if(voltage >= 4.0) return "FULL";
    if(voltage >= 3.7) return "GOOD";
    if(voltage >= 3.5) return "LOW";
    return "CRITICAL";
}
```

### 7.2 Cảm biến ánh sáng (LDR)
```cpp
int calculateLux(int adcValue) {
    // Công thức chuyển đổi cho LDR cụ thể
    float voltage = (adcValue * 5.0) / 1024.0;
    float resistance = 10000 * (5 - voltage) / voltage;
    return 12518931 * pow(resistance, -1.405);
}
```

Bạn cần thêm thông tin về phần nào của các phép tính ADC?