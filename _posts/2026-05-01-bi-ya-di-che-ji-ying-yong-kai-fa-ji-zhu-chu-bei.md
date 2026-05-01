---
layout:     post
title:      "比亚迪车机应用开发技术储备"
subtitle:   "深入解析比亚迪生态开放平台的应用分发与API集成指南"
date:       2026-05-01
author:     "Zhangxx"
header-img: "img/post-headers/2026-05-01-2026-05-01-bi-ya-di-che-ji-ying-yong-kai-fa-ji-zhu-chu-bei.jpg"
catalog:    true
description: "本文介绍了比亚迪生态开放平台的核心业务，包括面向开发者的应用分发接入流程、审核要点以及车辆数据API库的使用方法。通过详细的步骤解析和代码示例，为开发者提供了从注册到应用发布的完整技术储备，助力高效开发符合规范的车机应用。"
tags:
    - 车机应用开发
    - API集成
    - 应用分发
    - 智能座舱
    - 技术储备
---

# 比亚迪生态开放平台 - 车机应用开发技术储备

> 来源: https://openplatform.byd.com
> 整理日期: 2026-03-29

---

## 一、平台概述

比亚迪生态开放平台聚焦智能座舱与车生活场景，涵盖 **AIoT业务** 与 **应用分发业务**，面向智能硬件开发者及车机应用开发者，提供覆盖全链路的标准化技术赋能与生态支持。

---

## 二、应用分发业务

### 2.1 业务简介

应用分发业务旨在为开发者提供全面、规范、高效的三方应用接入解决方案。接入后可借助比亚迪车机应用市场的庞大用户基础实现应用的高效触达与快速推广。

开发者可调用车机系统核心功能API，进行应用的功能扩展与深度集成。

### 2.2 接入流程

```
开发者注册 > 实名认证 > 签署业务协议 > 应用注册 > 应用开发与自测 > 版本信息完善 > 应用提交 > 应用审核 > 应用发布
```

#### 步骤详解:

1. **注册账号**: 注册比亚迪开发者账户 (https://openplatform.byd.com/#/login?type=sms)
2. **成为开发者**: 提交个人或企业开发者认证资料，通过平台审核
   - 个人开发者: 姓名、身份证号、邮箱、地址、身份证正反面及手持身份证照
   - 企业开发者: 企业名称、统一社会信用代码、法人信息、营业执照、法人身份证正反面
   - 实名认证审核时间: **3-5个工作日**
3. **签署业务协议**: 签署《应用分发业务协议》和《合规承诺》
4. **应用注册**: 填写软件包类型、应用名称、应用类型
5. **应用开发与自测**: 使用平台提供的标准API接口进行开发和自测
6. **版本信息完善**: 填写应用信息、推广信息、资质信息、自检信息、联系人信息
   - 应用信息: APK上传、应用名称、适配平台、适配分辨率、适配屏幕、所属分类、接口清单、自测报告
   - 推广信息: 应用简介、应用描述、应用图标、应用预览图、版本更新说明
   - 资质信息: APP备案类型、统一社会信用代码、ICP备案号、版权证明、应用软件许可授权书、隐私政策、用户协议
   - 自检信息: 竞品分析报告、准入标准审核表
   - 联系人信息: 联系人、邮箱、电话
7. **应用审核**: 包括"功能测试"、"合规测试"、"用户体验测试"三部分
8. **应用发布**: 审核通过后由比亚迪负责上架

### 2.3 应用市场审核要点

- 应用名称不超过6个汉字或12个英文字符，不得含敏感词、夸大词、特殊符号
- 应用图标需清晰、无侵权
- 需提供隐私政策，符合法律法规
- 企业开发者需有效营业执照，个人开发者需身份证
- 需通过功能、合规、用户体验测试

### 2.4 期待的应用类型

- 导航地图: 导航、地图、路书
- 影音娱乐: 音乐、播客、视频、资讯、相机、图片
- 生活服务: 旅游、票务
- 实用工具: 浏览器、主题、车家互联
- 电子商务: 团购

---

## 三、API库概述

开放API内容包括 **18类** 车辆数据，各模块主要通过 **get、set、监听** 三种方式开放数据:
- `get` - 获取车辆状态数据
- `set` - 控制车辆和更改车辆设置项
- `监听(registerListener)` - 实时获取各模块数据变化

> set接口的返回值仅表示命令下发是否成功，需要通过监听接口确定设置是否成功。

### JAR包下载

下载地址: https://openplatform-cdn.byd.com/dop/ecoapp-portal/sdk/bydauto-openapi.jar

---

## 四、API调用流程 (以空调类为例)

### 4.1 声明权限 (AndroidManifest.xml)

```xml
<!-- 通用权限(动态申请) -->
<uses-permission android:name="android.permission.BYDAUTO_AC_COMMON"/>

<!-- get接口权限 -->
<uses-permission android:name="android.permission.BYDAUTO_AC_GET"/>

<!-- set接口权限 -->
<uses-permission android:name="android.permission.BYDAUTO_AC_SET"/>

<!-- 媒体中心controlMedia接口额外权限 -->
<uses-permission
    android:name="com.byd.mediacenter.STARTSERVER"
    android:protectionLevel="signatureOrSystem"/>
```

需要动态权限的类: 空调类、车身类、门锁类、仪表类、全景影像类、设置类

### 4.2 创建实例

```java
// 动态申请BYDAUTO_AC_COMMON权限后
BYDAutoAcDevice bydAutoAcDevice = BYDAutoAcDevice.getInstance(mContext);
```

### 4.3 调用接口

```java
bydAutoAcDevice.start(BYDAutoAcDevice.AC_CTRL_SOURCE_VOICE);
```

### 4.4 注册监听

```java
AbsBYDAutoAcListener listener = new AbsBYDAutoAcListener() {
    @Override
    public void onAcStarted() { super.onAcStarted(); }

    @Override
    public void onAcStoped() { super.onAcStoped(); }

    @Override
    public void onAcOnlineStateChanged(int state) { super.onAcOnlineStateChanged(state); }

    @Override
    public void onAcCtrlModeChanged(int mode) { super.onAcCtrlModeChanged(mode); }

    @Override
    public void onAcCycleModeChanged(int mode) { super.onAcCycleModeChanged(mode); }
};
bydAutoAcDevice.registerListener(listener);
```

### 4.5 取消监听

```java
bydAutoAcDevice.unregisterListener(listener);
```

### 4.6 注意事项

1. 各车型配置不同、电源档位不同，某些接口不能正确返回。set接口建议在ON档电下操作
2. 接口实际的输入输出以公开的JAR包为准
3. 非实车测试时，部分接口可能返回默认值 `65535`
4. API参数描述的范围是有效数据范围，非实车环境可能返回描述之外的值
5. 调用车机接口的APK需要**系统签名**才能安装运行