# DuckDevicePolicy

Makes apps see **no device-policy restrictions** on your own device. Hooks
`DevicePolicyManager` / `UserManager` checks and returns the "no restriction"
answer, per category, behind a master toggle.

Formerly **DuckPolicy** — same package, so it updates in place and keeps your
settings and scope.

> [!WARNING]
> Needs an **LSPosed 2.x fork** (modern Xposed API 101+). Mainline LSPosed 1.9.x
> will not load it — stay on 3.1 there.

## Scope decides what can work

- **System Framework (`android`)** — required for the device-wide categories:
  app install / uninstall, developer options & USB debugging, device-owner spoof.
  Those checks happen in `system_server` and in the installer process, not inside
  the app you are using.
- **The target app** — required for everything else, including **Device-owner /
  fully-managed checks**, which is what Google Photos reads before refusing to set
  up Locked Folder.

Do not scope the MDM app itself (Intune / Company Portal) or Authenticator.

## Categories

Camera · screen capture · device admin & managed profile · **device owner /
fully managed** · password & PIN · keyguard · encryption · managed app config ·
user restrictions · **app install / uninstall** · **developer options & ADB** ·
auto-lock · kiosk · IME & accessibility allow-lists · misc · Outlook enrollment
gate · device-wide device-owner spoof.

All on by default except the last. Toggles apply live; a scope change needs a
reboot.

## If a category seems to do nothing

The LSPosed log says what happened:

```
installed 68/68 hooks in system_server
first hit: package_install via UserManagerService#hasUserRestriction in system_server
```

`installed N/M` names every row that did not resolve; `first hit:` appears the
first time a category actually fires. Quote both when reporting a problem. The
Diagnostics tab is usually empty — the framework's file store is root-owned, so
the log is the only channel that works.

Source & issues: **https://github.com/Bouteillepleine/FuckDevicePolicy** ·
fork of [liyafe1997/FuckDevicePolicy](https://github.com/liyafe1997/FuckDevicePolicy).

---

### 简体中文

让 App 在你自己的设备上看到「没有任何设备策略限制」，按类别挂钩
`DevicePolicyManager` / `UserManager` 的检查并返回「无限制」，带总开关。
原名 DuckPolicy，包名不变，可直接覆盖升级。

**需要 LSPosed 2.x 分支**（新版 Xposed API 101+），原版 1.9.x 无法加载。

作用域决定能否生效：**系统框架（`android`）** 用于全设备类别（应用安装/卸载、
开发者选项与 USB 调试、设备所有者伪装）；**目标应用** 用于其余类别，包括
「设备所有者 / 完全托管检查」（Google 相册锁定文件夹读取的就是它）。
请勿将 MDM App（Intune / 公司门户）或 Authenticator 加入作用域。

反馈问题时，请附上 LSPosed 日志中的 `installed N/M hooks in <进程>` 与
`first hit:` 两行。
