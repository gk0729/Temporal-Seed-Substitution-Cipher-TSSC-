# TSSC（時種替換加密）- 中文输入法适配路线图

## 1. 概述

### 1.1 本文目的

本指南描述 TSSC 技術在中文輸入法場景下的設計思路、推進階段與未來願景。適合有興趣參與中文加密架構、輸入法安全、知識壓縮的開發者與愛好者。

### 1.2 技術定位

- **TSSC 技術現狀**：已在 GitHub 開源核心算法（Python範例），支持以時間戳做為亂數種子，動態生成每檔唯一的字符映射表。  
- **目標願景**：逐步推進 SDK 封裝、多平台適配、輸入法插件化及 i18n 全域字符支持。

### 1.3 適用範圍

- **現階段**：文本加密工具，適合技術原型、知識管理、實驗性輸入安全。
- **規劃階段**：未來適配主流中文輸入法（拼音/五筆/語音），跨多平台（Windows/macOS/Android/iOS/Web）。

***

## 2. 進度分層

### 2.1 已完成部分（v1.0）

- ✅ TSSC Python核心算法（GitHub已發布）
  - 動態亂數種子、映射表生成
  - 明文→加密→解密閉環流程
- ✅ 輸入文本加密/解密範例
  - 基本文本加密demo已可運行
- ✅ 基礎Unicode支持
  - 標準漢字、常用符號可正常加密

### 2.2 研發中的功能（v1.x）

- 🔄 API封裝（encode/decode接口、時間戳校準）
- 🔄 SDK文檔規格設計（語言多樣性、異步性能優化）
- 🔄 限定場景適配（簡易拼音/五筆本地腳本測試）

### 2.3 未來計劃（v2.0+）

- ⏳ 完整SDK包：Maven/NPM/C++/Swift封裝
- ⏳ 正式IME插件化（中文主流輸入法適配）
- ⏳ i18n極端字符支持（古文/emoji/多語擴展）
- ⏳ 跨平台時戳同步/NTP支持
- ⏳ SDK錯誤處理與容錯設計
- ⏳ 輸入速度性能壓力測試

***

## 3. 技術願景（從現階段到未來）

1. **輸入法加密閉環**  
   - 保持原有候選詞/聯想/快捷鍵體驗不變  
   - 最終輸入文本自動加密，保障隱私
  
2. **明文存儲透明還原**  
   - 無需改變用戶日常打字習慣  
   - 只需在輸入法/應用層加一個解密步驟，即可原文回顯

3. **權限與安全分層**  
   - 密鑰生成依賴本地設備時間戳（可自動/手動校準）
   - 支持按需求設置輕量/重度加密策略

4. **開源開放貢獻**  
   - 鼓勵社群參與新平台/新場景適配
   - 歡迎bug回報、性能測試、兼容性擴展

***

## 4. 問題排查與社群支持

- 輸入法卡頓/不兼容：目前僅限原型測試，正式插件化後將持續優化。
- 特殊字符映射失效：可於 Issues 報告或自行提交補全 PR。
- 時間同步失敗：暫時本地校準，未來將支持跨設備NTP同步。
- 設計問題與願景討論可通過 Issues 提交、Pull Request 補充、社群提案。

***

## 5. 參考資源與聯繫方式

- GitHub開源項目主頁：https://github.com/gk0729/Temporal-Seed-Substitution-Cipher-TSSC-
- 技術討論與願景提案：Issues 區、徵集 roadmap 補充
- 相關參考標準：UTF-8 編碼、IME API、NTP協議等（詳見官方文檔）

***

以下是願景文檔版本 [v1.0]  
2025-11-27

***

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

8. 附录

8.1 术语表（Glossary）
术语缩写 英文全称 中文名称 核心定义 
TSSC Temporal Seed Substitution Cipher 时种替换加密 基于时间戳动态生成种子，通过字符映射替换实现的可逆加密方案 
IME Input Method Editor 输入法编辑器 操作系统中负责接收用户输入、转换为目标字符的核心组件 
NTP Network Time Protocol 网络时间协议 用于跨设备同步时间戳的网络协议，保障加密/解密时间一致性 
动态映射表 Dynamic Mapping Table 动态映射表 TSSC 核心数据结构，存储「明文-密文」的时间相关映射关系（随时间戳更新） 
种子密钥 Seed Key 种子密钥 由当前时间戳生成的核心密钥，决定动态映射表的字符对应规则 

8.2 常用工具清单
工具类型 工具名称/链接 用途说明 
TSSC 调试工具 TSSC Debugger 验证加密/解密结果、查看动态映射表、模拟时间戳偏差测试 
字符编码检测工具 Charset Detector（Java）/ iconv（命令行） 校验输入法与 TSSC 编码一致性（UTF-8/GBK 等） 
输入法日志工具 Windows IME Log Viewer / Android Logcat（过滤 InputMethodService） 捕获输入法输入流程日志，定位拦截时机、加密执行异常 
时间同步工具 NTP Client（跨平台）/ 系统时间校准工具 保障跨设备时间戳同步（误差≤1s） 
兼容性测试工具 BrowserStack（Web 端）/ TestFlight（iOS）/ Firebase Test Lab（Android） 快速覆盖多系统、多输入法的兼容性测试场景 

8.3 核心代码片段速查

8.3.1 TSSC 加密/解密核心调用（跨平台）
// C/C++ 核心调用（桌面端）
#include "tssc_core.h"
std::string plaintext = "测试文本";
uint64_t timestamp = get_current_timestamp(); // 获取当前时间戳（毫秒级）
// 加密
std::string ciphertext = TSSC_Encode(plaintext.c_str(), plaintext.length(), timestamp);
// 解密
std::string decrypted = TSSC_Decode(ciphertext.c_str(), ciphertext.length(), timestamp);
// Web 端核心调用（浏览器输入法）
import { TSSCCore } from 'tssc-core';
const plaintext = "测试文本";
const timestamp = Date.now();
// 加密
const ciphertext = TSSCCore.encode(plaintext, timestamp);
// 解密
const decrypted = TSSCCore.decode(ciphertext, timestamp);
8.3.2 时间戳携带与提取示例
// 加密时附加时间戳（Java）
String ciphertext = TSSCCore.encode(plaintext, timestamp);
String output = ciphertext + "#" + timestamp; // 格式：密文#时间戳

// 解密时提取时间戳（Java）
String[] parts = input.split("#");
String ciphertext = parts[0];
long timestamp = Long.parseLong(parts[1]);
String plaintext = TSSCCore.decode(ciphertext, timestamp);
8.4 版本兼容性矩阵
TSSC 版本 支持的输入法类型 兼容系统版本 核心特性支持 
v1.0.0 拼音/五笔（离线） Windows 10+/macOS 12+/Android 9+/iOS 14+/Chrome 88+ 基础加密/解密、本地时间同步 
v1.1.0 + 语音转文字（离线） + Windows 11/macOS 13+/Android 10+ 特殊字符适配、NTP 网络时间同步 
v1.2.0 + 生僻字/Emoji 输入 + iOS 15+/Firefox 90+ 动态映射表扩展、异步加密优化 

8.5 常见错误码说明
错误码 描述 排查方向 
0x0001 时间戳偏差过大 检查设备时间同步状态、调整 NTP 同步周期、扩大时间戳容错范围（≤2s） 
0x0002 字符编码不匹配 统一使用 UTF-8 编码、通过编码检测工具校验输入文本编码 
0x0003 特殊字符未找到映射 更新 TSSC 动态映射表、添加生僻字/Emoji 对应条目 
0x0004 权限不足（初始化失败） 按平台要求申请输入法权限、系统时间访问权限 
0x0005 SDK 初始化未完成 确保输入法启动时先执行 TSSC_Init()，再调用加密/解密接口 

8.6 贡献指南

1. 若需补充新平台（如 Linux 输入法）或新输入法（如rime输入法）的适配方案，可提交 PR 至 doc/adaptation-extensions/ 目录，命名格式为「TSSC-IME-<平台/输入法名称>-适配补充.md」。

2. 发现适配问题或兼容性 Bug，可在 GitHub Issues 中按模板提交（需包含：系统版本、输入法版本、复现步骤、错误日志）。

3. 建议贡献内容：新平台适配代码、兼容性测试报告、错误码扩展说明、工具使用教程。

8.7 参考资源

• 官方文档：

◦ TSSC 核心 SDK 文档：https://github.com/gk0729/Temporal-Seed-Substitution-Cipher-TSSC-/blob/main/doc/TSSC-Core-SDK-Docs.md

◦ 各平台输入法开发指南：

◦ Windows IME：Microsoft Docs - IME 开发

◦ Android 输入法：Android Developers - 输入法框架

◦ iOS 输入法：Apple Developer - 自定义输入法

• 相关标准：

◦ UTF-8 编码标准：RFC 3629

◦ NTP 时间同步协议：RFC 5905
