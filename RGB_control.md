# arduino_learn

```c++
#include <Adafruit_NeoPixel.h>  //新增RGB程式庫


//紅(255,0,0)綠(0,255,0)藍(0,0,255)黃(255,255,0)紫(255,0,255)青(0,255,255)白(255,255,255)橙(255,128,0)

byte rgbpin=3,led_count=8; //rgbpin=rgb腳位 led_count為數量

byte Red[8]={ 255, 0  ,0,   255,255  ,0  ,255,255}; //宣告RGB陣列 配合上面顏色
byte Green[8]={0,  255,0   ,255 ,0,  255 ,255,128};
byte Blue[8]={ 0,  0,  255 ,0   ,255,255 ,255,0};

Adafruit_NeoPixel myrgb = Adafruit_NeoPixel (led_count, rgbpin, NEO_GRB + NEO_KHZ800);
//定義新的物件名稱為myrgb
//Adafruit_NeoPixel有三個參數,分別為LED的數量,硬體連接的腳位,以及LED採用RGB,800KHZ通訊訊號的速率

void setup() 
{
  int vrvalue=0,vrcount=0; //供可變電阻使用
  
  pinMode(rgbpin,OUTPUT);
  myrgb.clear();
  myrgb.show();  // clear，每個點的顏色清為0， show,顯示 2行組合成初始RGB不亮
  
  noInterrupts(); //暫停所有中斷
  TCCR1A=0;
  TCCR1B=0;
  TCNT1=65473;
  TCCR1B |= (1<<CS12);
  TIMSK1 |= (1<<TOIE1);
  interrupts(); //啟動所有中斷
  
  for(byte i=0;i<10;i++) timer[i]=0; //10個中斷(timer)預設為0ms
}

ISR(TIMER1_OVF_vect) //328IC內部timer1 溢位中斷
{
  byte i;
  TCNT1=65473; //65536-(16M/256/1000Hz(1ms))
  for(i=0;i<10;i++)
  {
    if(timer[i]>0)
    {
      timer[i]--;
    }
  }
}
//------------WS2812 全彩RGB的副程式,RGB燈條,分別以紅綠藍由下往上漸亮--------
void colorWipe(uint32_t c,uint8_t wait)  //c要輸入RGB顏色 , wait則輸入delay時間(ms)
{
  for(uint16_t i=0; i<myrgb.numPixels();i++)  //一個顏色使RGB漸亮,myrgb.numPixels()可改自己需要的數量 控制要亮幾顆
  {
    myrgb.setPixelColor(i,c);
    myrgb.show();
    delay(wait);
  }
}
//--------------分別顯示不同顏色RGB_副程式----------------------<br>
void rgbdata(byte r,byte g,byte b,byte count ) //r,g,b分別為輸入紅綠藍3色,配合上方RGB陣列使用 ,count為控制RGB亮哪顆
{
  myrgb.setPixelColor(count-1,myrgb.Color(r,g,b)); 
  myrgb.show();
}

測試程式:
void loop() 
{
  //第一個範例
  //8個RGB燈條,由下往上紅色漸亮,全滅後,由下往上綠色漸亮,全滅.....
       colorWipe(myrgb.Color(255,0,0),1000); //RED紅
       myrgb.clear();
       colorWipe(myrgb.Color(0,255,0),1000); //Green綠
       myrgb.clear();
       colorWipe(myrgb.Color(0,0,255),1000); //Blue藍
       myrgb.clear();
  //上面程式為RGB分別由第1顆漸亮至第8顆 顏色順序為紅綠藍 
  //當其中一個顏色跑完的時候,即清空RGB,讓他不亮
  
  
  //第二個範例
  
  for(int i=0;i<8;i++)rgbdata(Red[i],Green[i],Blue[i],i+1);
  
  //上面程式為 一次點亮8顆RGB LED 顏色由下到上順序為 紅綠藍黃紫青白橙
  //可在for迴圈當中加入delay(ms),可讓一顆一顆慢慢顯示
  
  //第三個範例
  //RGB加上可變電阻的應用,調可變電阻可讓RGB進行呼吸燈效果
    vrvalue=analogRead(A5); //讀取腳位A5的可變電阻值(0 ~ 1023)
    vrvalue=map(vrvalue,0,1023,0,255); //將0 ~ 1023轉成0~255
    rgbdata(vrvalue,0,0,1); //分別點亮8顆RGB 顏色為紅綠藍黃紫青白橙
    rgbdata(0,vrvalue,0,2);
    rgbdata(0,0,vrvalue,3);
    rgbdata(vrvalue,vrvalue,0,4);
    rgbdata(vrvalue,0,vrvalue,5);
    rgbdata(0,vrvalue,vrvalue,6);
    rgbdata(vrvalue,vrvalue,vrvalue,7);
    rgbdata(vrvalue,vrvalue/2,0,8);
    
}
```
