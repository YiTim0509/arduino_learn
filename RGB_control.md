# arduino_learn
新增RGB程式庫
```c++
#include <Adafruit_NeoPixel.h>
```


rgbpin=rgb腳位 led_count為數量
```c++
byte rgbpin=3,led_count=8; 
```
紅(255,0,0)綠(0,255,0)藍(0,0,255)黃(255,255,0)紫(255,0,255)青(0,255,255)白(255,255,255)橙(255,128,0)<br>
宣告RGB陣列 配合上面顏色
```c++
byte Red[8]={ 255, 0  ,0,   255,255  ,0  ,255,255}; 
byte Green[8]={0,  255,0   ,255 ,0,  255 ,255,128};
byte Blue[8]={ 0,  0,  255 ,0   ,255,255 ,255,0};
```
定義新的物件名稱為myrgb<br>
Adafruit_NeoPixel有三個參數,分別為LED的數量,硬體連接的腳位,以及LED採用RGB,800KHZ通訊訊號的速率
```c++
Adafruit_NeoPixel myrgb = Adafruit_NeoPixel (led_count, rgbpin, NEO_GRB + NEO_KHZ800);
```

供可變電阻使用
```c++
int vrvalue=0,vrcount=0;
```

clear，每個點的顏色清為0， show,顯示 2行組合成初始RGB不亮
```c++
myrgb.clear();
myrgb.show();
```
暫停所有中斷
```c++
noInterrupts(); 
```
啟動所有中斷
```c++
interrupts(); 
```

10個中斷(timer)預設為0ms
```c++
for(byte i=0;i<10;i++) timer[i]=0;
```
328IC內部timer1 溢位中斷<br>
65536-(16M/256/1000Hz(1ms))
```c++
ISR(TIMER1_OVF_vect) 
{
  byte i;
  TCNT1=65473;
  for(i=0;i<10;i++)
  {
    if(timer[i]>0)
    {
      timer[i]--;
    }
  }
}
```

WS2812 全彩RGB的副程式,RGB燈條,分別以紅綠藍由下往上漸亮<br>
c要輸入RGB顏色 , wait則輸入delay時間(ms)<br>
一個顏色使RGB漸亮,myrgb.numPixels()可改自己需要的數量 控制要亮幾顆
```c++
void colorWipe(uint32_t c,uint8_t wait)
{
  for(uint16_t i=0; i<myrgb.numPixels();i++)
  {
    myrgb.setPixelColor(i,c);
    myrgb.show();
    delay(wait);
  }
}
```

分別顯示不同顏色RGB_副程式<br>
r,g,b分別為輸入紅綠藍3色,配合上方RGB陣列使用 ,count為控制RGB亮哪顆
```c++
void rgbdata(byte r,byte g,byte b,byte count ) 
{
  myrgb.setPixelColor(count-1,myrgb.Color(r,g,b)); 
  myrgb.show();
}
```
讀取腳位A5的可變電阻值(0 ~ 1023)
```c++
vrvalue=analogRead(A5);
```
將0 ~ 1023轉成0~255
```c++
vrvalue=map(vrvalue,0,1023,0,255);
```
分別點亮8顆RGB 顏色為紅綠藍黃紫青白橙
```c++
rgbdata(vrvalue,0,0,1); 
rgbdata(0,vrvalue,0,2);
rgbdata(0,0,vrvalue,3);
rgbdata(vrvalue,vrvalue,0,4);
rgbdata(vrvalue,0,vrvalue,5);
rgbdata(0,vrvalue,vrvalue,6);
rgbdata(vrvalue,vrvalue,vrvalue,7);
rgbdata(vrvalue,vrvalue/2,0,8);
```



# 測試程式:
程式1:<br>
8個RGB燈條,由下往上紅色漸亮,全滅後,由下往上綠色漸亮,全滅.....
```c++
colorWipe(myrgb.Color(255,0,0),1000);
myrgb.clear();
colorWipe(myrgb.Color(0,255,0),1000);
myrgb.clear();
colorWipe(myrgb.Color(0,0,255),1000);
myrgb.clear();
```
上面程式為RGB分別由第1顆漸亮至第8顆 顏色順序為紅綠藍<br>
當其中一個顏色跑完的時候,即清空RGB,讓他不亮<br>


程式2:<br>
一次點亮8顆RGB LED 顏色由下到上順序為 紅綠藍黃紫青白橙<br>
可在for迴圈當中加入delay(ms),可讓一顆一顆慢慢顯示
```c++
for(int i=0;i<8;i++)rgbdata(Red[i],Green[i],Blue[i],i+1);
```

程式3:<br>
RGB加上可變電阻的應用,調可變電阻可讓RGB進行呼吸燈效果<br>
```c++
vrvalue=analogRead(A5); 
vrvalue=map(vrvalue,0,1023,0,255); 
rgbdata(vrvalue,0,0,1); 
rgbdata(0,vrvalue,0,2);
rgbdata(0,0,vrvalue,3);
rgbdata(vrvalue,vrvalue,0,4);
rgbdata(vrvalue,0,vrvalue,5);
rgbdata(0,vrvalue,vrvalue,6);
rgbdata(vrvalue,vrvalue,vrvalue,7);
rgbdata(vrvalue,vrvalue/2,0,8);
```



# 全部程式統整
注意:3個測試功能不能同時運作<br>
可用旗標引導哪個功能運作
```c++
#include <Adafruit_NeoPixel.h>
byte rgbpin=3,led_count=8; 
int vrvalue=0,vrcount=0;
byte Red[8]={ 255, 0  ,0,   255,255  ,0  ,255,255}; 
byte Green[8]={0,  255,0   ,255 ,0,  255 ,255,128};
byte Blue[8]={ 0,  0,  255 ,0   ,255,255 ,255,0};

Adafruit_NeoPixel myrgb = Adafruit_NeoPixel (led_count, rgbpin, NEO_GRB + NEO_KHZ800);

void setup() 
{
  int vrvalue=0,vrcount=0; 
  
  pinMode(rgbpin,OUTPUT);
  myrgb.clear();
  myrgb.show();
  
  noInterrupts(); 
  TCCR1A=0;
  TCCR1B=0;
  TCNT1=65473;
  TCCR1B |= (1<<CS12);
  TIMSK1 |= (1<<TOIE1);
  interrupts(); 
  
  for(byte i=0;i<10;i++) timer[i]=0;
}

ISR(TIMER1_OVF_vect) 
{
  byte i;
  TCNT1=65473; 
  for(i=0;i<10;i++)
  {
    if(timer[i]>0)
    {
      timer[i]--;
    }
  }
}

void colorWipe(uint32_t c,uint8_t wait)
{
  for(uint16_t i=0; i<myrgb.numPixels();i++)
  {
    myrgb.setPixelColor(i,c);
    myrgb.show();
    delay(wait);
  }
}

void rgbdata(byte r,byte g,byte b,byte count )
{
  myrgb.setPixelColor(count-1,myrgb.Color(r,g,b)); 
  myrgb.show();
}


void loop() 
{
  
  
   colorWipe(myrgb.Color(255,0,0),1000);
   myrgb.clear();
   colorWipe(myrgb.Color(0,255,0),1000);
   myrgb.clear();
   colorWipe(myrgb.Color(0,0,255),1000);
   myrgb.clear();

   for(int i=0;i<8;i++)rgbdata(Red[i],Green[i],Blue[i],i+1);

  
  
   vrvalue=analogRead(A5); 
   vrvalue=map(vrvalue,0,1023,0,255); 
   rgbdata(vrvalue,0,0,1); 
   rgbdata(0,vrvalue,0,2);
   rgbdata(0,0,vrvalue,3);
   rgbdata(vrvalue,vrvalue,0,4);
   rgbdata(vrvalue,0,vrvalue,5);
   rgbdata(0,vrvalue,vrvalue,6);
   rgbdata(vrvalue,vrvalue,vrvalue,7);
   rgbdata(vrvalue,vrvalue/2,0,8);
    
}
```
