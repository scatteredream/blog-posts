---
name: iot-design
title: 物联网毕设构思
date: 2025-05-31
---




| 层次            | 具体内容                                                     | 技术选型                                                     |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **ZigBee 节点** | ZigBee 模块（如 XBee、CC2530）采集车内环境数据，定时发送到网关 |                                                              |
| **网关边缘端**  | 树莓派/工控机用 Java 实现：ZigBee 协议解析、数据缓存、预处理（异常检测、局部计算）使用 MQTT/HTTP 向后端推送 | Java + jSerialComm（串口库） + Eclipse Paho（MQTT 客户端）   |
| **后端服务**    | Spring Boot 构建 RESTful API，接收网关推送的数据，存入数据库；提供前端查询接口 | Spring Boot + Spring Web                                     |
| **数据库**      | MySQL（车辆基础信息）、InfluxDB（时序数据，如速度、温度、油量变化） | Spring Data JPA + MySQL + InfluxDB                           |
| **前端**        | Vue 搭建 Web 页面，显示实时数据和历史趋势；开发 Flutter 跨平台应用 | Flutter 用 `mqtt_client` 或 `http` 库连接后端。WebSocket 实时推送 |

# 系统功能

**远程控制**（前端app控制后端）

- ✅ 远程开锁 / 解锁
- ✅ 远程启动 / 熄火
- ✅ 远程空调 / 制热启动
- ✅ 远程开关充电、查看充电状态
- ✅ 远程寻车（闪灯、鸣笛）

📊 **历史与分析**

- ✅ 历史行驶轨迹（某段时间内 GPS 路线）
- ✅ 历史速度曲线、能耗曲线
- ✅ 历史报警记录（超速、急刹车、碰撞、胎压异常）

⚠ **异常与报警**

- ✅ 实时推送：车门未锁、胎压低、碰撞报警
- ✅ 消息中心：历史报警记录
- ✅ Flutter 本地通知或 App 内消息提醒

🔧 **车辆管理**

- ✅ 车辆绑定 / 多车切换
- ✅ 基本信息：车牌号、型号、VIN、出厂时间
- ✅ 远程升级（仅展示为功能，毕业设计中可省略实现）

------

- ✨ 为什么选用 ZigBee 而不是 WiFi、蓝牙（功耗、组网能力）
- ✨ 边缘计算做了哪些处理（减轻云端压力、实时响应）
- ✨ 系统架构设计的合理性（分层、解耦、可扩展性）
- ✨ 实际测试（比如模拟多个 ZigBee 节点、测试边缘侧响应速度）

# 硬件层（车载设备模拟）

- ✅ 实时车辆位置（GPS 定位 + 地图显示）
- ✅ 当前车速、里程、剩余电量 / 油量
- ✅ 车内温度、胎压、碰撞状态
- ✅ 车门、车窗、后备箱、充电口状态（开/关/异常）

| 要素          | 模拟方法                                                     |      |
| ------------- | ------------------------------------------------------------ | ---- |
| **车速**      | 用一个旋转编码器（模拟轮速），或直接 MCU 中用程序生成速度值  |      |
| **总里程**    | MCU 内部累加（速度 × 时间），输出里程值                      |      |
| **温度**      | 接温度传感器（如 **DS18B20**），或程序内写死模拟数据         |      |
| **油量/电量** | 接**可调电位器**模拟传感器输入，或 MCU 内部计算              |      |
| **碰撞**      | 用**震动传感器**（震动检测模块 SW-420），或按键模拟          |      |
| **定位**      | 如果需要 GPS，可**加个 GPS 模块**（或直接用 MCU 固定值模拟） |      |
| Arduino Uno   | 控制核心，生成模拟数据，驱动传感器，组装数据包，通过串口发给 **ZigBee 模块**（XBee、CC2530） |      |
| Zigbee 模块   | XBee，无线发送数据到网关，与网关 ZigBee 模块建立网络（自组网或点对点） |      |

## 通信技术选型

ZigBee 的特点：

1.  ✅ 低功耗
2.  ✅ 低速率（最大 250 kbps）
3.  ✅ 短距离（几十米，室内更稳定）
4.  ✅ 点对点 / 星型 / Mesh 网络

但它 **主要用于家庭、工业短距无线**，比如：智能家居（灯、传感器、窗帘）、工厂自动化（温湿度、振动监测）、小型物联网场景（传感器网络）

在车联网（特别是商用/工业级）场景，ZigBee **并不主流**，原因：

-  ❌ 车速快、移动性强，ZigBee 不稳定
-  ❌ 带宽太小，不适合传视频、图片等
-  ❌ 没有直接连接互联网能力，需要网关中转

> 业内车联网实践用什么？

车联网（V2X, Vehicle-to-Everything）主流通信技术：

-  ✅ **蜂窝网络（4G/5G）** → 广域通信、直连云端
-  ✅ **Wi-Fi / DSRC（专用短程通信）** → 车-车、车-路直接通信
-  ✅ **蓝牙** → 车内设备短距通信（手机、耳机、娱乐系统）
-  ✅ **CAN 总线 / LIN 总线** → 车内模块互联（仪表盘、发动机、ECU）
-  ✅ **以太网（车载以太网）** → 高速数据、ADAS 系统

## 命令处理

Java后端服务收到前端的JSON请求（controller），`{"deviceId": "car-001",  "action": "SET_SPEED", "value": 50}`，翻译成对应的命令`action+value`（映射成底层的二进制包、AT 指令或其他节点协议格式），根据deviceId找路由到对应的网关，给硬件层发送对应的数据到串口。发送到Java边缘计算网关，网关再通过串口发送给节点。

- 后端发出命令：`0xA1 0x32` 表示 `SETSPEED 50` 
  - `0xA1` 表示“设置速度”
  - `0x32` 表示 50（十进制）
- 解析命令类型和参数，调用对应的控制函数

```cpp
#define MOTOR_PIN 9

void setup() {
  Serial.begin(9600);      // 初始化串口
  pinMode(MOTOR_PIN, OUTPUT);
}

void loop() {
    //检查串口缓存区里有没有收到完整指令（2 字节）
  if (Serial.available() >= 2) {
    byte command = Serial.read();   // 读取1个命令字节
    byte value = Serial.read();     // 读取1个值字节

    if (command == 0xA1) {          // 设置速度命令
      setMotorSpeed(value);
    }
    // 你可以扩展更多命令，例如开灯、关灯、测温等
  }
}

void setMotorSpeed(byte speed) {
  int pwmValue = map(speed, 0, 100, 0, 255);  
    // 将 0~100 映射到 PWM 范围，然后
  analogWrite(MOTOR_PIN, pwmValue);
}

// 回传网关
Serial.write(0xC1);            // 回传温度命令
Serial.write(currentTemp);     // 回传温度值

```

| 指令     | 命令字节 | 参数值             | 对应函数                        |
| -------- | -------- | ------------------ | ------------------------------- |
| 设置速度 | 0xA1     | 0~100              | `setMotorSpeed()`               |
| 打开前灯 | 0xB1     | 1（开），0（关）   | `setFrontLight()`               |
| 读取温度 | 0xC1     | 无                 | `sendTemperature()`（回传数据） |
| 设置方向 | 0xD1     | 0=左, 1=右, 2=直行 | `setDirection()`                |

### 控制硬件的原理

Arduino 是一个单片机开发板，它的 **GPIO（通用输入输出引脚）** 可以输出两种信号：
 ✅ **数字信号（0 或 1，HIGH 或 LOW）**
 ✅ **模拟信号（实际上是 PWM，模拟电压效果）**

比如：

- `digitalWrite(pin, HIGH)` → 给引脚拉高电平（5V）
- `analogWrite(pin, value)` → 输出 PWM 信号，模拟“中间值”（比如 2.5V）

这些信号接到外接硬件（如电机驱动板、继电器、LED、蜂鸣器等），就能驱动实际动作。

例如 Arduino → 电机驱动芯片（如 L298N、TB6612，因为单片机供电能力太弱）→ 小车电机

> 以 L298N 为例：
>
> - **IN1/IN2** → 接 Arduino 的数字引脚，用于控制转向（前/后）
>   - 驱动芯片内部的 H 桥电路根据 IN1/IN2 决定电流方向 → 控制电机前进/后退
> - **ENA（使能）** → 接 Arduino 的 PWM 引脚，用于控制速度（调速）
>   - PWM 占空比越高 → 平均电压越高 → 电机转速越快
>
> Arduino 程序：
>
> - `digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);` → 电机正转
> - `analogWrite(ENA, pwmValue);` → 电机调速
>
> 实际上，setMotorSpeed() 就是给 ENA 发送 PWM 信号。Arduino 发出的 PWM 信号 ≠ 直接供电，而是作为“调速指令”给驱动模块，驱动模块放大后再去推电机。

### analogWrite() 向针脚写入PWM值

向针脚写入一个逻辑值 ([PWM wave](https://assiss.github.io/arduino-zhcn/cn/Tutorial/PWM.html)). 可以用来点亮LED灯,调整其亮度或者驱动一个电机,控制其转速. 调用**analogWrite()**函数后, 对应的针脚会输出一个稳定的,指定占空比的方波.(在这个针脚下一次调用**analogWrite()** ,或者调用**digitalRead()**或者 **digitalWrite()**,针脚的输出会改变为相应的函数执行), PWM信号的频率近乎于490Hz.

在大多数的Arduino板上(MCU为 ATmega168 or ATmega328), 函数起作用的针脚为 3, 5, 6, 9, 10, 和 11. 在 Arduino Mega板上, 起作用的针脚为 2 到 13. 老一点的Arduino 板,MCU是 ATmega8 的,analogWrite()只支持针脚9, 10, and 11. 在执行analogWrite()之前,不需要调用pinMode()函数把针脚设置成输出模式。下面是通过电位器控制led亮灭。

```cpp
int ledPin = 9;      // LED 连接至针脚 9
int analogPin = 3;   // 电位器连接至针脚3
int val = 0;         // 定义一个变量存储读取的值

void setup()
{
  pinMode(ledPin, OUTPUT);   // 把针脚设置成输出
}

void loop()
{
  val = analogRead(analogPin);   // 读取输入
  analogWrite(ledPin, val / 4);  
    // 模拟量值为 0 到 1023, analogWrite 输出量值范围是 0 到 255
}
```

### 软硬件协同

实现可以通过读取命令、或者直接电位器调节的方式设置模块。

- 电位器接入 Arduino/ESP32 的 **ADC（模拟输入）引脚**
- 电机驱动板的调速（ENA 引脚）接到 Arduino 的 **PWM（模拟输出）引脚**

MCU pwm值来源两个：一个是电位器，再有一个就是串口命令。原来串口命令能覆盖电位器，但是电位器发生明显变化也会反过来覆盖。

| 元件             | 接法                                        |
| ---------------- | ------------------------------------------- |
| 电位器（中间脚） | 接 Arduino 的 A0（模拟输入引脚）            |
| 电位器（两边脚） | 接 5V 和 GND                                |
| L298N ENA        | 接 Arduino 的 D9（PWM 输出引脚）            |
| L298N IN1/IN2    | 接 Arduino 的 D7/D8（方向控制）             |
| 电机             | 接 L298N OUT1/OUT2                          |
| 电源             | 给 L298N 单独供电（电机电流大时需分开供电） |

```cpp
const int potPin = A0;      // 电位器
const int pwmPin = 9;       // PWM 输出
const int in1Pin = 7, in2Pin = 8;

int pwmValue = 0;
int lastPotValue = 0;
bool usePotControl = false;

const int potThreshold = 5;  // 电位器变化的触发阈值

void setup() {
  Serial.begin(9600);
  pinMode(pwmPin, OUTPUT);
  pinMode(in1Pin, OUTPUT);
  pinMode(in2Pin, OUTPUT);
  digitalWrite(in1Pin, HIGH);
  digitalWrite(in2Pin, LOW);

  lastPotValue = analogRead(potPin);
}

void loop() {
  // 读取当前电位器值
  int potValue = analogRead(potPin);
  int potMapped = map(potValue, 0, 1023, 0, 255);

  // 检测电位器是否有明显变化
  if (abs(potValue - lastPotValue) > potThreshold) {
    usePotControl = true;  // 硬件优先
  }
  lastPotValue = potValue;

  // 监听串口命令（SET 0~255 / FORCE）
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    if (cmd.startsWith("SET ")) {
      int val = cmd.substring(4).toInt();
      pwmValue = constrain(val, 0, 255);
      usePotControl = false;  // 串口命令优先
    } else if (cmd.startsWith("FORCE")) {
      usePotControl = false;  // 强制回串口模式
    }
  }

  // 根据当前优先权，决定输出
  if (usePotControl) {
    pwmValue = potMapped;
  }

  analogWrite(pwmPin, pwmValue);

  delay(10);
}

```

- ✅ **加遥控/蓝牙/Wi-Fi** → 软件可以覆盖掉硬件值（比如远程命令优先）
-  ✅ **加启动平滑曲线** → 防止突然全速，保护电机
-  ✅ **加速度、刹车控制** → 除了速度，还能控制响应曲线
-  ✅ **加显示屏/LED** → 显示当前速度百分比、方向状态
- ✅ **加入状态反馈** → MCU 回发当前状态到远程端（比如当前速度）
- ✅ **支持更多命令** → 不止调速，还能远程换方向、启停



# 网关（边缘计算+数据汇集）

✅ **Java 边缘计算**（网关）：

- **网关侧硬件设计**
  - 树莓派 / 工控机 + ZigBee 接收模块（用 USB ZigBee 模块，或直接插在串口）
  - Java 程序跑在网关，接收 ZigBee 数据包、解析、做边缘处理、转发给后端

- 串口通信：`jSerialComm` 或 `RXTX` 库
- 边缘计算：对采集数据做简单聚合、报警（如速度异常）
- 本地缓存：可用 `Ehcache`、`Caffeine`
- 通过 HTTP/MQTT（用 Eclipse Paho 客户端库） 上传给后端服务

# 后端（云服务器）

✅ **Java 后端服务**：

- Spring Boot + Spring Web + Spring Data JPA/MyBatis
- 定时任务、报警推送（可整合 WebSocket）
- 存数据库（MySQL / PostgreSQL）：车辆、历史记录、报警、用户等表
- 提供 RESTful 接口供前端查询
- 支持 WebSocket / 推送：实时报警、状态更新

# 前端（跨平台移动端、uniapp）

✅ **前端**：Flutter

- 登录/注册（Flutter 接后端接口）
- 实时车况显示（定时轮询 REST 接口，或者采用 WebSocket）[实时数据 InfluxDB]
- 历史数据展示（速度、油量、温度曲线，用 `fl_chart` 或 `syncfusion_flutter_charts`）[历史数据 MySQL]
- 异常报警（Flutter 本地通知、界面红色高亮、声音提醒）

- 实时推送（如 WebSocket）→ 后端用 `spring-boot-starter-websocket`，Flutter 用 `web_socket_channel` 
- 状态管理：`provider`、`riverpod` 或 `bloc`
- 实时数据：`mqtt_client` 或用定时轮询 HTTP
- 地图：`flutter_map`（开源）或 Google Maps 插件
- 图表：`fl_chart` 或 `syncfusion_flutter_charts`
- 本地通知：`flutter_local_notifications`

| 界面            | 功能                                                        |
| --------------- | ----------------------------------------------------------- |
| 登录界面        | 用户输入手机号 / 密码，调用后端接口登录                     |
| 主界面          | 地图 + 车状态概览（位置、剩余油量、电池、电压、胎压、温度） |
| 远程控制界面    | 按钮（开锁、上锁、闪灯、鸣笛、开空调、关空调）              |
| 历史数据界面    | 折线图（速度、油量、行驶里程），地图轨迹                    |
| 报警 / 消息中心 | 列表（时间、类型、状态），可点击查看详情                    |
| 设置界面        | 车辆信息、账号信息、退出登录                                |





