## Windows User Profile Service 加载失败（开机进临时桌面）

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
