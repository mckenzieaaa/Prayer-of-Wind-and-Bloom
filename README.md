# Prayer-of-Wind-and-Bloom
An interactive installation exploring Dong culture's worship of nature deities through wind, light, and sound.

## Concept & Inspiration 
This work is inspired by the polytheistic beliefs of the Dong people, specifically the worship of the Flower Goddess ("Sa Suj") and the Wind Deity. The floral pattern is derived from Dong bridal attire, symbolizing the Flower Goddess and her blessings for health and fertility. Wind, an invisible natural force, represents the presence of the Wind Deity.

The installation creates a dialogue between nature and humans: when wind flows through the space, it activates the system, causing lights to breathe and Dong Grand Songs to play. This mirrors the Dong belief in communicating with deities through natural elements and ritual sounds.

## Technical Implementation 
The project integrates **soft circuitry** with traditional Dong textile crafts, and is controlled by an **Arduino microcontroller**. It demonstrates skills in **sensor data processing, multi-output control (lights + sound), and state management**.

### Software & Libraries 
- **Arduino IDE**
- **Adafruit_NeoPixel.h** (for controlling the LED strips) 
- **SoftwareSerial.h** (for communication with the MP3 module)

## Code Modules 

### 1. Airflow Sensor Reader 
**Function:** Reads the digital signal from the airflow sensor.
int airflow=0;
void setup() {
 Serial.begin(9600);
 pinMode(5,INPUT);
}

void loop() {
airflow=digitalRead(5);
Serial.print("airflow= ");Serial.println(airflow,DEC);
delay(500);
} 

### 2. Main Light & Sensor Control 
**Function:** Controls LED breathing effect based on airflow sensor input. 
#include <Adafruit_NeoPixel.h> // 定义RGB LED相关参数
#define LED_PIN    10        // LED连接到D10端口
#define NUM_LEDS   4         // 4个LED
#define DELAY_TIME 500       // 依次亮起的延时时间（毫秒）

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);

int brightness = 0;          // 当前亮度
int fadeAmount = 2;          // 渐变步长
const int maxBrightness = 150; // 最大亮度值
unsigned long startTime;      // 计时器起始时间

void setup() {
    strip.begin();
    strip.show();
}

void loop() {
    // 依次点亮LED
    for(int i = 0; i < NUM_LEDS; i++) {
        strip.setPixelColor(i, strip.Color(255, 180, 0)); // 橙黄色
        strip.show();
        delay(DELAY_TIME);
    }
    
    delay(1000); // 全亮状态停留一秒
    
    // 依次熄灭LED
    for(int i = 0; i < NUM_LEDS; i++) {
        strip.setPixelColor(i, strip.Color(0, 0, 0)); // 熄灭
        strip.show();
        delay(DELAY_TIME);
    }
    
    delay(500); // 全灭状态停留半秒
    
    // 开始计时并进入呼吸模式
    startTime = millis();
    
    // 持续10秒的呼吸效果
    while(millis() - startTime < 10000) { // 10000ms = 10秒
        breatheEffect();
        delay(40);
    }
    
    // 全灭
    clearAll();
    delay(1000); // 暂停一秒后重新开始
}

// 橙黄色呼吸灯效果
void breatheEffect() {
    brightness = brightness + fadeAmount;
    if (brightness <= 0 || brightness >= maxBrightness) {
        fadeAmount = -fadeAmount;
    }
    
    // 设置橙黄色呼吸效果 (R:255, G:180, B:0)
    for(int i = 0; i < NUM_LEDS; i++) {
        int r = (brightness * 255) / maxBrightness;  // 红色分量
        int g = (brightness * 180) / maxBrightness;  // 绿色分量
        int b = 0;                                   // 蓝色分量
        strip.setPixelColor(i, strip.Color(r, g, b));
    }
    strip.show();
}

// 清除所有LED
void clearAll() {
    strip.clear();
    strip.show();
}

### 3. Audio Player Control 
**Function:** Plays/stops Dong Grand Songs via touch sensor.
#include <SoftwareSerial.h>

SoftwareSerial Serial1(10, 11);  // RX, TX

#define TOUCH_PIN A1

bool lastTouchState = false;
bool isPlaying = false;
unsigned char currentTrack = 0x01;  // 当前播放的音轨

void setup() {
  Serial.begin(115200);  // 用于调试输出
  Serial1.begin(9600);
  pinMode(TOUCH_PIN, INPUT);
  
  Serial.println("初始化语音模块");
  setVolume(0x1E);  // 设置最大音量
  delay(1000);
  Serial.println("初始化完成");
}

void loop() {
  bool touchState = digitalRead(TOUCH_PIN) == HIGH;
  
  if (touchState && !lastTouchState) {  // 检测到按下（上升沿）
    if (isPlaying) {
      Serial.println("停止播放");
      stopAudio();
      isPlaying = false;
    } else {
      Serial.println("开始播放");
      playAudio(currentTrack);
      isPlaying = true;
    }
    delay(50);  // 消抖
  }
  
  lastTouchState = touchState;
}

void playAudio(unsigned char track) {
  Serial.println("尝试播放音频: " + String(track) + ".mp3");
  unsigned char play[6] = {0xAA,0x07,0x02,0x00,track,track+0xB3};
  Serial1.write(play,6);
  currentTrack = track;
}

void stopAudio() {
  Serial.println("停止音频");
  unsigned char stop[4] = {0xAA,0x04,0x00,0xAE};
  Serial1.write(stop,4);
}

void setVolume(unsigned char vol) {
  Serial.println("设置音量: " + String(vol));
  unsigned char volume[5] = {0xAA,0x13,0x01,vol,vol+0xBE};
  Serial1.write(volume,5);
}
