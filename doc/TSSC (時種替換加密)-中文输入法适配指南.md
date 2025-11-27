# TSSC（時種替換加密）- 中文输入法适配指南

## 1. 概述

### 1.1 文档目的

本文档提供 TSSC（時種替換加密）技术与主流中文输入法的适配标准、实现步骤、兼容性测试方案及问题排查指南，适用于开发者在 GitHub 项目中集成 TSSC 加密功能，确保输入法输入流程与加密逻辑的无缝衔接。

### 1.2 核心定义

- **TSSC 技术**：基于时间戳动态替换字符映射的加密方案，通过实时生成时间相关密钥对输入文本进行加密，核心特性为「时间关联性」「映射动态性」「可逆性」。
- **适配核心目标**：在不影响输入法原有输入体验（候选词展示、联想功能、输入速度）的前提下，实现输入文本的实时加密/解密，支持「明文输入 → 加密传输/存储 → 解密还原」的全流程闭环。

### 1.3 适用范围

- **支持的中文输入法类型**：
  - 拼音输入法（全拼 / 双拼）
  - 五笔输入法
  - 语音转文字输入法（离线版）

- **支持的集成场景**：
  - 桌面端（Windows / macOS）
  - 移动端（Android / iOS）
  - Web 端（浏览器输入法）

- **兼容的 TSSC 版本**：
  - v1.0+（需支持自定义字符映射接口）

---

## 2. 适配前提

### 2.1 开发环境要求

- **编程语言**：根据输入法平台选择  
  - C/C++：适用于桌面端  
  - Java/Kotlin：适用于 Android  
  - Swift：适用于 iOS  
  - JavaScript：适用于 Web 端

- **依赖库**：TSSC 核心 SDK（需包含 `tssc_encode()` / `tssc_decode()` 接口、时间戳同步模块）。

- **调试工具**：
  - 输入法日志打印工具
  - 字符编码检测工具（如 UTF-8 / GBK 校验）
  - TSSC 加密结果验证工具

### 2.2 权限与依赖配置

- **权限要求**：
  - 允许输入法访问系统时间（用于同步 TSSC 密钥生成）
  - 允许读写输入文本缓冲区（用于加密/解密拦截）

- **依赖配置**：  
  在项目 `pom.xml` / `build.gradle` / `package.json` 中添加 TSSC SDK 依赖（示例见 3.1 节）。

---

## 3. 核心适配步骤

### 3.1 集成 TSSC 核心 SDK

#### 3.1.1 依赖引入示例

- **Maven（Java 项目）**：

```xml
<dependency>
    <groupId>com.tssc.encryption</groupId>
    <artifactId>tssc-core</artifactId>
    <version>1.0.0</version>
</dependency>
```

- **NPM（Web 项目）**：

```bash
npm install tssc-core --save
```

- **原生集成（C/C++）**：

```c
#include "tssc_core.h"          // 引入 TSSC 核心头文件
#pragma comment(lib, "tssc_core.lib") // 链接 TSSC 静态库
```

#### 3.1.2 初始化配置

在输入法启动时初始化 TSSC 模块，配置时间戳同步周期、字符编码格式（默认 UTF-8）：

```java
// Java 示例
TSSCConfig config = new TSSCConfig();
config.setTimeSyncInterval(30000); // 时间戳同步周期 30s
config.setCharset("UTF-8");        // 字符编码
TSSCCore.init(config);             // 初始化 TSSC 核心
```

---

### 3.2 输入文本加密拦截实现

#### 3.2.1 拦截时机

在输入法「文本确认输入」阶段拦截（即用户选择候选词后、文本传入目标应用前），避免干扰候选词生成逻辑。

#### 3.2.2 加密流程代码示例

```kotlin
// Android 输入法示例（基于 InputMethodService）
override fun onCommitText(text: CharSequence, newCursorPosition: Int): Boolean {
    // 1. 获取当前时间戳（与 TSSC 密钥生成同步）
    val timestamp = System.currentTimeMillis()
    // 2. 调用 TSSC 加密接口（明文 → 加密文本）
    val encryptedText = TSSCCore.encode(text.toString(), timestamp)
    // 3. 将加密文本传入目标应用
    super.onCommitText(encryptedText, newCursorPosition)
    return true
}
```

---

### 3.3 解密还原适配（可选）

若需支持加密文本回显/编辑，需在输入法「文本输入接收」阶段拦截解密：

```javascript
// Web 输入法示例
function onInputReceived(encryptedText) {
    // 1. 获取加密时对应的时间戳（需从传输/存储中同步）
    const timestamp = getEncryptedTimestamp();
    // 2. 调用 TSSC 解密接口（加密文本 → 明文）
    const plainText = TSSCCore.decode(encryptedText, timestamp);
    // 3. 回显明文到输入框
    inputElement.value = plainText;
}
```

---

### 3.4 时间戳同步机制

- **核心要求**：加密与解密使用同一时间戳（误差 ≤ 1s），否则会导致解密失败。

- **实现方案**：
  1. **本地时间同步**：定期与系统时间校准（避免设备时间偏差）。
  2. **网络时间同步（可选）**：对于跨设备场景，通过 NTP 服务器同步时间戳（需处理网络延迟）。
  3. **时间戳携带**：加密文本可附加时间戳后缀（格式：`加密文本#时间戳`），解密时自动提取。

---

## 4. 兼容性适配要点

### 4.1 字符编码兼容

- 确保 TSSC 加密/解密使用与输入法一致的编码（优先 UTF-8，避免 GBK/GB2312 等老旧编码）。
- 处理特殊字符：对 emoji、全角符号、生僻字等，需在 TSSC 映射表中预留对应条目，避免加密丢失。

### 4.2 输入法功能无侵入

- 不干扰候选词生成：加密逻辑仅作用于「最终输入文本」，不修改输入法的词库查询、联想算法。
- 保持输入速度：TSSC 加密操作需异步执行（≤ 10ms/次），避免输入卡顿。
- 支持快捷键/手势：适配输入法原有快捷键（如回车确认、退格删除），在加密状态下保证功能正常。

### 4.3 跨平台适配差异

| 平台     | 适配重点                          | 注意事项                                 |
|----------|-----------------------------------|------------------------------------------|
| Windows  | 基于 IME 接口拦截输入文本         | 需处理 32/64 位系统兼容                  |
| macOS    | 通过 InputMethodKit 框架集成      | 需申请系统输入法权限（Security & Privacy）|
| Android  | 继承 `InputMethodService` 重写 `commitText` | 适配 Android 10+ 隐私权限限制       |
| iOS      | 基于 `UIInputViewController` 实现 | 需通过 App Store 输入法审核              |
| Web 端   | 监听 `input` 事件拦截输入         | 兼容 Chrome / Firefox / Safari 等浏览器  |

---

## 5. 测试方案

### 5.1 功能测试

- **加密正确性**：输入明文后，验证加密文本与 TSSC 标准工具生成结果一致。
- **解密正确性**：加密文本解密后与原始明文完全匹配（无字符丢失 / 错乱）。
- **时间戳容错**：测试时间戳偏差 ±1s 时的解密成功率（需 ≥ 99.9%）。

### 5.2 兼容性测试

- **输入法覆盖**：测试主流输入法（搜狗、百度、讯飞、QQ 输入法）的适配效果。
- **应用覆盖**：
  - 文本编辑器（Notepad++、VS Code）
  - 聊天工具（微信、QQ）
  - 浏览器输入框
- **系统版本覆盖**：
  - Windows 10 / 11
  - macOS 12+ / 13+
  - Android 9+ / 10+ / 11+
  - iOS 14+ / 15+

### 5.3 性能测试

- **输入延迟**：统计加密操作耗时（需 ≤ 10ms/次）。
- **稳定性**：连续输入 1000 字，无崩溃、无加密失败。

---

## 6. 问题排查指南

### 6.1 常见问题及解决方案

| 问题现象               | 可能原因                              | 解决方案                                       |
|------------------------|---------------------------------------|------------------------------------------------|
| 解密后文本错乱         | 加密/解密时间戳不一致、编码不匹配    | 同步时间戳、统一使用 UTF-8 编码               |
| 输入法卡顿             | TSSC 加密操作同步执行、耗时过长      | 改为异步执行、优化加密算法效率                |
| 候选词不显示           | 加密逻辑干扰词库查询流程             | 调整拦截时机（仅在文本确认后加密）            |
| 特殊字符加密失败       | TSSC 映射表未包含该字符              | 更新 TSSC 映射表，添加特殊字符条目            |
| 权限不足导致集成失败   | 未申请系统输入法/时间访问权限        | 按平台要求配置权限，引导用户授权              |

### 6.2 日志调试建议

- **开启 TSSC 日志**：配置 `TSSCCore.setLogEnable(true)`，输出加密/解密过程、时间戳、错误码。
- **输入法日志**：记录输入文本、拦截时机、加密结果，便于定位流程问题。

---

## 7. 版本迭代与维护

- **版本兼容**：TSSC SDK 升级时，需验证旧版本适配代码的兼容性，避免 breaking change。
- **问题反馈**：在 GitHub 项目中维护 `TSSC-IME-Adaptation-Issues` 文档，收集用户适配问题并更新解决方案。
- **更新频率**：根据主流输入法版本更新（如搜狗输入法大版本迭代），同步优化适配逻辑。

---

## 8. 附录

### 8.1 参考资源

- TSSC 核心 SDK 文档：<https://github.com/tssc-project/tssc-core>
- 各平台输入法开发文档：
  - Windows IME 开发：Microsoft Docs
  - Android 输入法开发：Android Developers
  - iOS 输入法开发：Apple Developer

### 8.2 联系方式

若遇到适配问题，可在 GitHub 项目 Issues 中提交，或联系 TSSC 技术支持：`tssc-support@example.com`
