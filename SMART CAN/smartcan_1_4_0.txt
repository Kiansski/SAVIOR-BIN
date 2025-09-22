//libraries
  #include <Wire.h>
  #include <LiquidCrystal_I2C.h>
  #include <Servo.h>
//library assignment
  LiquidCrystal_I2C lcd(0x27, 16, 2);
  Servo myservo;
//assignment
  const int trigPin = 2;
  const int echoPin = 3;
  const int wodSensor = A3; 
  const int wodBuzzer = 5;
  const int micPin = 12;
  const int sixHours = 10000;
//float is used to get the exact calculation with decimals (no float = whole number)
  float duration;
  float distanceCM;
//unsigned long
  unsigned long prevTime1 = millis();
  unsigned long prevTime2 = millis();
  unsigned long wetStarttime = 0; 
//boolean(true or false)
  bool lidOpen = false;
  bool wetstartTimer = false;

void setup(){
//baud of smonitor
  Serial.begin(9600);
//for us sensor
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
//for wod sensor
  pinMode(wodSensor, INPUT);
  pinMode(wodBuzzer, OUTPUT);
//mic sensor
  pinMode(micPin, INPUT);
//for lcd 2 row 16 col
  lcd.init();
  lcd.backlight();
//lcd startup text (1)
  lcd.setCursor(0,0);
  lcd.print("ETHICAL CAN is");
  lcd.setCursor(0,1);
  lcd.print("Starting...");
  delay(3000);
//lcd startup text(2)
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("System Started");
  delay(700);
  lcd.clear();
//servo pin
  myservo.attach(11);
//state default display
    lcd.setCursor(0,1);
    lcd.print("State: Undefined");

}

//activator of programs
void loop(){

  //activate program 1
    // clean signal
    digitalWrite(trigPin, LOW); 
    delayMicroseconds(10);  
  // send trigger signal
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
  //recieve the signal pulse
    duration = pulseIn(echoPin, HIGH);
  //convert the recieved pulse to measurements
    distanceCM = (duration * 0.034) / 2;
  //print the distance to the serial monitor
    Serial.print("Distance: ");
    Serial.print(distanceCM);
    Serial.print(" cm");
    Serial.println();
  //activate program 2 w/ smonitor (on line 79)
    int value = analogRead(wodSensor);
    Serial.print("Water: ");
    Serial.print(value);
    Serial.println();
  //activate program 3 w/ smonitor (on line 139)
    int micVal = digitalRead(micPin);
    Serial.print("value:");
    Serial.print(micVal);
    Serial.println();
//condition for action to lcd
  //wod Sensor
    if (value <= 50){
      lcd.setCursor(0, 0);
      lcd.print("                ");
      lcd.setCursor(0, 0);
      lcd.print("Type: DRY"); //what to show on the lcd
    }else if(value > 50){
      lcd.setCursor(0, 0);
      lcd.print("                ");
      lcd.setCursor(0, 0);
      lcd.print("Type: WET"); //what to show on the lcd
    }

  //Close lid    
    if ((millis() - prevTime1 >= 6000) && lidOpen && distanceCM >= 15){
      myservo.write(0);
      prevTime1 = millis();
      lidOpen = false;
      lcd.setCursor(90, 1);
      lcd.print("                ");
      lcd.setCursor(0, 1);
      lcd.print("State: Closed"); //show on lcd
    } 
  //Open lid
    if ((millis() - prevTime2 >= 1000) && !lidOpen && distanceCM < 10){
      myservo.write(90);
      prevTime1 = millis(); //to reset the time on prev 1
      prevTime2 = millis();
      lidOpen = true;
      lcd.setCursor(0, 1);
      lcd.print("                ");
      lcd.setCursor(0, 1);
      lcd.print("State: Open"); //show on lcd
    }
  //wod timer  mechanism
      //start when when detected
      if(value > 50 && !wetstartTimer){
        wetStarttime = millis();
        wetstartTimer = true;
      }

      //if dry stop timer off the buzzer
      if(value <= 50){
        wetstartTimer = false;
        analogWrite(wodBuzzer, 0);

      }
      if ((millis() - wetStarttime >= sixHours) && wetstartTimer){
        analogWrite(wodBuzzer, 240);
        lcd.setCursor(0, 0);
        lcd.print("                ");
        lcd.setCursor(0, 0);
        lcd.print("Smell detected!!");
      } 
  //Close lid (By Microphone)
    if ((millis() - prevTime1 >= 6000) && lidOpen && micVal == 0){
      myservo.write(0);
      prevTime1 = millis();
      lidOpen = false;
      lcd.setCursor(90, 1);
      lcd.print("                ");
      lcd.setCursor(0, 1);
      lcd.print("State: Closed"); //show on lcd
    } 
  //Open lid (By Microphone)
    if ((millis() - prevTime2 >= 1000) && !lidOpen && micVal == 1){
      myservo.write(90);
      prevTime1 = millis(); //to reset the time on prev 1
      prevTime2 = millis();
      lidOpen = true;
      lcd.setCursor(0, 1);
      lcd.print("                ");
      lcd.setCursor(0, 1);
      lcd.print("State: Open"); //show on lcd
    }
  //interval of the reading and lcd flickering
    delay(200);
}




