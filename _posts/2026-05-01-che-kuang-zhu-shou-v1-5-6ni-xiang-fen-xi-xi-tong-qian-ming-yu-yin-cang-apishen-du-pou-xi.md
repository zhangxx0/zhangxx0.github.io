---
layout:     post
title:      "车况助手V1.5.6逆向分析：系统签名与隐藏API深度剖析"
date:       2026-05-01
author:     "Zhangxx"
header-img: "img/post-headers/2026-05-01-2026-05-01-che-kuang-zhu-shou-v1-5-6ni-xiang-fen-xi-xi-tong-qian-ming-yu-yin-cang-apishen-du-pou-xi.jpg"
catalog:    true
description: "本文对比亚迪车况助手V1.5.6进行逆向分析，揭示了其使用BYD内部测试CA签发的系统级签名证书，以及调用48条BYD车机专属权限的能力。分析发现该应用使用了未公开的扩展API，如ADAS、碰撞检测和大数据接口，为开发者提供了深入了解车机应用权限模型的参考。"
tags:
    - 智能座舱
    - 车机应用开发
    - API集成
    - 技术储备
---

> 不识庐山真面目，只缘身在此山中。  


# 车况助手 V1.5.6 逆向分析报告

> 分析日期: 2026-04-06
> APK 文件: 车况助手 V1.5.6.apk

---

## 一、基本信息

| 项目 | 值 |
|------|-----|
| 包名 | `com.huawei.carstatushelper` |
| 版本 | V1.5.6 (versionCode=53) |
| minSdk | 25 (Android 7.1) |
| targetSdk | 33 (Android 13) |
| 入口 Activity | `SplashActivity` → `MainActivity` |
| 语言 | Kotlin (经 R8/ProGuard 混淆) |

### 签名证书

| 项目 | 值 |
|------|-----|
| 签发者 (Issuer) | `BYDAUTO-Co.Ltd-H-Test-DCA`, C=CN |
| 持有者 (Subject) | `987654326@byd.com`, OU=`ITCENTER_H_TEST` |
| 组织 | 比亚迪汽车工业有限公司 |
| 有效期 | 2018-01-22 ~ 2038-01-17 |
| 算法 | SHA1withRSA 2048位 |
| SHA1 指纹 | `78:50:00:71:05:60:BF:0A:85:FF:48:DA:D0:8E:9C:98:E2:3C:0C:A1` |
| CRL 分发点 | `http://10.44.40.47:8010/bydca_test/...` (BYD 内网地址) |

**结论**: 这是 BYD 内部测试 CA 签发的**系统级签名证书**，非第三方开发者可获取。系统签名使该应用拥有 `signature` 级别权限，可直接调用所有 `BYDAUTO_*` 权限保护的接口，且不受 hidden API 限制。

---

## 二、权限声明全景 (AndroidManifest.xml)

### 2.1 BYD 车机专属权限 — 共 48 条

按模块分组:

| 模块 | COMMON | GET | SET | 说明 |
|------|--------|-----|-----|------|
| AC (空调) | ✅ | ✅ | ✅ | 空调控制 |
| BODYWORK (车身) | ✅ | ✅ | ✅ | 车门/车窗/VIN/电源档位 |
| CHARGING (充电) | ✅ | ✅ | ✅ | 充电状态/充电量 |
| DOOR_LOCK (门锁) | ✅ | ✅ | - | 只读门锁状态 |
| INSTRUMENT (仪表) | ✅ | ✅ | ✅ | 故障/蜂鸣器/保养 |
| PANORAMA (全景) | ✅ | ✅ | - | 全景影像 |
| SETTING (设置) | ✅ | ✅ | ✅ | 车辆配置 |
| POWER (电源) | ✅ | ✅ | ✅ | 电源管理 (**非标准20类**) |
| REAR_VIEW_MIRROR (后视镜) | ✅ | ✅ | ✅ | 后视镜控制 (**非标准20类**) |
| SPEED (车速) | - | ✅ | - | 车速/油门/制动 |
| STATISTIC (统计) | - | ✅ | - | 里程/电量/油耗 |
| ENERGY (能量) | - | ✅ | - | 工作模式 |
| ENGINE (发动机) | - | ✅ | - | 转速/排量 |
| GEARBOX (变速箱) | - | ✅ | - | 档位/驻车 |
| LIGHT (车灯) | - | ✅ | - | 灯光状态 |
| LOCATION (定位) | - | ✅ | - | BYD 定位 |
| MULTIMEDIA (媒体) | - | ✅ | - | 播放信息 |
| PM2P5 (空气) | - | ✅ | - | PM2.5 |
| RADAR (雷达) | - | ✅ | - | 探头距离 |
| SAFETY_BELT (安全带) | - | ✅ | - | 安全带状态 |
| SENSOR (传感器) | - | ✅ | - | 光照强度 |
| TIME (时间) | - | ✅ | - | 车机时间 |
| TYRE (轮胎) | - | ✅ | - | 胎压/温度 |
| ADAS (辅助驾驶) | - | ✅ | - | ADAS 数据 (**非标准20类**) |
| BIGDATA (大数据) | - | ✅ | - | 大数据接口 (**非标准20类**) |
| COLLISION (碰撞) | - | ✅ | - | 碰撞检测 (**非标准20类**) |
| AUDIO (音频) | - | ✅ | - | 音频状态 |

> **关键发现**: 该应用声明了 `POWER`、`REAR_VIEW_MIRROR`、`ADAS`、`BIGDATA`、`COLLISION` 等权限，这些不在 BYD 官方公开文档的标准 20 类 API 中，说明该应用使用了**未公开的扩展 API**。

### 2.2 Android 标准权限

| 权限 | 用途 |
|------|------|
| `INTERNET` | 网络访问 |
| `SYSTEM_ALERT_WINDOW` | 悬浮窗 (浮窗显示车况) |
| `READ_PHONE_STATE` | 读取设备信息 |
| `RECEIVE_BOOT_COMPLETED` | 开机自启 |
| `FOREGROUND_SERVICE` | 前台服务 |
| `READ_EXTERNAL_STORAGE` | 存储读取 |
| `WRITE_EXTERNAL_STORAGE` | 存储写入 |

---

## 三、应用架构分析

### 3.1 组件结构

```
com.huawei.carstatushelper/
├── SplashActivity              # 启动页 (权限申请入口)
├── MainActivity                # 主界面 (车况仪表盘)
├── activity/
│   └── FuelConsumptionStatisticsActivity  # 油耗统计
├── service/
│   └── FloatingService         # 悬浮窗服务
├── receiver/
│   ├── BootCompleteReceiver    # 开机广播接收器
│   └── BootCompleteService     # 开机自启后台服务
├── test/
│   ├── AdasTestActivity        # ADAS 测试页
│   └── ReflectTestActivity     # 反射调用测试页
├── floating/                   # 悬浮窗 UI 组件
├── view/                       # 自定义 View
│   ├── EngineSpeedView         # 发动机转速表
│   ├── MotorSpeedView          # 电机转速表
│   ├── RearMotorSpeedView      # 后电机转速表
│   ├── CarSpeedView            # 车速表
│   └── EnginePowerView         # 发动机功率表
└── util/
    └── BydManifest             # 权限常量字典
```

### 3.2 启动流程

```
用户点击图标
    │
    ▼
SplashActivity
    ├── 检查权限 (静态字段 x = String[] 权限数组)
    ├── 动态申请 COMMON 类权限
    ├── onRequestPermissionsResult 处理授权结果
    ├── SharedPreferences 记录授权状态
    └── 授权完成 → 跳转 MainActivity
            │
            ▼
      MainActivity
            ├── 初始化 11 个 BYDAuto Device 实例
            ├── 注册大量 Listener 监听数据变化
            └── 显示实时车况仪表盘

同时 (开机自启):
    BOOT_COMPLETED 广播
        │
        ▼
    BootCompleteReceiver
        │
        ▼
    BootCompleteService (后台持续监控)
        ├── 初始化 7 个 BYDAuto Device 实例
        ├── 注册雷达/全景/档位等监听器
        ├── TTS 语音播报
        └── 配合 FloatingService 悬浮窗显示
```

---

## 四、权限获取机制详解

### 4.1 SplashActivity — 动态权限申请入口

`SplashActivity` 是启动页，负责权限申请:

- 静态字段 `x` 类型为 `String[]` — **需要动态申请的权限数组**
- 实现了 `View.OnClickListener` (用户点击同意授权)
- 有 `onRequestPermissionsResult` 回调处理授权结果
- 使用 `SharedPreferences` 记录授权状态
- 集成了 `TextToSpeech` (TTS 语音播报)

**流程**: 启动时在 `onCreate` 中检查权限 → 弹出授权请求 → 用户同意后跳转 `MainActivity`

### 4.2 BydManifest$permission — 权限常量字典

`com.huawei.carstatushelper.util.BydManifest` 是自定义权限常量类:

- `BydManifest$permission` — 所有权限字符串常量 (标准 Android + BYD 车机权限共 200+ 条)
- `BydManifest$permission_group` — 权限分组

相当于 BYD 版的 `android.Manifest.permission`，是该应用自己维护的**完整权限字典**，覆盖了 BYD 车机系统所有已知权限。

### 4.3 MainActivity — 设备实例获取

`MainActivity` 持有大量 BYDAuto 设备实例字段:

| 混淆字段名 | 类型 | 说明 |
|-----------|------|------|
| `t` | `BYDAutoEngineDevice` | 发动机 |
| `u` | `BYDAutoSpeedDevice` | 车速 |
| `v` | `BYDAutoStatisticDevice` | 统计 |
| `w` | `BYDAutoEnergyDevice` | 能量 |
| `x` | `BYDAutoGearboxDevice` | 变速箱 |
| `y` | `BYDAutoAcDevice` | 空调 |
| `z` | `BYDAutoChargingDevice` | 充电 |
| `A` | `BYDAutoTyreDevice` | 轮胎 |
| `B` | `BYDAutoBodyworkDevice` | 车身 |
| `C` | `BYDAutoSettingDevice` | 设置 |
| `D` | `BYDAutoInstrumentDevice` | 仪表 |

均通过 `BYDAuto{Module}Device.getInstance(context)` 获取单例，然后注册 Listener 监听数据变化。

### 4.4 MainActivity 内部类 — Listener 回调

| 内部类 | 父类 | 监听内容 |
|--------|------|---------|
| `MainActivity$9` | `AbsBYDAutoStatisticListener` | 里程/电量/油耗变化 |
| `MainActivity$b` (混淆) | `AbsBYDAutoSpeedListener` | 车速/油门/制动变化 |
| 其他 `$c` ~ `$k` | 各模块 Listener | 发动机/能量/变速箱等 |

`MainActivity$9` 的 `onElecDrivingRangeChanged` 回调中包含 **5 个反射 try-catch 块**，说明在统计数据回调中也通过反射调用了未公开 API。

### 4.5 BootCompleteService — 开机自启后台服务

开机广播 → 启动 `BootCompleteService`，该服务持有:

| 混淆字段名 | 类型 | 说明 |
|-----------|------|------|
| `a` | `BYDAutoRadarDevice` | 雷达 |
| `b` | `BYDAutoPanoramaDevice` | 全景 |
| `c` | `BYDAutoGearboxDevice` | 变速箱 |
| `g` | `BYDAutoBodyworkDevice` | 车身 |
| `h` | `BYDAutoChargingDevice` | 充电 |
| `i` | `BYDAutoEnergyDevice` | 能量 |
| `j` | `BYDAutoStatisticDevice` | 统计 |

开机后立即初始化设备实例并注册监听器，实现**后台持续监控车况** (雷达障碍物距离、全景影像状态、档位变化等)，配合 TTS 语音播报。

内部类:
- `BootCompleteService$4` (继承 `AbsBYDAutoRadarListener`) — 监听雷达障碍物距离，包含反射调用
- `BootCompleteService$a` (继承 `AbsBYDAutoPanoramaListener`) — 监听全景影像状态

### 4.6 FloatingService — 悬浮窗服务

利用 `SYSTEM_ALERT_WINDOW` 权限创建悬浮窗，在其他应用上方显示实时车况数据:

- `onCreate` 方法有 **1143 条指令**，是整个应用最大的方法之一
- 包含 JSON 解析 (`JSONException` catch 块)，可能从配置文件读取悬浮窗布局
- 实现了 `View.OnClickListener`，支持点击交互

### 4.7 ReflectTestActivity — 反射调用机制 (核心发现)

这是最值得关注的部分。`ReflectTestActivity` 大量使用 Java 反射:

- `onClick` 方法有 **890 条指令**，包含 **7 个 try-catch 块**，全部捕获反射异常:
  - `ClassNotFoundException`
  - `NoSuchMethodException`
  - `IllegalAccessException`
  - `InvocationTargetException`
  - `InstantiationException`

- 持有设备实例:
  - `BYDAutoSettingDevice`
  - `BYDAutoADASDevice`
  - `BYDAutoBodyworkDevice`

**结论**: 该应用通过反射调用了 BYD 车机系统中**未公开的 API 方法**，绕过了正常的 SDK 接口限制。同样的反射模式也出现在:
- `MainActivity$9` — StatisticListener 的 `onElecDrivingRangeChanged` 回调 (5 个反射 try-catch)
- `BootCompleteService$4` — RadarListener 的 `onRadarObstacleDistanceChanged` 回调 (4 个反射 try-catch)

---

## 五、使用的 BYDAuto Device 完整清单

从 DEX 中提取到的所有 BYDAuto 设备类引用:

### 5.1 标准 20 类 API (官方公开)

| 设备类 | 使用位置 |
|--------|---------|
| `BYDAutoAcDevice` | MainActivity |
| `BYDAutoBodyworkDevice` | MainActivity, BootCompleteService, ReflectTestActivity |
| `BYDAutoChargingDevice` | MainActivity, BootCompleteService |
| `BYDAutoDoorLockDevice` | (权限声明，具体使用位置在混淆代码中) |
| `BYDAutoEngineDevice` | MainActivity |
| `BYDAutoEnergyDevice` | MainActivity, BootCompleteService |
| `BYDAutoGearboxDevice` | MainActivity, BootCompleteService |
| `BYDAutoInstrumentDevice` | MainActivity, ReflectTestActivity |
| `BYDAutoPanoramaDevice` | BootCompleteService |
| `BYDAutoRadarDevice` | BootCompleteService |
| `BYDAutoSensorDevice` | (权限声明) |
| `BYDAutoSettingDevice` | MainActivity, ReflectTestActivity |
| `BYDAutoSpeedDevice` | MainActivity |
| `BYDAutoStatisticDevice` | MainActivity, BootCompleteService |
| `BYDAutoTyreDevice` | MainActivity |

### 5.2 非标准扩展 API (未公开)

| 设备类 | 使用位置 | 说明 |
|--------|---------|------|
| `BYDAutoADASDevice` | AdasTestActivity, ReflectTestActivity | 辅助驾驶 |
| `BYDAutoPowerDevice` | (权限声明) | 电源管理 |
| `BYDAutoBigDataDevice` | (权限声明) | 大数据接口 |
| `BYDAutoCollisionDevice` | (权限声明) | 碰撞检测 |

---

## 六、权限获取策略总结

```
┌─────────────────────────────────────────────────────────────┐
│                    权限获取五层策略                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  第1层: Manifest 静态声明                                    │
│    → 48 条 BYD 权限 (含 5 类非公开扩展 API)                   │
│    → 7 条 Android 标准权限                                   │
│                                                             │
│  第2层: 系统签名                                             │
│    → BYD 内部测试 CA 签发的系统级证书                          │
│    → 自动获得 signature 级别权限                              │
│                                                             │
│  第3层: 动态权限申请                                          │
│    → SplashActivity 启动时申请 COMMON 类权限                  │
│    → SharedPreferences 记录授权状态                           │
│                                                             │
│  第4层: 标准 API 调用                                        │
│    → BYDAuto{Module}Device.getInstance(context)              │
│    → registerListener / unregisterListener                   │
│    → getXxx() / setXxx()                                    │
│                                                             │
│  第5层: 反射调用未公开 API                                    │
│    → Class.forName + getMethod + invoke                     │
│    → 绕过 SDK 接口限制访问隐藏方法                             │
│    → 出现在 ReflectTestActivity 及多处 Listener 回调中         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 七、对第三方开发的参考价值

### 7.1 可借鉴的部分

1. **权限声明模板**: 该应用的 Manifest 权限声明覆盖了所有已知模块，可作为权限声明参考
2. **动态权限申请流程**: SplashActivity 中的权限检查 → 申请 → 回调 → 记录状态的完整流程
3. **设备初始化模式**: `getInstance(context)` + `registerListener` 的标准调用模式
4. **后台监控架构**: `BOOT_COMPLETED` → `ForegroundService` 的开机自启后台监控模式
5. **悬浮窗实现**: `SYSTEM_ALERT_WINDOW` + `FloatingService` 的 HUD 式信息展示
6. **Listener 回调模式**: 继承 `AbsBYDAuto{Module}Listener` 监听各类数据变化

### 7.2 无法复制的部分

1. **系统签名**: BYD 内部测试 CA 证书，第三方无法获取，需通过开放平台申请
2. **非公开 API**: `ADAS`、`BIGDATA`、`COLLISION`、`POWER`、`REAR_VIEW_MIRROR` 等扩展权限对第三方不可用
3. **反射调用**: 非系统签名应用受 hidden API 限制，反射调用未公开方法会被拒绝

### 7.3 第三方开发者正确路径

```
1. Mock 模式开发 (无需权限)
   → 使用模拟数据完成 UI 和业务逻辑开发

2. 注册 BYD 开放平台 (并行进行)
   → 实名认证 → 创建应用 → 声明所需权限

3. 获取测试签名
   → 提交未签名 APK → BYD 平台签名后返回

4. 实车联调
   → 安装签名后的 APK → 验证权限和 API 可用性

5. 提交审核上架
   → 完善材料 → 功能/合规/体验审核 → 上架
```

---

## 附录: DEX 中发现的关键类清单

```
com.huawei.carstatushelper.SplashActivity
com.huawei.carstatushelper.MainActivity
com.huawei.carstatushelper.MainActivity$9          (StatisticListener)
com.huawei.carstatushelper.a                       (数据模型, 30+ String 字段)
com.huawei.carstatushelper.b                       (SpeedListener)
com.huawei.carstatushelper.c ~ k                   (各模块 Listener)
com.huawei.carstatushelper.activity.FuelConsumptionStatisticsActivity
com.huawei.carstatushelper.service.FloatingService
com.huawei.carstatushelper.receiver.BootCompleteService
com.huawei.carstatushelper.receiver.BootCompleteService$4  (RadarListener)
com.huawei.carstatushelper.receiver.a              (PanoramaListener)
com.huawei.carstatushelper.test.AdasTestActivity
com.huawei.carstatushelper.test.ReflectTestActivity
com.huawei.carstatushelper.util.BydManifest$permission
com.huawei.carstatushelper.util.BydManifest$permission_group
com.huawei.carstatushelper.view.EngineSpeedView
com.huawei.carstatushelper.view.MotorSpeedView
com.huawei.carstatushelper.view.RearMotorSpeedView
com.huawei.carstatushelper.view.CarSpeedView
com.huawei.carstatushelper.view.EnginePowerView
com.huawei.carstatushelper.floating.a ~ e          (悬浮窗组件)
```