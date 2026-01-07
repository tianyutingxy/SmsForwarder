# SmsForwarder 代码架构文档

## 项目概述

SmsForwarder 是一个 Android 短信和应用通知转发器，主要功能是监控手机的短信和所有应用的推送通知，通过配置的转发渠道（如短信、邮件、Webhook、Telegram Bot 等）转发到其他设备或服务。

## 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                      Android 应用层                        │
├─────────────────────────────────────────────────────────────┤
│  Activity/Fragment (UI层)                                    │
│  - MainActivity: 主界面                                      │
│  - SplashActivity: 启动页                                    │
│  - Fragment: 规则、发送通道、日志、设置等界面               │
├─────────────────────────────────────────────────────────────┤
│  Service (后台服务层)                                        │
│  - ForegroundService: 前台服务，保持应用运行                 │
│  - NotificationService: 监听系统通知                         │
│  - HttpServerService: HTTP 服务器                           │
│  - LocationService: 位置服务                                 │
│  - BluetoothScanService: 蓝牙扫描服务                       │
├─────────────────────────────────────────────────────────────┤
│  Receiver (广播接收层)                                       │
│  - SmsReceiver: 接收短信广播                                │
│  - CallReceiver: 接收通话广播                                │
│  - NetworkChangeReceiver: 网络变化监听                      │
│  - BatteryReceiver: 电量变化监听                            │
│  - 其他系统广播接收器                                        │
├─────────────────────────────────────────────────────────────┤
│  Worker (后台任务层)                                         │
│  - SendWorker: 消息转发任务                                 │
│  - ActionWorker: 自动任务执行                                │
│  - LoadAppListWorker: 加载应用列表                          │
├─────────────────────────────────────────────────────────────┤
│  Core (核心业务层)                                           │
│  - SendUtils: 消息发送工具                                   │
│  - 各种 Sender Utils: 具体转发渠道实现                      │
│  - Rule: 规则匹配逻辑                                        │
│  - ConditionUtils: 条件判断工具                               │
├─────────────────────────────────────────────────────────────┤
│  Database (数据持久层)                                        │
│  - Room Database                                             │
│    - Rule: 转发规则                                          │
│    - Sender: 转发通道                                        │
│    - Msg: 消息记录                                           │
│    - Logs: 转发日志                                          │
│    - Task: 自动任务                                          │
│    - Frpc: 内网穿透配置                                      │
└─────────────────────────────────────────────────────────────┘
```

## 核心模块详解

### 1. 消息监控模块

#### 1.1 短信监控 (SmsReceiver)
- **位置**: `receiver/SmsReceiver.kt`
- **功能**: 
  - 监听系统短信广播 (`SMS_RECEIVED`, `SMS_DELIVER`)
  - 支持 MMS (彩信) 处理
  - 识别 SIM 卡槽信息
  - 支持短信指令功能 (`smsf#`)
- **流程**: 
  ```
  收到短信 → 解析短信内容 → 构建 MsgInfo → 提交 SendWorker
  ```

#### 1.2 应用通知监控 (NotificationService)
- **位置**: `service/NotificationService.kt`
- **功能**:
  - 继承 `NotificationListenerService` 监听系统通知
  - 过滤自身通知和空通知
  - 支持自动消除指定应用通知
  - 支持仅锁屏状态转发
- **流程**:
  ```
  通知到达 → 提取标题和内容 → 构建 MsgInfo → 提交 SendWorker
  ```

#### 1.3 通话记录监控 (CallReceiver)
- **位置**: `receiver/CallReceiver.kt`
- **功能**:
  - 监听来电、去电、未接来电等通话事件
  - 获取通话记录详情
  - 识别 SIM 卡槽信息

### 2. 规则匹配模块

#### 2.1 规则实体 (Rule)
- **位置**: `database/entity/Rule.kt`
- **核心字段**:
  - `type`: 消息类型 (sms/app/call)
  - `filed`: 匹配字段 (transpond_all/phone_num/msg_content/package_name/uid等)
  - `check`: 匹配方式 (is/contain/startwith/endwith/regex等)
  - `value`: 匹配值
  - `simSlot`: SIM 卡槽 (ALL/SIM1/SIM2)
  - `senderList`: 关联的发送通道列表
  - `senderLogic`: 发送逻辑 (ALL/UntilFail/UntilSuccess/Retry)
  - `silentPeriodStart/End`: 免打扰时间段

#### 2.2 规则匹配逻辑
- **匹配流程**:
  ```kotlin
  Rule.checkMsg(MsgInfo) → 
    根据 filed 选择匹配字段 → 
    根据 check 执行匹配逻辑 → 
    返回是否匹配
  ```
- **支持的匹配方式**:
  - `is`: 完全匹配
  - `notis`: 不完全匹配
  - `contain`: 包含
  - `notcontain`: 不包含
  - `startwith`: 开头匹配
  - `endwith`: 结尾匹配
  - `regex`: 正则表达式
- **支持逻辑运算符**: `&&` (与) 和 `||` (或)

### 3. 转发通道模块

#### 3.1 发送通道实体 (Sender)
- **位置**: `database/entity/Sender.kt`
- **核心字段**:
  - `type`: 通道类型 (见下方支持的类型)
  - `name`: 通道名称
  - `jsonSetting`: JSON 格式的配置信息
  - `status`: 启用状态

#### 3.2 支持的转发通道类型

| 类型常量 | 类型值 | 说明 | 实现类 |
|---------|--------|------|--------|
| TYPE_DINGTALK_GROUP_ROBOT | 0 | 钉钉群机器人 | DingtalkGroupRobotUtils |
| TYPE_EMAIL | 1 | 邮件 | EmailUtils |
| TYPE_BARK | 2 | Bark | BarkUtils |
| TYPE_WEBHOOK | 3 | Webhook | WebhookUtils |
| TYPE_WEWORK_ROBOT | 4 | 企业微信机器人 | WeworkRobotUtils |
| TYPE_WEWORK_AGENT | 5 | 企业微信应用 | WeworkAgentUtils |
| TYPE_SERVERCHAN | 6 | Server酱 | ServerchanUtils |
| TYPE_TELEGRAM | 7 | Telegram Bot | TelegramUtils |
| TYPE_SMS | 8 | 短信 | SmsUtils |
| TYPE_FEISHU | 9 | 飞书 | FeishuUtils |
| TYPE_PUSHPLUS | 10 | PushPlus | PushplusUtils |
| TYPE_GOTIFY | 11 | Gotify | GotifyUtils |
| TYPE_DINGTALK_INNER_ROBOT | 12 | 钉钉企业内部机器人 | DingtalkInnerRobotUtils |
| TYPE_FEISHU_APP | 13 | 飞书应用 | FeishuAppUtils |
| TYPE_URL_SCHEME | 14 | URL Scheme | UrlSchemeUtils |
| TYPE_SOCKET | 15 | Socket | SocketUtils |

#### 3.3 发送逻辑 (SenderLogic)
- **ALL**: 执行所有通道，无论成功失败
- **UntilFail**: 遇到失败即终止
- **UntilSuccess**: 遇到成功即终止
- **Retry**: 重试发送

### 4. 消息转发流程

#### 4.1 完整转发流程

```
1. 消息到达 (SmsReceiver/NotificationService/CallReceiver)
   ↓
2. 构建 MsgInfo 对象
   ↓
3. 提交 SendWorker (WorkManager)
   ↓
4. SendWorker.doWork():
   - 自动任务处理 (autoTaskProcess)
   - 免打扰时间段检查
   - 重复消息过滤
   - 查询匹配的规则列表
   - 规则匹配检查
   - 保存消息到数据库
   - 为每个匹配的规则创建日志
   - 调用 SendUtils.sendMsgSender()
   ↓
5. SendUtils.sendMsgSender():
   - 根据 Sender 类型选择对应的 Utils
   - 调用具体通道的 sendMsg() 方法
   - 更新转发日志
   - 执行发送逻辑 (senderLogic)
   ↓
6. 具体通道实现 (如 TelegramUtils.sendMsg()):
   - 构建请求参数
   - 发送 HTTP 请求
   - 处理响应
   - 更新日志状态
```

#### 4.2 消息模板替换

支持在消息内容中使用模板变量，如：
- `{receive_time}`: 接收时间
- `{from}`: 发送方
- `{sms}`: 短信内容
- `{card_slot}`: 卡槽信息
- `{app_name}`: 应用名称
- `{title}`: 通知标题
- 等等...

### 5. 数据持久层

#### 5.1 数据库表结构

**Rule (规则表)**
- `id`: 主键
- `type`: 消息类型
- `filed`: 匹配字段
- `check`: 匹配方式
- `value`: 匹配值
- `sender_id`: 关联发送通道ID (已废弃，使用 sender_list)
- `sender_list`: 发送通道列表 (JSON)
- `sender_logic`: 发送逻辑
- `sim_slot`: SIM 卡槽
- `status`: 状态
- `silent_period_start/end`: 免打扰时间段

**Sender (发送通道表)**
- `id`: 主键
- `type`: 通道类型
- `name`: 通道名称
- `json_setting`: 配置信息 (JSON)
- `status`: 状态

**Msg (消息表)**
- `id`: 主键
- `type`: 消息类型
- `from`: 发送方
- `content`: 内容
- `sim_slot`: SIM 卡槽
- `sim_info`: SIM 信息
- `sub_id`: 订阅ID
- `call_type`: 通话类型

**Logs (日志表)**
- `id`: 主键
- `type`: 消息类型
- `msg_id`: 关联消息ID
- `rule_id`: 关联规则ID
- `sender_id`: 关联发送通道ID
- `forward_status`: 转发状态 (0=失败, 1=处理中, 2=成功)
- `forward_response`: 转发响应

**Task (任务表)**
- `id`: 主键
- `type`: 任务类型
- `conditions`: 触发条件 (JSON)
- `actions`: 执行动作 (JSON)
- `status`: 状态

**Frpc (内网穿透表)**
- `id`: 主键
- `uid`: 唯一标识
- `name`: 名称
- `config`: 配置内容
- `autorun`: 是否自启动

### 6. 后台服务

#### 6.1 ForegroundService
- **功能**: 
  - 保持应用在后台运行
  - 管理通知监听服务
  - 启动定时任务 (Cron)
  - 管理 Frpc 内网穿透
  - 处理警报 (振动、闪光灯、音乐)
- **保活机制**: 使用 Cactus 库实现多进程保活

#### 6.2 NotificationService
- **功能**: 监听系统通知
- **权限**: `BIND_NOTIFICATION_LISTENER_SERVICE`

### 7. 自动任务系统

#### 7.1 任务触发条件
- **TASK_CONDITION_SMS**: 短信触发
- **TASK_CONDITION_APP**: 应用通知触发
- **TASK_CONDITION_CALL**: 通话触发
- **TASK_CONDITION_CRON**: 定时任务触发

#### 7.2 任务执行动作
- **TASK_ACTION_SENDSMS**: 发送短信
- **TASK_ACTION_NOTIFICATION**: 发送通知
- **TASK_ACTION_ALARM**: 警报 (振动/闪光/音乐)
- **TASK_ACTION_CLEANER**: 清理操作
- **TASK_ACTION_FRPC**: Frpc 操作
- **TASK_ACTION_HTTPSERVER**: HTTP 服务器操作
- **TASK_ACTION_SETTINGS**: 设置操作
- **TASK_ACTION_RESEND**: 重发消息
- **TASK_ACTION_RULE**: 规则操作

### 8. 工具类

#### 8.1 核心工具类
- **SendUtils**: 消息发送核心逻辑
- **PhoneUtils**: 电话相关工具 (发送短信、获取SIM信息等)
- **SettingUtils**: 设置管理
- **HistoryUtils**: 历史记录管理
- **Log**: 日志工具
- **DataProvider**: 数据提供者 (时间判断等)

#### 8.2 各通道工具类
位于 `utils/sender/` 目录下，每个通道都有对应的 Utils 类实现具体的发送逻辑。

## 关键设计模式

1. **Repository 模式**: 数据库操作通过 Repository 层封装
2. **Worker 模式**: 使用 WorkManager 处理后台任务
3. **策略模式**: 不同转发通道使用不同的策略实现
4. **观察者模式**: 使用 LiveEventBus 进行组件间通信

## 数据流向

```
系统事件 (短信/通知/通话)
    ↓
BroadcastReceiver/NotificationListenerService
    ↓
构建 MsgInfo
    ↓
WorkManager (SendWorker)
    ↓
规则匹配 (Rule.checkMsg)
    ↓
查询匹配的规则列表
    ↓
为每个规则创建日志
    ↓
SendUtils.sendMsgSender
    ↓
根据 Sender.type 选择对应的 Utils
    ↓
执行具体通道的发送逻辑
    ↓
更新日志状态
    ↓
数据库持久化
```

## 配置文件

- **AndroidManifest.xml**: 应用配置、权限、组件注册
- **build.gradle**: 依赖管理
- **proguard-rules.pro**: 代码混淆规则

## 主要依赖

- **Room**: 数据库 ORM
- **WorkManager**: 后台任务管理
- **LiveEventBus**: 事件总线
- **Cactus**: 应用保活
- **XUI**: UI 组件库
- **XHttp**: HTTP 请求库
- **Gson**: JSON 解析

## 扩展点

1. **新增转发通道**: 
   - 在 `Constants.kt` 中添加新的 `TYPE_*` 常量
   - 创建对应的 `*Utils.kt` 实现发送逻辑
   - 在 `SendUtils.kt` 的 `when` 语句中添加新分支
   - 在 `Sender.kt` 的 `imageId` 中添加图标映射

2. **新增规则匹配字段**:
   - 在 `Constants.kt` 中添加新的 `FILED_*` 常量
   - 在 `Rule.kt` 的 `checkMsg()` 方法中添加匹配逻辑

3. **新增自动任务动作**:
   - 在 `Constants.kt` 中添加新的 `TASK_ACTION_*` 常量
   - 在 `ActionWorker.kt` 的 `when` 语句中添加执行逻辑

## 注意事项

1. **权限管理**: 应用需要大量敏感权限，需要在运行时动态申请
2. **保活机制**: 使用多种保活策略确保应用在后台持续运行
3. **电池优化**: 需要引导用户将应用加入电池优化白名单
4. **通知权限**: Android 13+ 需要通知权限才能监听通知
5. **纯客户端模式**: 支持纯客户端模式，不执行转发逻辑
