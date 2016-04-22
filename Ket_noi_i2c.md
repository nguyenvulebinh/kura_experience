# Kết nối Raspberry với Arduino thông qua Kura sử dụng kết nối i2c nhận dữ liệu từ sensor nhiệt ẩm

##Chuẩn bị thiết bị và tiến hành kết nối lại như hình: 
+ **Raspberry Pi**
+ **Arduino**
+ **Cảm biến nhiệt ẩm DHT11**

![alt tag](https://github.com/nguyenvulebinh/kura_experience/blob/master/thietbi.jpg)

##Phần mềm cài đặt trên các thiết bị:

+ **Raspberry Pi:** Cài đặt chương trình trên Arduino

```c
#include <dht.h>
#include "Wire.h"
dht DHT1;
#define D1 7
#define DS3231_I2C_ADDRESS 0x68
#define SLAVE_ADDRESS 0x39

int number;
int count;
byte bSec, bMin, bHour, bDate, bMonth, bYear, bHum, bTemp;

//convert dec to binary coded decimal
byte decToBcd(byte val)
{
  return( (val/10*16) + (val%10) );
}
// Convert binary coded decimal to normal decimal numbers
byte bcdToDec(byte val)
{
  return( (val/16*10) + (val%16) );
}

void setDS3231time(byte bSec, byte bMin, byte bHour, byte dayOfWeek, byte
bDate, byte bMonth, byte bYear)
{
  // sets time and date data to DS3231
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0); // set next input to start at the bSecs register
  Wire.write(decToBcd(bSec)); // set bSecs
  Wire.write(decToBcd(bMin)); // set bMins
  Wire.write(decToBcd(bHour)); // set bHours
  Wire.write(decToBcd(dayOfWeek)); // set day of week (1=Sunday, 7=Saturday)
  Wire.write(decToBcd(bDate)); // set date (1 to 31)
  Wire.write(decToBcd(bMonth)); // set bMonth
  Wire.write(decToBcd(bYear)); // set bYear (0 to 99)
  Wire.endTransmission();
}
void readDS3231time(byte *bSec,
byte *bMin,
byte *bHour,
byte *dayOfWeek,
byte *bDate,
byte *bMonth,
byte *bYear)
{
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0); // set DS3231 register pointer to 00h
  Wire.endTransmission();
  Wire.requestFrom(DS3231_I2C_ADDRESS, 7);
  // request seven bytes of data from DS3231 starting from register 00h
  *bSec = bcdToDec(Wire.read() & 0x7f);
  *bMin = bcdToDec(Wire.read());
  *bHour = bcdToDec(Wire.read() & 0x3f);
  *dayOfWeek = bcdToDec(Wire.read());
  *bDate = bcdToDec(Wire.read());
  *bMonth = bcdToDec(Wire.read());
  *bYear = bcdToDec(Wire.read());
}
void displayTime()
{ 
  byte  dayOfWeek;
  // retrieve data from DS3231
  readDS3231time(&bSec, &bMin, &bHour, &dayOfWeek, &bDate, &bMonth,
  &bYear);
  // send it to the serial monitor
  Serial.print(bHour, DEC);
  // convert the byte variable to a decimal number when displayed
  Serial.print(":");
  if (bMin<10)
  {
    Serial.print("0");
  }
  Serial.print(bMin, DEC);
  Serial.print(":");
  if (bSec<10)
  {
    Serial.print("0");
  }
  Serial.print(bSec, DEC);
  Serial.print(" ");
  Serial.print(bDate, DEC);
  Serial.print("/");
  Serial.print(bMonth, DEC);
  Serial.print("/20");
  Serial.print(bYear, DEC);
  Serial.print("  ");
  switch(dayOfWeek){
  case 1:
    Serial.println("Sunday");
    break;
  case 2:
    Serial.println("Monday");
    break;
  case 3:
    Serial.println("Tuesday");
    break;
  case 4:
    Serial.println("Wednesday");
    break;
  case 5:
    Serial.println("Thursday");
    break;
  case 6:
    Serial.println("Friday");
    break;
  case 7:
    Serial.println("Saturday");
    break;
  }
}

void displayValue(char strName[], int value, char strUnit[])
{
    Serial.print(strName);
    Serial.print(value);
    Serial.println(strUnit);
}

void setTime(char cTime[])
{
  int dM, dW, mo, ye;
  int arr[3]={};
  dM = 12;
  dW = 3;
  ye = 16;
  mo = 4;  
  char temp;
  int i=0;
  int j=0;
  temp = cTime[0];
  while (temp != '\0')
  {    
    if ( temp == ':')
      j++;
    else
      arr[j] = arr[j]*10 + temp - '0';
    i++;
    temp = cTime[i];
  }
 setDS3231time(arr[2],arr[1],arr[0],dW,dM,mo,ye); 
}

void receiveData(int byteCount){
  while(Wire.available()) {
    number = Wire.read();
    Serial.print("data received: ");
    Serial.println(number);
    if (number = 1)
      digitalWrite(13, HIGH);
    else 
      digitalWrite(13, LOW);
  }
}

// callback for sending data
void sendData(){
//  Wire.write(bSec);
//  Wire.write(bMin);
//  Wire.write(bHour);
//  Wire.write(bDate);
//  Wire.write(bMonth);
//  Wire.write(bYear);
   
  count ++;
  if (count % 2 == 0)
  {
    displayValue("Send Humidity1   : ",bHum,"%");
    Wire.write(bHum);
  }
  else 
  {
    displayValue("Send Temperature1: ",bTemp,"C");
    Wire.write(bTemp);
  }
}

void getData()
{
    DHT1.read11(D1);
    bTemp = DHT1.temperature;
    bHum = DHT1.humidity;
}

void setup()
{
    Serial.begin(9600);
    Serial.println("begin");
    Wire.begin(SLAVE_ADDRESS);
    count = 0;
// define callbacks for i2c communication
    Wire.onReceive(receiveData);
    Wire.onRequest(sendData);
}

void loop()
{
    
    Serial.println("_____________________________________________________________________");
    getData();
    //displayTime();        
//    displayValue("Temperature1: ",bTemp,"C");
//    displayValue("Humidity1   : ",bHum,"%");
    delay(5000);
}
```





##Kết quả
![alt tag](https://github.com/nguyenvulebinh/kura_experience/blob/master/ketqua.jpg)
