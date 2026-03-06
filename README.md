# Intel NUC 10 黑苹果 (Hackintosh) EFI - macOS Sonoma

本仓库提供适用于 **Intel NUC 10 (Frost Canyon)** 的 OpenCore EFI 配置，专门为 **macOS Sonoma** 优化。此配置以 `Intel NUC10i7FNH` 为基准进行调试。

## 💻 硬件配置与支持状态

| 组件 | 详细信息 | 状态 / 备注 |
| :--- | :--- | :--- |
| **CPU** | Intel Core i7-10710U / i5-10210U | 原生支持 |
| **核显** | Intel UHD Graphics 620 | 通过 WhateverGreen 驱动，支持硬件加速 |
| **无线网卡** | Intel Wi-Fi 6 AX201 | 通过 `itlwm` / `AirportItlwm` 驱动正常工作 |
| **蓝牙** | Intel Bluetooth | 通过 `IntelBluetoothFirmware` 驱动 (见下方已知问题) |
| **读卡器** | Genesys Logic GL9755 | 通过 `GenericCardReaderFriend` / `AppleSDXC` 支持 |
| **有线网卡** | Realtek LAN Ethernet | 正常工作 |
| **声卡** | ALC256 | 前后音频接口正常输出 |

## ⚠️ 已知问题与注意事项

### 1. 蓝牙低功耗 (BLE) 音频连接问题 (Sonoma)
由于 macOS Monterey/Sonoma 中苹果对蓝牙协议栈的修改，`IntelBluetoothFirmware` 驱动无法处理 BLE 高保真音频握手。
- **影响:** AirPods 或其他基于 BLE 的耳机连接后可能会立即断开。
- **解决方案:** 使用旧版协议的无线耳机，或者直接更换原装苹果拆机网卡（如 BCM94360NG），以获得 100% 的原生体验（包括隔空投送和接力）。

### 2. SD 读卡器
NUC10 使用了 Genesys Logic GL9755 读卡器。虽然本 EFI 已经通过伪装使其支持热插拔，但**请不要在开机启动时插入 SD 卡**，这偶尔会导致启动过程卡死或变慢。

---

## 🛠 完整安装教程

### 第一步：准备安装 U 盘与 BIOS 设置

1. **制作 macOS Sonoma 安装 U 盘**
   使用 macOS 自带的 `createinstallmedia` 命令将 Sonoma 安装程序写入 U 盘。
2. **替换 EFI 文件夹**
   - 挂载 U 盘的 EFI 分区（必须是 FAT32 格式）。
   - 将本仓库中的 `EFI` 文件夹完整复制并覆盖 U 盘 EFI 分区中的原有文件夹。
3. **设置 NUC10 BIOS**
   - **强烈建议使用 BIOS 版本：v0066**（已验证的最佳版本）。
   - 关闭 Secure Boot (安全启动)。
   - 关闭 Fast Boot (快速启动)。
   - 设置 Boot Mode 为 UEFI ONLY。
   - **关于 CFG Lock（重要）：** NUC10 的最新 v0066 BIOS 已经封锁了 NVRAM 隐藏变量的写入权限，这意味着**无法通过 CFGLock.efi 等工具进行硬件级解锁**（会报错 `Problem writing variable`）。因此，本 EFI 必须且已经开启了 OpenCore 的 `AppleXcpmCfgLock=true` 选项，在软件层面动态绕过 CFG Lock 限制。

### 第二步：安装 macOS

1. 插入准备好的安装 U 盘，开机按 `F10` 调出启动菜单，选择从 U 盘启动。
2. 在 OpenCore 引导界面选择 `Install macOS Sonoma`。
3. 进入恢复界面后，打开 **磁盘工具 (Disk Utility)**。
4. 选择你的目标内置 NVMe 固态硬盘，点击“抹掉”，格式选择为 **APFS**，方案选择为 **GUID 分区图 (GUID Partition Map)**。
5. 退出磁盘工具，选择“安装 macOS”，并选择刚刚格式化好的硬盘作为目标磁盘。
6. 安装过程中系统会自动重启多次。每次重启时，请确保 OpenCore 默认选中你的内部硬盘（macOS Installer）继续安装，而不是重新进入第一步的初始安装界面。

### 第三步：安装后完善 (Post-Installation)

为了让 NUC10 能够脱离 U 盘独立启动，需要将 EFI 迁移到内部 NVMe 硬盘中：

1. 此时系统仍需插着 U 盘引导启动，进入 macOS 系统桌面。
2. 下载并打开 [MountEFI](https://github.com/corpnewt/MountEFI)（或其他 EFI 挂载工具）。
3. 分别挂载 **U 盘的 EFI 分区** 和 **内部 NVMe 硬盘的 EFI 分区**。
4. 将 U 盘 EFI 分区中的 `EFI` 文件夹，完整复制到内部 NVMe 硬盘的 EFI 分区中。
5. **生成你自己的 SMBIOS (三码):**
   - 下载并运行 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)。
   - 为型号 `Macmini8,1` (NUC 常见机型设定) 生成新的序列号。
   - 使用 ProperTree 或 OpenCore Configurator 打开内置硬盘的 `/EFI/OC/config.plist`。
   - 定位到 `PlatformInfo -> Generic`，将新生成的 `SystemProductName`、`SystemSerialNumber`、`SystemUUID` 和 `MLB` 填入对应位置并保存。
6. 拔出 U 盘，重启电脑。现在你的 NUC10 已经可以独立启动并成为一台完美的 Hackintosh。

## 四、关键配置与启动参数说明

本 EFI 内置了一些专门针对 NUC10 优化的关键配置：

- **SMBIOS 机型**: `Macmini8,1` (模拟 Mac mini 2018，最适合 NUC 的无头模式体验)。
- **CFG Lock 绕过**: `AppleXcpmCfgLock = true` (必需，见上文 BIOS 设定的说明)。
- **核心启动参数 (boot-args)**: 
  - 包含 `igfxonln=1` 强制核显在线输出，防止睡眠唤醒后黑屏。
  - 包含 `ipc_control_port_options=0` 解决特定环境下的兼容性。

---

## 💡 免责声明

*此 EFI 仅供研究和交流使用。在进行任何系统级别的修改前，请务必备份你的重要数据。原始 EFI 架构参考了 `hackintosh-club/intel-nuc10` 等开源项目并进行了 Sonoma 适配与优化。*
