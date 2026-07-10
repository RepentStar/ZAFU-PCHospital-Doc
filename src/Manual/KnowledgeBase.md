# 知识沉淀

## 历史典型案例

## 疑难杂症解决方案

### Windows User Profile Service 加载失败（开机进临时桌面）

**问题描述**

暗影精灵9系列（HP OMEN 17-ck2xxx / Windows 11）笔记本电脑开机时偶尔黑屏，显示白色文字 "User Profile System" 配置同步/读取失败。继续开机进入桌面后，桌面主题、个性化等一切配置都变为类似重装系统后的初始化状态，仅用户头像保留。重启一到两次后桌面恢复正常，但此时用户头像反而变成默认灰色 SVG 线条人物 icon。发生频率约一两周一次。

**根因分析**

桌面主题同步失败的根本原因也是因为 Windows 用户 profile 加载失败，所以只需要关注后者：

- Windows 事件日志中出现 `User Profiles Service` 错误：事件 ID **1511**（临时 profile 登录）、**1500/1508/1509**（Access is denied）、**1552**（`User hive is loaded by another process (Registry Lock)`）[^win-prof-1] [^win-prof-2]。
- `C:\Users` 下产生多轮 `TEMP.USERNAME.*` 临时用户目录残留。
- 开机时用户注册表 hive（`NTUSER.DAT`）被进程（如 `svchost.exe`、`WmiPrvSE.exe`、`GamingServices`）锁住，Windows 加载不了 `C:\Users\username`，于是进入临时桌面。
- Windows **Fast Startup**（快速启动）是触发条件：关机会将部分内核/驱动状态写入休眠文件，若上次用户 hive 未卸载干净，坏状态会被带入下次开机。而 Restart 不走 Fast Startup，所以重启能恢复[^win-prof-3]。

**解决方案**

1. **关闭 Windows Fast Startup**（管理员 PowerShell）[^win-prof-4]：

    ```powershell
    powercfg /h off
    ```

    或修改注册表 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Power`，将 `HiberbootEnabled` 设为 `0`。

2. **清理残留临时 profile 目录**：删除 `C:\Users\TEMP.USERNAME.*` 目录（仅几百字节的 profile 壳文件，不含用户数据）。若遇到 `SFAP\cache1.bin` 等系统保护文件无法删除，可跳过，不影响修复。

3. **备份注册表**：修改前导出相关注册表项为 `.reg` 文件存放桌面。

**影响说明**

关闭 Fast Startup 后开机速度可能慢几秒到十几秒，但不会影响启动应用加载和重启速度。现代 SSD 机器体感差异通常不明显。

**验证方法**

- 查看注册表 `HiberbootEnabled` 值是否为 `0`
- 确认 `Current profile: C:\Users\username`、`Loaded: True`、`Status: 0`
- 事件日志不再出现 1511/1552 等 profile 错误

[^win-prof-1]: [Troubleshoot user profiles with events — Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/troubleshoot-user-profiles-events)

[^win-prof-2]: [User hive is loaded by another process (Registry Lock), Event ID 1552 — TheWindowsClub](https://www.thewindowsclub.com/user-hive-is-loaded-by-another-process-registry-lock-event-id-1552)

[^win-prof-3]: [User profile cannot be loaded with Fast Startup — Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/4145747/user-profile-cannot-be-loaded-with-fast-startup)

[^win-prof-4]: [How to disable Fast Startup on Windows 11 — Windows Central](https://www.windowscentral.com/software-apps/windows-11/how-to-enable-or-disable-fast-startup-on-windows-11)

---

### HP OMEN 笔记本关机移动后开机风扇不转 / CPU 积热降频

**问题描述**

暗影精灵9系列（HP OMEN 17-ck2xxx）笔记本电脑在关机拔掉电源，有概率启动时风扇完全不转动，导致快速积热、CPU 降频、系统卡爆。只能通过强制关机再开机解决，重启后风扇恢复正常。

**根因分析**

- 机器型号 `HP OMEN by HP Laptop 17-ck2xxx`，BIOS 版本 `F.15`（发布时间 2025-12-11）。
- Windows 事件日志出现 **Kernel-Processor-Power 37**：CPU 速度被系统固件限制，持续约 71 秒。与"风扇不转 → 积热 → CPU 降频 → 卡爆"的症状一致。
- 网上搜索后发现该系列已知存在 **EC（嵌入式控制器）/BIOS 风扇初始化异常**：风扇随机停转或启动后不进入正常工作模式，重启/EC reset 后恢复[^win-pow37-1] [^win-pow37-2]。

**解决方案**

按顺序尝试：

1. **EC Reset**[^win-pow37-3]
    - 关机，拔掉电源适配器
    - 长按电源键 **20–30 秒**
    - 等待 **1 分钟**（让电容彻底放电）
    - 插电开机

2. **更新 BIOS**[^win-pow37-4]
    - 访问 HP 官网驱动下载页，搜索 OMEN 17-ck2000 系列
    - 将 BIOS 从 F.15 更新到最新版本

3. **更新 HP 固件与管理软件**[^win-pow37-2]
    - 通过 **HP Support Assistant** 检查所有固件/驱动更新
    - 通过 **OMEN Gaming Hub** 检查性能/散热相关固件更新

**验证方法**

- 事件日志不再出现 `Kernel-Processor-Power 37`
- 迁移场景（关机移动后重新开机）风扇正常转动、无异常积热

[^win-pow37-1]: [OMEN 17-ck2000 FANS NOT WORKING — HP Community](https://h30434.www3.hp.com/t5/Gaming-Notebooks/OMEN-17-3-inch-Gaming-Laptop-PC-17-ck2000-FANS-NOT-WORKING/td-p/9414583)

[^win-pow37-2]: [Common Issues Troubleshooting Guide From Former HP OMEN Support Team — Reddit](https://www.reddit.com/r/HPOmen/comments/11n3zaz/common_issues_troubleshooting_guide_from_former/)

[^win-pow37-3]: [HP Laptop — Performing a Hard Reset (EC Reset)](https://support.hp.com/us-en/document/ish_3939773-2337994-16)

[^win-pow37-4]: [HP OMEN 17-ck2000 驱动与 BIOS 下载](https://support.hp.com/us-en/drivers/laptops)
