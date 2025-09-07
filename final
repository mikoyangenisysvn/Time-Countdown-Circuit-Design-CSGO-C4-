#include <ShiftRegister74HC595.h>
#include <LiquidCrystalSerial.h>

// Khởi tạo IC 74HC595 (DATA, CLOCK, LATCH)
const uint8_t dataPin = D3;
const uint8_t latchPin = D4;
const uint8_t clockPin = D2;

const uint8_t dataPin_LCD = D0;
const uint8_t latchPin_LCD = D1;

ShiftRegister74HC595<1> sr(dataPin, clockPin, latchPin);
LiquidCrystalSerial lcd(clockPin, dataPin_LCD, latchPin_LCD);

// Mảng mã phím bàn phím
char keypad[4][3] = {
    {'1', '2', '3'},
    {'4', '5', '6'},
    {'7', '8', '9'},
    {'*', '0', '#'},
};

// Cột của bàn phím
const int colPins[3] = {D7, D6, D5};
uint8_t rowState[] = {0b0111, 0b1011, 0b1101, 0b1110}; // Chỉ 3 cột

// Biến quản lý
char enteredCode[8] = "";          // Lưu mã đã nhập
int codeIndex = 0;                 // Vị trí nhập
const char correctCode[] = "7355608"; // Mã kích hoạt/tắt bom
bool isBombArmed = false;          // Trạng thái bom
unsigned long timerStart = 0;      // Thời gian bắt đầu đếm ngược
unsigned long countdownTime = 30000; // 30 giây đếm ngược mặc định
bool isEnteringTime = true;        // Trạng thái nhập thời gian
unsigned long enteredTime = 0;     // Lưu thời gian người dùng nhập

bool ledState = false;             // Trạng thái của LED
unsigned long lastBlinkTime = 0;   // Lưu thời gian lần nhấp nháy cuối cùng

char quetPhim();
void handleTimeInput(char key);
void handleKeyInput(char key) ;
void checkCode();
void displayCountdown(unsigned long timeLeft);
void bombExplode();
void controlLED(unsigned long timeLeft);
unsigned long calculateTotalSeconds();
char old_key,key;

void setup() {
    lcd.begin(16, 2);
    delay(100);

    // // Cấu hình cột là INPUT_PULLUP
    for (int i = 0; i < 3; i++) {
        pinMode(colPins[i], INPUT_PULLUP);
    }

    // // Bắt đầu ở trạng thái nhập thời gian
    lcd.clear();
    lcd.print("Set Time (s):");
    Serial.begin(115200);
}

void loop() {

    key = quetPhim();
    if(key != ' '){
    //   old_key = key;
      if (isEnteringTime) {
        handleTimeInput(key); // Xử lý nhập thời gian
      } else {
        handleKeyInput(key); // Xử lý nhập mã
      }
        Serial.println(key);
    }
    

    // Đếm ngược nếu bom đã kích hoạt
    if (isBombArmed) {
        unsigned long elapsedTime = millis() - timerStart;
        if (elapsedTime < countdownTime) {
            unsigned long timeLeft = countdownTime - elapsedTime;
            displayCountdown(timeLeft); // Hiển thị đếm ngược
            controlLED(timeLeft);       // Điều khiển LED nhấp nháy
        } else {
            bombExplode(); // Mô phỏng bom nổ
        }
    }
}

// Hàm quét phím
char quetPhim() {
    uint8_t row,col;
    for (row = 1; row < 5; row++) {
        sr.set(1, HIGH);
        sr.set(2, HIGH);
        sr.set(3, HIGH);
        sr.set(4, HIGH);
        sr.set(row, LOW);
        delay(5);
        // Đọc tín hiệu từ các cột
        if(digitalRead(colPins[0])== LOW || digitalRead(colPins[1]) == LOW || digitalRead(colPins[2]) == LOW){
            delay(50);
            for (col = 0; col < 3; col++) {
                if (digitalRead(colPins[col]) == LOW) {
                        while(digitalRead(colPins[col]) == LOW);
                        Serial.print("row: ");
                        Serial.println(row);
                        Serial.print("col: ");
                        Serial.println(col);
                        return keypad[row-1][col];
                }
            } 
        }

    }
    return ' ';
}

// Hàm xử lý nhập thời gian
void handleTimeInput(char key) {
    if (key == '#') {
        // Khi nhấn #, xác nhận thời gian đã nhập
        countdownTime = calculateTotalSeconds() * 1000;  // Chuyển sang mili giây
        lcd.clear();
        lcd.print("Enter Code:");  // Chuyển sang trạng thái nhập mã
        isEnteringTime = false;
    } else if (key == '*') {
        // Khi nhấn *, xóa toàn bộ thời gian đã nhập
        enteredTime = 0;
        lcd.clear();
        lcd.print("HH:MM:SS");
    } else if (key >= '0' && key <= '9') {
        // Chỉ xử lý khi nhập số (0-9)
        int digit = key - '0';  // Chuyển ký tự thành số
        if (enteredTime < 999999) {  // Giới hạn tối đa 6 chữ số
            enteredTime = enteredTime * 10 + digit;
            
            // Định dạng hiển thị HH:MM:SS
            unsigned long hours = enteredTime / 10000;
            unsigned long minutes = (enteredTime % 10000) / 100;
            unsigned long seconds = enteredTime % 100;
            
            lcd.clear();
            lcd.print("Time: ");
            
            // Hiển thị giờ với 2 chữ số
            if (hours < 10) lcd.print("0");
            lcd.print(hours);
            lcd.print(":");
            
            // Hiển thị phút với 2 chữ số
            if (minutes < 10) lcd.print("0");
            lcd.print(minutes);
            lcd.print(":");
            
            // Hiển thị giây với 2 chữ số
            if (seconds < 10) lcd.print("0");
            lcd.print(seconds);
        }
    }
}

// Hàm xử lý nhập mã
void handleKeyInput(char key) {
    if (key == '#') {
        // Nhấn # để xác nhận mã
        enteredCode[codeIndex] = '\0'; // Kết thúc chuỗi
        checkCode();
    } else if (key == '*') {
        // Nhấn * để xóa mã
        codeIndex = 0;
        memset(enteredCode, '\0', sizeof(enteredCode));
        lcd.clear();
        lcd.print("Enter Code:");
    } else {
        // Nhập số (tối đa 7 ký tự)
        if (codeIndex < 7) {
            enteredCode[codeIndex++] = key;
            lcd.setCursor(0, 1);
            lcd.print(enteredCode); // Hiển thị chuỗi mã đã nhập
        }
    }
}

// Hàm kiểm tra mã đã nhập
void checkCode() {
    lcd.clear();
    if (strcmp(enteredCode, correctCode) == 0) {
        if (!isBombArmed) {
            // Kích hoạt bom
            isBombArmed = true;
            timerStart = millis();
            lcd.print("Bomb Armed!");
        } else {
            // Tắt bom            
            isBombArmed = false;
            lcd.print("Bomb Defused!");
        }
    } else {
        lcd.print("Wrong Code!");
        memset(enteredCode, '\0', sizeof(enteredCode)); // Sets all elements to '\0'
    }
    delay(2000);
    lcd.clear();
    lcd.print("Enter Code:");
    codeIndex = 0; // Reset mã nhập
}

// Hàm hiển thị đếm ngược
void displayCountdown(unsigned long timeLeft) {
    lcd.clear();
    lcd.setCursor(0, 0);       // In trên dòng đầu tiên
    lcd.print("Time Left: ");  
    lcd.print(timeLeft / 1000);
    lcd.print("s");
}

// Hàm xử lý khi bom nổ
void bombExplode() {
    lcd.clear();
    lcd.print("Terrorist win!");
    sr.set(6,HIGH);
    for (int i = 0; i < 10; i++) { // Nháy LED để mô phỏng nổ
        sr.set(4, LOW);
        delay(200);
        sr.set(4, HIGH);
        delay(200);
    }
    while (1); // Vô hạn (reset để tiếp tục)
}

void controlLED(unsigned long timeLeft) {
    // Tính toán chu kỳ nhấp nháy (thời gian bật/tắt LED)
    unsigned long blinkInterval = map(timeLeft, 0, countdownTime, 50, 1000); 
    // Tốc độ nhấp nháy giảm dần từ 50ms đến 1000ms

    if (millis() - lastBlinkTime >= blinkInterval) {
        lastBlinkTime = millis();  // Cập nhật thời gian
        ledState = !ledState;      // Đảo trạng thái LED
        sr.set(7, ledState ? LOW : HIGH); // Điều khiển LED qua Q6
    }
}

unsigned long calculateTotalSeconds() {
    unsigned long hours = enteredTime / 10000;
    unsigned long minutes = (enteredTime % 10000) / 100;
    unsigned long seconds = enteredTime % 100;
    
    return hours * 3600 + minutes * 60 + seconds;
}
