## HP OMEN 笔记本关机移动后开机风扇不转 / CPU 积热降频

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
