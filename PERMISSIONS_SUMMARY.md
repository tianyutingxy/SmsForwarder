# 权限保留清单

根据需求：**只保留应用通知监听 → 邮件转发功能，用户可选择监听哪些app**

## ✅ 已保留的权限（共 16 个）

### 1. 应用列表相关（用户选择监听哪些app）
- ✅ `com.android.permission.GET_INSTALLED_APPS` - 获取已安装应用列表
- ✅ `android.permission.QUERY_ALL_PACKAGES` - 查询所有包（Android 11+需要）
- ✅ `android.permission.INSTALL_PACKAGES` - 安装包权限（用户要求保留）

### 2. 通知监听相关（核心功能）
- ✅ `android.permission.BIND_NOTIFICATION_LISTENER_SERVICE` - 绑定通知监听服务（必需）
- ✅ `android.permission.POST_NOTIFICATIONS` - 发送通知（Android 13+需要）
- ✅ `android.permission.CANCEL_NOTIFICATIONS` - 取消通知
- ✅ `android.permission.ACTION_NOTIFICATION_LISTENER_SETTINGS` - 打开通知监听设置

### 3. 网络相关（发邮件需要）
- ✅ `android.permission.INTERNET` - 网络访问（必需）
- ✅ `android.permission.ACCESS_NETWORK_STATE` - 检查网络状态

### 4. 前台服务相关（保活）
- ✅ `android.permission.FOREGROUND_SERVICE` - 前台服务（Android 9+需要）
- ✅ `android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` - 忽略电池优化

### 5. 存储相关（查看和导出日志）
- ✅ `android.permission.READ_EXTERNAL_STORAGE` - 读取外部存储
- ✅ `android.permission.WRITE_EXTERNAL_STORAGE` - 写入外部存储
- ✅ `android.permission.MANAGE_EXTERNAL_STORAGE` - 管理外部存储（Android 11+）

### 6. 系统相关
- ✅ `android.permission.RECEIVE_BOOT_COMPLETED` - 开机启动（用于启动服务）
- ✅ `android.permission.SCHEDULE_EXACT_ALARM` - 定时任务（如果需要）

## ❌ 已移除的权限

### 短信相关
- ❌ `android.permission.RECEIVE_SMS`
- ❌ `android.permission.RECEIVE_MMS`
- ❌ `android.permission.RECEIVE_WAP_PUSH`
- ❌ `android.permission.READ_SMS`
- ❌ `android.permission.SEND_SMS`

### 通话相关
- ❌ `android.permission.CALL_PHONE`
- ❌ `android.permission.READ_CALL_LOG`
- ❌ `android.permission.READ_PHONE_STATE`
- ❌ `android.permission.READ_PHONE_NUMBERS`
- ❌ `android.permission.READ_PRIVILEGED_PHONE_STATE`

### 联系人相关
- ❌ `android.permission.READ_CONTACTS`
- ❌ `android.permission.WRITE_CONTACTS`
- ❌ `android.permission.GET_ACCOUNTS`

### 蓝牙相关
- ❌ `android.permission.BLUETOOTH`
- ❌ `android.permission.BLUETOOTH_ADMIN`
- ❌ `android.permission.BLUETOOTH_CONNECT`
- ❌ `android.permission.BLUETOOTH_SCAN`
- ❌ `android.permission.BLUETOOTH_ADVERTISE`

### 定位相关
- ❌ `android.permission.ACCESS_FINE_LOCATION`
- ❌ `android.permission.ACCESS_COARSE_LOCATION`
- ❌ `android.permission.ACCESS_BACKGROUND_LOCATION`

### 存储相关（已保留，用于查看和导出日志）
- ✅ `android.permission.READ_EXTERNAL_STORAGE` - 读取外部存储
- ✅ `android.permission.WRITE_EXTERNAL_STORAGE` - 写入外部存储
- ✅ `android.permission.MANAGE_EXTERNAL_STORAGE` - 管理外部存储（Android 11+）

### 其他
- ❌ `android.permission.VIBRATE` - 振动
- ❌ `android.permission.CAMERA` - 相机
- ❌ `android.permission.BATTERY_STATS` - 电池统计
- ❌ `android.permission.READ_LOGS` - 读取日志
- ❌ `android.permission.REBOOT` - 重启
- ❌ `android.permission.KILL_BACKGROUND_PROCESSES` - 杀死后台进程
- ❌ `android.permission.WRITE_SETTINGS` - 写入设置

## 📝 说明

1. **INSTALL_PACKAGES 权限**：虽然通常用于安装应用，但用户明确要求保留，可能是为了在某些场景下读取应用信息。

2. **存储权限**：已保留，用于查看和导出日志功能。

3. **所有权限改动都是注释形式**，可以随时恢复。

## 🔄 回退方法

如果需要恢复某个权限，只需在 AndroidManifest.xml 中取消对应行的注释即可。
