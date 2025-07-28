---
name: iot-thing-model
title: 物模型
date: 2025-07-27
---

# 大模型控制智能家居

## Capability

**BaseCapability（基础能力）**
 定义所有设备都具备的通用能力，比如开关（Power）

**CategoryCapability（品类能力）**
 针对设备大类（如灯、空调、窗帘）定义的能力，比如灯具的亮度调节（Brightness）

**ProductCapability（产品特有能力）**
 针对具体型号或功能更丰富的产品，添加额外能力，如色温调节（ColorTemperature）、色彩模式（ColorMode）

```json
{
  "deviceType": "colorLight",
  "capabilities": [
    {
      "name": "power",
      "type": "boolean"
    },
    {
      "name": "brightness",
      "type": "integer",
      "min": 0,
      "max": 100,
      "unit": "%"
    },
    {
      "name": "colorTemperature",
      "type": "integer",
      "min": 2000,
      "max": 6500,
      "unit": "K"
    }
  ]
}

```

## 系统架构总览

```markdown
┌────────────────────────────────────────────────┐
│                    客户端                      │  （App/Web/小程序/语音设备）
│  ┌─────────┐   ┌──────────┐   ┌────────────┐  │
│  │ 语音/   │──▶│ ASR/STT  │──▶│  NLU      │  │
│  │ 文本输入│   └──────────┘   └────────────┘  │
│  └─────────┘                             │   │
└─────────────────────────────────────────▼────┘
              ▲                             │
              │     ┌───────────────┐       │
              └────▶│对话管理层（DM）│◀──────┘
                    └───────────────┘
                             │
       ┌─────────────────────┼─────────────────────┐
       │                     │                     │
┌────────────┐       ┌──────────────┐       ┌───────────┐
│ 意图/槽位  │       │ 对话状态 &  │       │ 设备控制  │
│ 管理 (NLU) │──────▶│ 会话存储    │──────▶│  API 层  │
└────────────┘       └──────────────┘       └───────────┘
                             │
                           日志／审计／监控
```

### 核心模块

1. **ASR/STT**（可选语音）。
2. **NLU**：意图识别 + 槽位抽取。
3. **DM（对话管理）**：根据当前意图与上下文状态，决定下一步动作／回复。
4. **SessionStorage**：存储会话上下文、用户状态、历史操作。
5. **Function Calling / Device API**：将意图映射到具体设备控制接口（MQTT/CoAP/REST/WebSocket）。
6. **安全与鉴权**：OAuth2.0/JWT/设备访问令牌。
7. **日志与监控**：所有指令及回复入库、监控平台报警。

## 关键技术细节

### 意图识别 & 槽位抽取（NLU）

- **模型**：可用开源框架（Rasa、Snips）或自研轻量级分类+CRF/Seq2Seq。
- **核心意图**：开关设备（开灯/关灯）、调节参数（调高温度）、查询状态（室温多少）、情景模式（回家模式）。
- **槽位**：`device`（灯、空调）、`location`（客厅、卧室）、`value`（25℃、50%）。

```java
public class IntentResult {
    private String intent;
    private Map<String, String> slots;
    // getters/setters
}
```

### 对话管理（DM）

- **状态机**：定义对话流转逻辑，例如：
  - 如果缺少槽位，DM 发起补全询问；
  - 如果意图确认，直接触发 Function Calling；
  - 多轮确认、歧义消解等。
- **策略**：基于规则的 FSM + 简单策略；亦可结合强化学习/蒙特卡洛树搜索进行优化。

```java
public class DialogManager {
    public DialogueAct handle(IntentResult nlu, Session session) {
        if (!session.hasSlot("device")) {
            return ask("请问您想控制哪个设备？");
        }
        if (!session.hasSlot("location")) {
            return ask("哪一个房间的" + session.getSlot("device") + "？");
        }
        // 槽位齐全，执行控制
        return executeControl(session);
    }
}
```

### 会话存储（SessionStorage）

| 方案      | 适用场景                      | 优势                         | 劣势                 |
| --------- | ----------------------------- | ---------------------------- | -------------------- |
| In‑Memory | 单机、调试、POC               | 极低延迟、无外部依赖         | 不持久，无法分布式   |
| Redis     | 分布式、高并发、需要 TTL 管理 | 水平扩展、持久化配置、TTL    | 网络延迟、运维成本   |
| RDBMS     | 事务要求、历史数据分析        | 结构化查询、事务、成熟生态   | 性能瓶颈、扩展性较弱 |
| VectorDB  | 语义检索、海量历史对话        | 基于相似度召回，检索相关性高 | 架构复杂、向量化开销 |

- 存储：index:userId 用于按照创建时间戳去
- **接口同前面设计**：支持多用户、多会话、持久化与过期策略。
- sessionId: userID_timestamp_randomBits

**短时语音交互场景（5–15秒）**

- 如果你的系统主要是语音唤醒＋单轮指令（如“小爱同学，开灯”），那么用户发完一条指令后、返回结果之前就可认为一次会话结束。
- 建议：在 **5–10秒** 内无新语音输入，即认为本次交互结束，清理短期会话上下文。

**短对话补充场景（1–2分钟）**

- 当用户需要多轮补全槽位（例如“把灯调到多少亮度？”→“80”），通常流程不会超过 1 分钟。
- 建议：若 **1–2 分钟** 内仍无用户回复，就主动结束当前会话，下次发起新请求时开启新的 session。

**长会话场景（10–30分钟）**

- 在 App 或 Web 上，用户可能边看界面边对话，甚至切到后台再回来；此时希望保持会话连续。
- 建议：在 **20–30 分钟** 无任何交互，就自动关闭，会话存储 TTL 设置为 30 分钟；超过则重新生成新的 sessionId。

**可配置的分层过期**

- **短期上下文**（slot filling、多轮确认）使用较短的过期（如 2 分钟），过期后清空当前多轮状态但保留历史记录。
- **长期历史**（行为日志、偏好设置、场景记忆）使用更长的 TTL（如数小时到数天），用于个性化推荐或日志审计。

```java
public class SessionExpiryPolicy {
    // 语音单轮指令
    public static final Duration SHORT_TURN_TIMEOUT = Duration.ofSeconds(10);
    // 多轮补充
    public static final Duration MULTI_TURN_TIMEOUT = Duration.ofMinutes(2);
    // 长会话保留
    public static final Duration LONG_SESSION_TTL    = Duration.ofMinutes(30);
    /**
 * 根据最后一次活动时间判断是否需要开启新会话
 */
	public boolean isExpired(Session session) {
    	Duration idle = Duration.between(session.getLastAccess(), Instant.now());
        if (session.isInSlotFilling()) {
            return idle.compareTo(MULTI_TURN_TIMEOUT) > 0;
        } else if (session.isVoiceSingleTurn()) {
            return idle.compareTo(SHORT_TURN_TIMEOUT) > 0;
        } else {
            return idle.compareTo(LONG_SESSION_TTL) > 0;
        }
    }
}

```
- slotFilling 标志：在对话管理中，若当前 Session 处于“等待槽位补全”状态，就用多轮补充超时；否则按单轮或长会话处理。
- 最后访问时间：每次用户输入或系统输出都要更新 session.lastAccess。
- 过期处理：一旦判定过期，可删除或标记 closed，下次请求自动新建 sessionId 并关联新上下文。

**扩展**：为每个对话保存上下文对象 `Session { userId, sessionId, slots, lastAccess, history }`。

```java
public class Session {
    private String userId;
    private String sessionId;
    private Map<String,String> slots = new HashMap<>();
    private List<String> history = new ArrayList<>();
    // ...
}
```

### 4. Function Calling 与设备控制

- **适配层**：将意图和槽位映射到具体 API 调用。
- **通信协议**：MQTT（轻量、实时）、CoAP、或 HTTP RESTful。

```java
public class DeviceController {
    private MqttClient mqttClient;

    public void turnOn(String device, String location) {
        String topic = String.format("home/%s/%s/command", userId, location);
        String payload = new JSONObject()
            .put("device", device)
            .put("action", "on").toString();
        mqttClient.publish(topic, payload.getBytes(), 1, false);
    }
}
```

### 安全与鉴权

- **用户认证**：前端登录获取 JWT；
- **设备授权**：对每条控制指令附带 Token；
- **校验**：网关层或消息中间件拦截，防止非法调用。

### 日志、审计与监控

- 保存所有对话流水、设备反馈、异常日志。
- 结合 ELK / Prometheus / Grafana 实时监控设备连通性、对话成功率、延迟。

## 示例流程

1. **用户**：“帮我把客厅的灯打开。”
2. **ASR** → 文本
3. **NLU**：`intent=TurnOn`, `slots={device:灯, location:客厅}`
4. **DM**：槽位齐全 → 调用 `DeviceController.turnOn("灯","客厅")`
5. **DeviceController** 发布 MQTT 消息
6. **设备** 执行后上报状态
7. **DM** 收到反馈，回复 “客厅的灯已打开”。

## 可扩展方向

1. **多模态交互**：结合摄像头、手势、遥控器等。
2. **场景编排**：支持用户自定义 “回家模式” 一键执行多条指令。
3. **语义记忆**：引入向量检索，自动推荐常用指令或习惯场景。
4. **离线容错**：网络不通时缓存指令，恢复网络后再下发。
5. **多设备协同**：跨品牌协议适配（Zigbee、Z-Wave、Matter）。