#include <LiquidCrystal.h>
#include <DHT.h>
#define DHTPIN A1
#define DHTTYPE DHT11
int relay1 = 4;
int relay2 = 5;
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal lcd(12, 11, 10, 9, 8, 7);


#include <PulseSensorPlayground.h>


const int PulseWire = A0;
int Threshold = 550;

int myBPM;
int buzzer = 3;
int ldr_sensor = 2;

PulseSensorPlayground pulseSensor;
String msg;
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  dht.begin();
  pinMode(buzzer, OUTPUT);
  pinMode(ldr_sensor, INPUT);
  pinMode(relay1, OUTPUT);
  pinMode( relay2, OUTPUT);
  digitalWrite(buzzer, LOW);
  digitalWrite(relay1, HIGH);
  digitalWrite(relay2, HIGH);
  pulseSensor.analogInput(PulseWire);
  pulseSensor.setThreshold(Threshold);

  lcd.clear();
  lcd.begin(16, 2);  // Initialize the non-I2C LiquidCrystal display
  lcd.setCursor(0, 0);
  lcd.print("Smart pillow ");
  delay(3000);
  if (pulseSensor.begin()) {
    // Serial.println("We created a pulseSensor Object !");  //This prints one time at Arduino power-up,  or on Arduino reset.
  }


}


void loop() {
  // put your main code here, to run repeatedly:
  digitalWrite(relay2, HIGH);
  int ldr_value = digitalRead(ldr_sensor);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("ldr_value: ");
  lcd.print(ldr_value);
  delay(2000);

  int temperature = dht.readTemperature();
  int humidity = dht.readHumidity();
  if (!isnan(temperature) && !isnan(humidity)) {

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp:");
    lcd.print(temperature);
    lcd.setCursor(0, 1);
    lcd.print("humi:");
    lcd.print(humidity);
    delay(2000);

  }
  if (pulseSensor.sawStartOfBeat()) {
    myBPM = pulseSensor.getBeatsPerMinute();

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Heartbeat:");
    lcd.setCursor(0, 1);
    lcd.print(myBPM);
    delay(2000);

    if (myBPM > 100) {
      digitalWrite(buzzer, HIGH);
      delay(2000);
      digitalWrite(buzzer, LOW);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Heartrate is");
      lcd.setCursor(0, 1);
      lcd.print("  Abnormal");
      delay(1000);
      msg = "abnormal heart rate";
      SendMessage();
      delay(2000);
      upload();
      delay(1000);
    }
  }
  if (temperature >= 35)
  {
    digitalWrite(relay1, LOW);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("High temperature ");
    lcd.setCursor(0, 1);
    lcd.print("fan on");
    digitalWrite(buzzer, HIGH);
    delay(2000);
    digitalWrite(buzzer, LOW);
    digitalWrite(relay1, HIGH);
    upload();
    delay(1000);
  }
  else if (ldr_value == 1) {
    digitalWrite(relay2, LOW);
    digitalWrite(buzzer, HIGH);
    delay(2000);
    digitalWrite(relay2, HIGH);
    digitalWrite(buzzer, LOW);
    upload();
    delay(1000);


  }
  else if (ldr_value == 0) {
    digitalWrite(relay2, HIGH);
    lcd.clear();
    lcd.print("Light off ");
    lcd.setCursor(0, 1);
    lcd.print("GOOD MORNING :)");
  }

}
void SendMessage()
{
  Serial.println("Setting the GSM in text mode");
  Serial.println("AT+CMGF=1\r");
  delay(2000);
  Serial.println("Sending SMS to the desired phone number!");
  Serial.println("AT+CMGS=\"+917075799644\"\r");
  // Replace x with mobile number
  delay(2000);
  Serial.println(msg);    // SMS Text
  delay(2000);
  Serial.write(26);               // ASCII code of CTRL+Z
  delay(2000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Message sent");
  delay(1000);
}
void upload()
{
  int ldr_value = digitalRead(ldr_sensor);
  int temperature = dht.readTemperature();
  int humidity = dht.readHumidity();
  Serial.println("AT");
  delay(1000);
  Serial.println("AT+CPIN?");
  delay(1000);
  Serial.println("AT+CREG?");
  delay(1000);
  Serial.println("AT+CGATT?");
  delay(1000);
  Serial.println("AT+CIPSHUT");
  delay(1000);
  Serial.println("AT+CIPSTATUS");
  delay(2000);
  Serial.println("AT+CIPMUX=0");
  delay(2000);
  Serial.println("AT+CSTT=\"Airtel Internet\"");//start task and setting the APN,
  //Serial.println("AT+CSTT=\"Airtel GPRS(40407)\"");//start task and setting the APN,
  delay(1000);
  Serial.println("AT+CIICR");//bring up wireless connection
  delay(6000);
  Serial.println("AT+CIFSR");//get local IP adress
  delay(1000);
  Serial.println("AT+CIPSPRT=0");
  delay(3000);
  Serial.println("AT+CIPSTART=\"TCP\",\"api.thingspeak.com\",\"80\"");//start up the connection
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("data sending...");
  lcd.setCursor(0, 1);
  lcd.print("to server");
  delay(10000);
  Serial.println("AT+CIPSEND");//begin send data to remote server
  delay(10000);

  String str = "GET https://api.thingspeak.com/update?api_key=H53T3675HILN1KGA&field1="  + String(ldr_value)
               + "&field2=" + String(temperature) + "&field3=" + String(humidity) + "&field4=" + String(myBPM); //+"&field5=" + String(n) +"&field6=" + String(o);//+ "&field2="+ "100";
  Serial.println(str);//begin send data to remote server
  delay(6000);
  Serial.write(26);//sending
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("DATA SENT");
  lcd.setCursor(0, 1);
  lcd.print("TO SERVER");
  delay(2000);
  lcd.clear();//waitting for reply, important! the time is base on the condition of internet
  Serial.println();
  Serial.println("AT+CIPSHUT");//close the connection
  delay(1000);
}
