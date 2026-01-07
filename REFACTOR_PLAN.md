# 项目精简重构计划

## 目标
- 只保留：应用通知监听 → 邮件转发
- 删除：短信、通话、其他所有转发通道、自动任务、Frpc、蓝牙、定位等

## 保留的核心模块

### ✅ 必须保留
1. **通知监听**
   - `service/NotificationService.kt` - 核心通知监听服务
   
2. **邮件发送**
   - `utils/sender/EmailUtils.kt` - 邮件发送实现
   - `utils/mail/EmailSender.kt` - 邮件发送底层
   - `entity/setting/EmailSetting.kt` - 邮件配置实体

3. **消息处理核心**
   - `entity/MsgInfo.kt` - 消息信息实体
   - `workers/SendWorker.kt` - 消息发送Worker（需精简）
   - `utils/SendUtils.kt` - 发送工具（需精简，只保留EMAIL分支）

4. **规则匹配**
   - `database/entity/Rule.kt` - 规则实体（需精简，只保留app相关字段）
   - `database/repository/RuleRepository.kt` - 规则仓库

5. **数据库基础**
   - `database/AppDatabase.kt` - 数据库（需精简表）
   - `database/entity/Sender.kt` - 发送通道实体（需精简，只保留EMAIL）
   - `database/entity/Msg.kt` - 消息实体（需精简字段）
   - `database/entity/Logs.kt` - 日志实体
   - `database/repository/SenderRepository.kt` - 发送通道仓库
   - `database/repository/MsgRepository.kt` - 消息仓库
   - `database/repository/LogsRepository.kt` - 日志仓库

6. **基础框架**
   - `App.kt` - Application（需精简初始化）
   - `core/Core.kt` - 核心工具
   - `utils/SettingUtils.kt` - 设置工具（需精简配置项）

7. **UI核心（需大幅简化）**
   - `activity/MainActivity.kt` - 主界面（需简化Tab）
   - `fragment/LogsFragment.kt` - 日志页面
   - `fragment/RulesFragment.kt` - 规则页面（需简化）
   - `fragment/SendersFragment.kt` - 发送通道页面（需简化，只显示邮件）
   - `fragment/SettingsFragment.kt` - 设置页面（需简化）

### ❌ 需要删除

#### 1. Receiver（广播接收器）
- `receiver/SmsReceiver.kt` - 短信接收器
- `receiver/CallReceiver.kt` - 通话接收器
- `receiver/PhoneStateReceiver.kt` - 电话状态接收器
- `receiver/BootCompletedReceiver.kt` - 开机启动（如果不需要可以删除）
- `receiver/BatteryReceiver.kt` - 电量接收器
- `receiver/BluetoothReceiver.kt` - 蓝牙接收器
- `receiver/NetworkChangeReceiver.kt` - 网络变化接收器
- `receiver/SimStateReceiver.kt` - SIM卡状态接收器
- `receiver/LockScreenReceiver.kt` - 锁屏接收器（如果不需要可以删除）
- `receiver/CactusReceiver.kt` - 保活接收器（如果不需要可以删除）

#### 2. Service（后台服务）
- `service/BluetoothScanService.kt` - 蓝牙扫描服务
- `service/LocationService.kt` - 定位服务
- `service/HttpServerService.kt` - HTTP服务器服务
- `service/ForegroundService.kt` - 前台服务（可能需要保留，但需精简）

#### 3. Sender Utils（发送通道工具，只保留EmailUtils）
- `utils/sender/BarkUtils.kt`
- `utils/sender/DingtalkGroupRobotUtils.kt`
- `utils/sender/DingtalkInnerRobotUtils.kt`
- `utils/sender/FeishuUtils.kt`
- `utils/sender/FeishuAppUtils.kt`
- `utils/sender/GotifyUtils.kt`
- `utils/sender/PushplusUtils.kt`
- `utils/sender/ServerchanUtils.kt`
- `utils/sender/SmsUtils.kt`
- `utils/sender/SocketUtils.kt`
- `utils/sender/TelegramUtils.kt`
- `utils/sender/UrlSchemeUtils.kt`
- `utils/sender/WebhookUtils.kt`
- `utils/sender/WeworkAgentUtils.kt`
- `utils/sender/WeworkRobotUtils.kt`

#### 4. Setting实体（配置实体，只保留EmailSetting）
- `entity/setting/BarkSetting.kt`
- `entity/setting/DingtalkGroupRobotSetting.kt`
- `entity/setting/DingtalkInnerRobotSetting.kt`
- `entity/setting/FeishuSetting.kt`
- `entity/setting/FeishuAppSetting.kt`
- `entity/setting/GotifySetting.kt`
- `entity/setting/PushplusSetting.kt`
- `entity/setting/ServerchanSetting.kt`
- `entity/setting/SmsSetting.kt`
- `entity/setting/SocketSetting.kt`
- `entity/setting/TelegramSetting.kt`
- `entity/setting/UrlSchemeSetting.kt`
- `entity/setting/WebhookSetting.kt`
- `entity/setting/WeworkAgentSetting.kt`
- `entity/setting/WeworkRobotSetting.kt`

#### 5. Workers（后台任务）
- `workers/ActionWorker.kt` - 自动任务执行（如果不需要自动任务）
- `workers/LoadAppListWorker.kt` - 加载应用列表（可能需要保留）

#### 6. 数据库实体
- `database/entity/Task.kt` - 自动任务表（如果不需要）
- `database/entity/Frpc.kt` - Frpc配置表
- `database/repository/TaskRepository.kt` - 任务仓库
- `database/repository/FrpcRepository.kt` - Frpc仓库

#### 7. UI Fragment
- `fragment/TasksFragment.kt` - 任务页面
- `fragment/FrpcFragment.kt` - Frpc页面
- `fragment/ServerFragment.kt` - 服务器页面
- `fragment/ClientFragment.kt` - 客户端页面
- `fragment/AppListFragment.kt` - 应用列表（可能需要保留用于规则配置）

#### 8. 其他工具类
- `utils/task/` - 自动任务相关工具
- `server/` - 服务器相关代码
- `utils/PhoneUtils.kt` - 电话工具（部分功能可能需要保留，如获取应用列表）

## 重构步骤（按顺序执行）

### 阶段1：分析完成 ✅
- [x] 创建分析文档

### 阶段2：配置和入口精简
- [ ] 步骤2.1：清理AndroidManifest.xml（注释不需要的权限和组件）
- [ ] 步骤2.2：精简App.kt（注释不需要的初始化）
- [ ] 步骤2.3：精简ForegroundService（如果保留的话）

### 阶段3：核心功能精简
- [ ] 步骤3.1：精简NotificationService（确保只处理app类型）
- [ ] 步骤3.2：精简SendWorker（移除SMS/CALL分支）
- [ ] 步骤3.3：精简SendUtils（只保留EMAIL分支）
- [ ] 步骤3.4：精简Rule实体（移除SMS/CALL相关）

### 阶段4：数据库精简
- [ ] 步骤4.1：精简数据库实体（移除Task、Frpc等）
- [ ] 步骤4.2：精简Repository（移除不需要的仓库）

### 阶段5：UI精简
- [ ] 步骤5.1：精简MainActivity（只保留必要的Tab）
- [ ] 步骤5.2：精简Fragment（移除不需要的页面）
- [ ] 步骤5.3：UI现代化改造

### 阶段6：彻底清理
- [ ] 步骤6.1：删除无用代码文件
- [ ] 步骤6.2：删除无用资源文件
- [ ] 步骤6.3：清理依赖（移除不需要的库）

## 回退策略

每个步骤都会：
1. 先注释，不直接删除
2. 编译测试通过后再考虑删除
3. 使用Git提交每个步骤，方便回退

## 注意事项

1. **NotificationService** 是核心，不能删除
2. **EmailUtils** 是唯一的发送通道，必须保留
3. **数据库迁移**：如果删除表，需要考虑数据迁移或重建数据库
4. **权限**：通知监听权限必须保留
5. **保活**：如果删除ForegroundService，需要确保NotificationService能稳定运行
