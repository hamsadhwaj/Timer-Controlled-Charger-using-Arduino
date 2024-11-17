#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <EEPROM.h>

// Initialize the I2C LCD object with address 0x27 (16x2 display)
LiquidCrystal_I2C lcd(0x27, 16, 2);

#define outputA 10   // clk pin of rotary encoder
#define outputB 11   // dt pin of rotary encoder
#define button 2     // sw pin of rotary encoder
#define led 13       // status LED
#define relay 12     // relay control pin

int count = 1;          // Encoder count state
int current_state;      // Current state of encoder A pin
int previous_state;     // Previous state of encoder A pin
int hh = 0, mm = 0, ss = 0; // Hours, minutes, and seconds for timer
int h = 0, m = 0, s = 0;     // Control flags for HH, MM, SS
int ledToggle = 0;      // LED toggle state
int previousState = HIGH;
unsigned int previousPress;  // For debouncing
volatile int buttonFlag;     // Button interrupt flag
int buttonDebounce = 20;     // Debounce delay time

void setup() {
  lcd.init();            // Initialize the LCD
  lcd.backlight();       // Turn on the LCD backlight

  pinMode(outputA, INPUT);
  pinMode(outputB, INPUT);
  pinMode(button, INPUT_PULLUP);
  pinMode(led, OUTPUT);
  pinMode(relay, OUTPUT);
  digitalWrite(relay, HIGH); // Set relay pin high initially
  
  attachInterrupt(digitalPinToInterrupt(button), button_ISR, CHANGE);
  Serial.begin(9600);
  previous_state = digitalRead(outputA);

  // Check EEPROM for stored values
  if (EEPROM.read(0) != 0 || EEPROM.read(1) != 0 || EEPROM.read(2) != 0) {
    lcd.setCursor(0, 0);
    lcd.print("Load Preset?");
    lcd.setCursor(0, 1);
    lcd.print("      <NO>      ");
    int temp = 5;
    while (temp > 0 && ledToggle == 0) {
      lcd.setCursor(14, 0);
      lcd.print(temp);
      temp--;
      delay(1000);
    }
    if (temp == 0 && ledToggle == 0) {
      hh = EEPROM.read(0);
      mm = EEPROM.read(1);
      ss = EEPROM.read(2);
      timer();
    } else {
      EEPROM.write(0, 0);
      EEPROM.write(1, 0);
      EEPROM.write(2, 0);
      ledToggle = 0;
    }
  }
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(" HH  MM  SS  OK ");
  lcd.setCursor(0, 1);
  lcd.print(" 0   0   0      ");
}

void loop() {
  encoder();
  if (count == 1) {
    lcd.setCursor(0, 0);
    lcd.print("<HH> MM  SS  OK ");
    while (count == 1) {
      encoder();
      if (ledToggle) {
        h = 1;
        m = 0;
        s = 0;
        lcd.setCursor(0, 0);
        lcd.print(" HH  MM  SS  OK ");
        lcd.setCursor(1, 1);
        lcd.print(hh);
        lcd.setCursor(5, 1);
        lcd.print(mm);
        lcd.setCursor(9, 1);
        lcd.print(ss);
        lcd.setCursor(0, 1);
        lcd.print('<');
        lcd.setCursor(3, 1);
        lcd.print('>');
        
        while (ledToggle) {
          encoder();
          lcd.setCursor(1, 1);            
          lcd.print(hh);
        }
      }
      EEPROM.write(0, hh);
      h = 0;
      m = 0;
      s = 0;
      lcd.setCursor(0, 0);
      lcd.print("<HH> MM  SS  OK ");
      lcd.setCursor(0, 1);
      lcd.print(' ');
      lcd.setCursor(3, 1);
      lcd.print(' ');
    }
  } else if (count == 2) {
    lcd.setCursor(0, 0);
    lcd.print(" HH <MM> SS  OK ");
    while (count == 2) {
      encoder();
      if (ledToggle) {
        h = 0;
        m = 1;
        s = 0;
        lcd.setCursor(0, 0);
        lcd.print(" HH  MM  SS  OK ");
        lcd.setCursor(1, 1);
        lcd.print(hh);
        lcd.setCursor(5, 1);
        lcd.print(mm);
        lcd.setCursor(9, 1);
        lcd.print(ss);
        lcd.setCursor(4, 1);
        lcd.print('<');
        lcd.setCursor(7, 1);
        lcd.print('>');
        
        while (ledToggle) {
          encoder();
          lcd.setCursor(5, 1);            
          lcd.print(mm);
        }
      }
      EEPROM.write(1, mm);
      h = 0;
      m = 0;
      s = 0;
      lcd.setCursor(0, 0);
      lcd.print(" HH <MM> SS  OK ");
      lcd.setCursor(4, 1);
      lcd.print(' ');
      lcd.setCursor(7, 1);
      lcd.print(' ');
    }
  } else if (count == 3) {
    lcd.setCursor(0, 0);
    lcd.print(" HH  MM <SS> OK ");
    while (count == 3) {
      encoder();
      if (ledToggle) {
        h = 0;
        m = 0;
        s = 1;
        lcd.setCursor(0, 0);
        lcd.print(" HH  MM  SS  OK ");
        lcd.setCursor(1, 1);
        lcd.print(hh);
        lcd.setCursor(5, 1);
        lcd.print(mm);
        lcd.setCursor(9, 1);
        lcd.print(ss);
        lcd.setCursor(8, 1);
        lcd.print('<');
        lcd.setCursor(11, 1);
        lcd.print('>');
        
        while (ledToggle) {
          encoder();
          lcd.setCursor(9, 1);            
          lcd.print(ss);
        }
        EEPROM.write(2, ss);
        h = 0;
        m = 0;
        s = 0;
      }
      lcd.setCursor(0, 0);
      lcd.print(" HH  MM <SS> OK ");
      lcd.setCursor(8, 1);
      lcd.print(' ');
      lcd.setCursor(11, 1);
      lcd.print(' ');
    }
  } else if (count == 4) {
    lcd.setCursor(0, 0);
    lcd.print(" HH  MM  SS <OK>");
    while (count == 4) {
      encoder();
      if (ledToggle) {
        timer();
      }
    }
  }
}

void encoder() {
  current_state = digitalRead(outputA);
  if (current_state != previous_state) {
    if (digitalRead(outputB) != current_state) {
      if (count < 4 && ledToggle == 0)
        count++;
      else {
        if (h == 1 && hh < 99) {
          hh = hh + 1;
        } else if (m == 1 && mm < 480) {
          mm = mm + 1;
        } else if (s == 1 && ss < 59) {
          ss = ss + 1;
        }
      }
    } else {
      if (count > 1 && ledToggle == 0)
        count--;
      else {
        if (h == 1 && hh > 0) {
          hh = hh - 1;
        } else if (m == 1 && mm > 0) {
          mm = mm - 1;
        } else if (s == 1 && ss > 0) {
          ss = ss - 1;
        }
      }
    }
    Serial.print("Position: ");
    Serial.println(count);
  }
  previous_state = current_state;
}

void button_ISR() {
  buttonFlag = 1;
  if ((millis() - previousPress) > buttonDebounce && buttonFlag) {
    previousPress = millis();
    if (digitalRead(button) == LOW && previousState == HIGH) {
      ledToggle = !ledToggle;
      digitalWrite(led, ledToggle);
      previousState = LOW;
    } else if (digitalRead(button) == HIGH && previousState == LOW) {
      previousState = HIGH;
    }
    buttonFlag = 0;
  }
}

void timer() {
  digitalWrite(relay, HIGH); // Initially, keep the relay activated (active low logic)
  while (1) {
    lcd.setCursor(0, 0);
    lcd.print("Charging...     ");
    lcd.setCursor(0, 1);
    lcd.print("Time Left: ");
    lcd.print(hh);
    lcd.print(":");
    lcd.print(mm);
    lcd.print(":");
    lcd.print(ss);
    
    // Check if time is up (countdown reaches zero)
    if (ss == 0 && mm == 0 && hh == 0) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Charged");
      
      // Deactivate the relay when countdown is complete
      digitalWrite(relay, LOW); // Deactivate the relay when time is up
      
      // Clear stored timer values in EEPROM
      EEPROM.write(0, 0);
      EEPROM.write(1, 0);
      EEPROM.write(2, 0);
      
      // Stop the program (infinite loop)
      while (1); 
    }
    
    // Countdown logic
    if (ss == 0) {
      ss = 59;
      if (mm > 0) mm--;
      else if (hh > 0) hh--;
    }
    
    // Update EEPROM with remaining time
    EEPROM.write(0, hh);
    EEPROM.write(1, mm);
    EEPROM.write(2, ss);
    
    delay(1000); // Wait for 1 second
    ss--; // Decrement seconds
  }
}
