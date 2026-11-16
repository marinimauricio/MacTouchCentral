# MacTouchCentral

**面向 macOS 外接触摸屏的、以证据为先的输入桥接工具。**

[![macOS 12+](https://img.shields.io/badge/macOS-12%2B-111111?style=flat-square&logo=apple&logoColor=white)](#要求) [![Apple 芯片](https://img.shields.io/badge/Apple%20silicon-arm64-555555?style=flat-square)](#构建) [![许可证: MIT](https://img.shields.io/badge/license-MIT-yellow?style=flat-square)](LICENSE)

MacTouchCentral 在用户态连接 USB HID 或红外触摸面框与 macOS。它读取触摸报告，将其转换为指针输入，把触摸面与正确的外接显示器关联起来，并提供诊断信息，方便逐型号验证，而不是凭感觉宣称兼容。

项目采用保守的验证规则：Windows 上确认的能力只作为证据记录；只有在 macOS 上观察到并安全复现对应报告流之后，才会标记为 macOS 支持。

## 当前状态

| 项目 | 状态 |
| --- | --- |
| MK Technology MK S2300-G100 在 macOS 上 | 单指点击、移动和拖拽已验证 |
| MK Technology MK S2300-G100 在 Windows 11 上 | 正确映射显示器后已验证十点同时触摸 |
| 该面框在 macOS 上的多点触摸 | 仍在调查；当前接口只提供一个触点 |
| 登录时启动 | 原生登录项设置，可启用或禁用 |
| 发布方式 | 源码构建已验证；签名和公证尚未完成 |

## 这个项目解决什么问题

外接触摸面板在不同系统上的行为可能不同。视频线决定显示器的位置，USB 线决定 HID 接口提供哪些报告；当有多个显示器时，两者尤其容易对应错误。

MacTouchCentral 将问题拆成三个可观察层次：

1. **设备层**：识别 USB 接口、描述符和报告格式。
2. **映射层**：选择触摸应该控制的外接显示器。
3. **交互层**：区分点击、移动和拖拽，同时不掩盖设备本身的问题。

这样，多点触摸失败时可以知道究竟是面板没有发送触点、解析器不认识格式，还是交互策略有意忽略了它。

## 参考设备

当前参考设备是 **MK Technology MK S2300-G100** 红外面框（`VID:PID 1FF7:0013`）。

### macOS

- 能稳定收到单触点报告。
- 触摸后指针会移动到对应位置。
- 单指拖拽可用。
- 实际十指测试仍只收到单触点格式。

### Windows 11

- 可以通过 Tablet PC 触摸设置把面框分配给正确的显示器。
- 浏览器多点触摸测试检测到十个同时触点。
- 这证明 Windows 配置下的硬件具备多点能力，但还不能说明 Windows 在开启多触点报告前做了什么初始化。

详见 [MK S2300-G100 测试报告](docs/MK-S2300-G100.md) 和 [项目状态](PROJECT_STATUS.md)。

## 设备矩阵

兼容性按型号和操作系统分别记录。“候选”只是调查线索，不是兼容性承诺。

| 设备或系列 | macOS 状态 | 下一项证据 |
| --- | --- | --- |
| MK S2300-G100 | 单触点已验证；多点待调查 | 只读比较 Windows USB 流量 |
| LG Smart Monitor Swing | 已有测试线索 | 用当前版本重新验证 |
| ELAN 触摸面板 | 候选 | 采集描述符并验证映射 |
| LG CreateBoard | 候选 | 验证 USB HID 报告和显示器分配 |
| 其他 USB HID 触摸屏 | 只有通用路径 | 添加可复现的型号报告 |

## 安全边界

在理解设备协议之前，多点触摸调查必须保持只读。禁止重放未公开的厂商命令、猜测性发送 `SET_REPORT`、更新固件，或发布含有机器特征的原始抓包。

公开文件不包含凭据、序列号、原始抓包、原始 HID 日志或本机路径。任何硬件结论都应注明型号、系统、测试方法和实际观察结果。

## 构建

### 要求

- macOS 12 或更高版本
- 带 macOS SDK 的 Xcode
- 当前验证目标为 Apple 芯片；Intel 构建尚未纳入测试矩阵

### 从源码构建

```bash
git clone https://github.com/marinimauricio/MacTouchCentral.git
cd MacTouchCentral
open MacTouchCentral.xcodeproj
```

命令行构建：

```bash
xcodebuild \
  -project MacTouchCentral.xcodeproj \
  -scheme MacTouchCentral \
  -destination 'platform=macOS,arch=arm64' \
  MACOSX_DEPLOYMENT_TARGET=12.0 \
  CODE_SIGNING_ALLOWED=NO \
  CODE_SIGNING_REQUIRED=NO \
  build
```

辅助脚本会构建、安装到 `/Applications` 并启动：

```bash
scripts/build_install_run.sh
```

## 第一次运行

1. 启动 MacTouchCentral，并在 macOS 提示时授予 **辅助功能** 权限。
2. 只有当设备需要读取原始报告时，才授予 **输入监控** 权限。
3. 如果有多个外接显示器，在 App 的 **Touchscreen** 菜单中选择目标显示器。
4. 使用启动设置启用或禁用登录时启动。
5. 报告问题前先打开诊断页面；可以提供计数器和显示器选择，不要提供私有原始日志。

## 开发路线

- 保持单触点路径稳定，同时调查新设备。
- 通过只读抓包识别 MK 面框从 Windows 状态切换到 macOS 多点报告的过程。
- 只有观察到真实的多触点报告格式之后，才添加解析器。
- 确认多指输入不会意外移动鼠标指针。
- 用可重复的测试流程验证 macOS 上的两指、五指和十指。
- 为每种新增面板建立独立的兼容性记录。
- 安装流程稳定后，制作签名并公证的发布版本。

## 目录结构

```text
MacTouchCentral/   macOS App、设置、菜单栏控制和诊断工具
Core/              HID 解码、触点跟踪和事件桥接
scripts/           构建、安装和测试脚本
docs/              硬件报告和项目证据
PROJECT_STATUS.md  已验证行为、待办事项和安全规则
```

## 支持项目

如果 MacTouchCentral 对你的设备有帮助，可以通过 [PayPal](https://paypal.me/mlsmarini) 支持后续硬件测试和 macOS 开发。

## 代码来源说明

当前代码仍保留 [Sebastian Hueber](https://hci.rwth-aachen.de/touchup) 及其他贡献者的上游 TouchMyMac 和 TouchUpCore 工作。MacTouchCentral 使用了新的项目身份和文档方向，但改名不会让现有实现追溯性地变成完全原创。未来的全新实现部分会单独标明。

## 许可证

[MIT](LICENSE) © 2024 Sebastian Hueber, © 2026 Jinghui Chen。其他修改和文档按其各自作者保留署名。
