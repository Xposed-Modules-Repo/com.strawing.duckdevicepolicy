# DuckDevicePolicy

Makes apps see **no device-policy restrictions** on your own device —
`DevicePolicyManager` / `UserManager` checks answered "no restriction", per
category, behind a master toggle. Formerly DuckPolicy, same package, so it updates
in place and keeps your settings and scope.

<img src="https://raw.githubusercontent.com/Xposed-Modules-Repo/com.strawing.duckdevicepolicy/main/screenshot.png" width="300" alt="DuckDevicePolicy" />

**Needs an LSPosed 2.x fork** (Xposed API 101+). Mainline 1.9.x will not load it.

**Scope matters.** `android` (System Framework) for the device-wide categories —
app install / uninstall, developer options, device-owner spoof. The target app for
everything else, including the device-owner checks Google Photos reads for Locked
Folder. Never scope Intune / Company Portal / Authenticator.

**Not working?** The LSPosed log has `installed N/M hooks in <process>`, which names
anything that did not resolve, and `first hit: <category> ...` the first time one
fires. Quote both when reporting.

Source & issues: **https://github.com/Bouteillepleine/DuckDevicePolicy** · fork of
[liyafe1997/FuckDevicePolicy](https://github.com/liyafe1997/FuckDevicePolicy).

---

### 简体中文

让 App 在你自己的设备上看到「没有任何设备策略限制」，按类别挂钩
`DevicePolicyManager` / `UserManager` 的检查并返回「无限制」，带总开关。
原名 DuckPolicy，包名不变，可直接覆盖升级。**需要 LSPosed 2.x 分支**（Xposed API 101+）。

作用域决定能否生效：**系统框架（`android`）** 用于全设备类别（应用安装/卸载、开发者选项、
设备所有者伪装）；**目标应用** 用于其余类别，包括 Google 相册锁定文件夹读取的
「设备所有者 / 完全托管检查」。请勿将 Intune / 公司门户 / Authenticator 加入作用域。

反馈问题时请附上 LSPosed 日志中的 `installed N/M hooks in <进程>` 与 `first hit:` 两行。
