# arduino_learn

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
//--------------分別顯示不同顏色RGB_副程式----------------------
void rgbdata(byte r,byte g,byte b,byte count ) //r,g,b分別為輸入紅綠藍3色,配合上方RGB陣列使用 ,count為控制RGB亮哪顆
{
  myrgb.setPixelColor(count-1,myrgb.Color(r,g,b)); 
  myrgb.show();
}

測試程式:
void loop() 
{
  //--------------執行前,8個RGB燈條,由下往上紅色漸亮,全滅後,由下往上綠色漸亮,全滅......---
       colorWipe(myrgb.Color(255,0,0),1000); //RED紅
       myrgb.clear();
       colorWipe(myrgb.Color(0,255,0),1000); //Green綠
       myrgb.clear();
       colorWipe(myrgb.Color(0,0,255),1000); //Blue藍
       myrgb.clear();
       sw_status=20;
}
  
