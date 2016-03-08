---
title: Pantower-PM25-Sensor
date: '2016-03-08'
description:
categories:
- Blog
tags:
- Hardware
- Arduino

---

# AQI

杭城的空气每况日下，同时也想看看一些有机气体的指数，首当其冲就是甲醛，看了下国内外的传感器，发现Pantower有款传感器支持甲醛整合的，于是马云家入手了一个（型号是PMS5003S，不带S的比较多，带S就直接添加了甲醛传感器）

在玩的过程中遇到了一些问题，记录下

# 上传和显示数据串口冲突

接上Arduino uno，通过I2C链接LCD1602（手头只有这个，暂时只好先用这个显示数据），由于PMS5003S的数据都是串口输出，通过查看Spec，数据包每帧都是32个字节，而uno上只有一个RX接口，因此必须在编译上传时拔掉接线。当然也有其他的办法：譬如用软串口，参考一下SoftwareSerial；譬如换块mega的板子。

# I2C和串口冲突

在使用串口读数据和使用i2c设置lcd显示的时候，在同一个loop里面全都执行了，导致串口读数据丢失，害得我查了一整天，后来直接打印串口数据，才找到这个问题。
PMS5003S串口的数据示例：

> 1:66|2:77|3:0|4:28|5:0|6:38|7:0|8:53|9:0|10:61|11:0|12:29|13:0|14:42|15:0|
> 16:54|17:24|18:177|19:6|20:242|21:1|22:15|23:0|24:20|25:0|26:4|27:0|28:2|29:0|30:4|31:3|32:175|
> 1:66|2:77|3:0|4:28|5:0|6:38|7:0|8:52|9:0|10:59|11:0|12:29|13:0|14:41|15:0|
> 16:53|17:23|18:175|19:6|20:195|21:1|22:10|23:0|24:16|25:0|26:2|27:0|28:2|29:0|30:4|31:3|32:109|
> 1:66|2:77|3:0|4:28|5:0|6:36|7:0|8:52|9:0|10:62|11:0|12:28|13:0|14:41|15:0|
> 16:54|17:23|18:67|19:6|20:167|21:1|22:21|23:0|24:22|25:0|26:4|27:0|28:2|29:0|30:4|31:2|32:249|

主要解决办法就是i2c设置lcd显示，让它一秒执行一次，不要每个loop都执行（新手原因^_^）

# 实现代码

新建一个arduino工程，拷进去：（使用了LiquidCrystal_I2C库）
<pre><code>
#include &lt;Wire.h&gt;
#include &lt;LiquidCrystal_I2C.h&gt;

LiquidCrystal_I2C lcd(0x27, 16, 2);
#define receiveDatIndex 24

uint8_t receiveDat[receiveDatIndex]; //receive data from the air detector module

void setup() {
  Serial.begin(9600);
  // put your setup code here, to run once:
  lcd.begin();
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print("for JOJO~");
  lcd.setCursor(0,1);
}

int i = 0;

static unsigned char ucRxBuffer[250];
static unsigned char ucRxCnt = 0;
long _P25;
long _P10;
long _P100;
float _HCHO;
long _length;

//read data for pantower sensor including HCHO
void getPM25(unsigned char ucData) {
  ucRxBuffer[ucRxCnt++] = ucData;

  if (ucRxBuffer[0] != 0x42 && ucRxBuffer[1] != 0x4D)  {
ucRxCnt = 0;
  }

  //for debug data
  Serial.print(ucRxCnt);
  Serial.print(":");
  Serial.print(ucData);
  Serial.print("|");
  if (ucRxCnt > 31) {
//usa standard
//_P10 = (float)ucRxBuffer[4] * 256 + (float)ucRxBuffer[5];
//_P25 = (float)ucRxBuffer[6] * 256 + (float)ucRxBuffer[7];
//_P100 = (float)ucRxBuffer[8] * 256 + (float)ucRxBuffer[9];
//china standard
_P10 = (float)ucRxBuffer[10] * 256 + (float)ucRxBuffer[11];
_P25 = (float)ucRxBuffer[12] * 256 + (float)ucRxBuffer[13];
_P100 = (float)ucRxBuffer[14] * 256 + (float)ucRxBuffer[15];
_HCHO = ((float)ucRxBuffer[28] * 256 + (float)ucRxBuffer[29])/1000;
Serial.println();
ucRxCnt = 0;
  }
}

void readValue() {
unsigned int length = Serial.available();
if (length > 0)  
{
getPM25(Serial.read());
}
}

void loop() {
// put your main code here, to run repeatedly:
readValue();
delay(3);
static unsigned long OledTimer=millis();   //every 0.5s update the temperature and humidity from DHT11 sensor
if (millis() - OledTimer >=1000) 
{
  //if at every loop,print data to LCD, it will be conflict for I2C and serial
  // Print a message to the LCD.
  OledTimer=millis(); 
  lcd.setCursor(0,0);
  lcd.print("PM2.5=");
  lcd.print(_P25);
  lcd.print("ug/m3 ");
  lcd.setCursor(0,1);
  //lcd.clear();
  lcd.print("HCHO=");
  lcd.print(_HCHO);
  lcd.print("mg/m3");
}
}
</code></pre>



