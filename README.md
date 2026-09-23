# DuckPolicy

An LSPosed / Xposed module that makes apps see **no device-policy restrictions**
on your own device. It hooks `DevicePolicyManager` and `UserManager` restriction
*checks* and returns the "no restriction" answer — **per category**, behind a
**master toggle**.

> **4.1 fixes sideloading.** If a work profile or MDM blocked installing APKs
> from normal apps (Telegram, a browser, a file manager) and DuckPolicy did not
> help, that is fixed — see *How it works* below for why the earlier builds could
> not.

> v3 is a full Kotlin rewrite and fork of
> [liyafe1997/FuckDevicePolicy](https://github.com/liyafe1997/FuckDevicePolicy).
> The original neutralises `UserManager` policies (e.g. work-profile / Intune
> device-wide restrictions); this rewrite generalises that into a per-category UI
> spanning `DevicePolicyManager` **and** `UserManager`.

Source, issues & builds: **https://github.com/Bouteillepleine/FuckDevicePolicy**

> [!WARNING]
> **4.x requires a modern-API framework.** This build uses the modern Xposed API
> (libxposed, `minApiVersion=101` / `targetApiVersion=102`), which only the
> **LSPosed 2.x forks** implement. **Mainline LSPosed 1.9.x will not load it** —
> the module will simply do nothing there. Stay on **3.1** if you are on 1.9.x.
>
> Upgrading from 3.x also **resets your settings**: a modern module gets no
> world-readable preference redirect, so 3.x toggles usually cannot be read and you
> start from the defaults (everything on, as shipped). Your **scope is preserved**.
> Upgrading 4.0 → 4.1 keeps everything; the two new categories arrive switched on.

![DuckPolicy v3 UI](https://raw.githubusercontent.com/Xposed-Modules-Repo/com.strawing.duckdevicepolicy/209fca574d143c7b459c0935f8626c684c1a761f/screenshot.png)

## Features

- **Master toggle** + **per-category** switches, with **All / None** quick actions.
- 15 categories:
  - **Camera block** — report the camera as not disabled
  - **Screenshot / screen-record block** — allow screen capture
  - **Device admin & owner checks** — look unmanaged (no admin / owner / managed profile)
  - **Password & PIN policy** — drop length, complexity, expiry and wipe rules
  - **Lock-screen feature limits** — re-enable camera, notifications, etc. on keyguard
  - **Storage-encryption enforcement** — report encryption as not required
  - **Managed app configuration** — return empty app-restriction bundles
  - **User restrictions (`DISALLOW_*`)** — clear them, incl. `UserManager.hasUserRestriction`
  - **App install / uninstall block** *(new in 4.1)* — sideload APKs from any app
    (Telegram, a browser, a file manager) and uninstall admin-locked apps
    (`no_install_unknown_sources`, `no_install_unknown_sources_globally`,
    `no_install_apps`, `no_uninstall_apps`)
  - **Developer options & USB debugging block** *(new in 4.1)* — re-enable developer
    settings and ADB (`no_debugging_features`)
  - **Auto-lock timeout** — remove the forced maximum time-to-lock
  - **Kiosk / lock-task mode** — report lock-task as permitted / unrestricted
  - **Allowed IMEs & accessibility** — remove input-method / accessibility allow-lists
  - **Misc** — auto-time, cross-profile, Bluetooth and other smaller restrictions
  - **Outlook enrollment gate** *(new in 3.1)* — hooks Outlook's own private
    `DevicePolicy` class (`requiresDeviceManagement` / `isPolicyApplied`) so it
    skips the "your organization requires device management" enrollment screen.
    Different layer than the rows above (Outlook's own gate, not the framework
    or Intune MAM SDK); only installs when scoped into
    `com.microsoft.office.outlook`.
- **Material You** dynamic colours (Android 12+) and an adaptive icon.
- Small: R8 + resource shrinking, ~1.8 MB.

## Install & scope

1. Install and enable **DuckPolicy** in LSPosed.
2. Set the module **scope**:
   - **System Framework** (`android`) for the broadest, system-wide effect — the
     recommended default for work-profile / user-restriction cases.
   - or specific target app(s) whose policy view you want to change.
3. Reboot (or force-stop the scoped processes) to apply.

> [!IMPORTANT]
> **Do not scope the MDM app itself** (e.g. Microsoft Intune / Company Portal)
> **or Microsoft Authenticator** — hooking those can expose Xposed/root to
> detection. Outlook is different and included in the default scope: it has no
> meaningful Xposed/root detection, only its own enrollment-gate check, which
> the **Outlook enrollment gate** category neutralises. It does **not** touch
> Intune MAM app-protection restrictions (screenshot block, copy/paste, PIN)
> inside Outlook — those are enforced independently and need a separate hook.
>
> Since 4.0 the settings are brokered by the framework itself (remote
> preferences) instead of a world-readable file, so a toggle applies **live** to
> every already-hooked process — no reboot. A reboot (or force-stop) is still
> needed after a **scope** change, because that is when hooks get installed. The
> app also shows you its **actual LSPosed scope**, which the legacy API could not
> report.

To see which restrictions are actually applied on your device:
`adb shell dumpsys device_policy` (look under `userRestrictions:`).

## How it works (and its limits)

Most hooks patch the **client-side** `DevicePolicyManager` / `UserManager` wrappers
inside each scoped process — installed from `onSystemServerStarting` for System
Framework and `onPackageReady` for apps — changing what an app (or the framework)
*sees* when it queries policy. Intended for your own device.

**Two categories go deeper**, because a client-side wrapper is not where the answer
is decided. `android.os.UserManager` only binder-calls `UserManagerService`, and the
code that refuses an install — inside `system_server`, and inside the package-installer
UI, which is a process the module is not even scoped into — asks that service directly.
That is why up to 4.0 the *User restrictions* category could be on and installing from
Telegram would still be blocked. Since **4.1**, the **App install / uninstall block**
and **Developer options & USB debugging block** categories hook
`com.android.server.pm.UserManagerService` itself (`hasUserRestriction`,
`hasUserRestrictionOnAnyUser`, `getUserRestrictionSource`, `getUserRestrictionSources`,
`$LocalService.getUserRestriction`), which is what the upstream module does.

Those rows answer for **every process on the device**, not just scoped ones, so unlike
every other category they are deliberately filtered to the specific `DISALLOW_*` keys
their category names, instead of flattening every restriction the system asks about.
They require **System Framework** (`android`) scope; from an app's scope they do nothing.

## Credits

- Original module and concept:
  [liyafe1997/FuckDevicePolicy](https://github.com/liyafe1997/FuckDevicePolicy).
- v3 rewrite, the 4.0 modern-API migration and the 4.1 service-side hooks:
  [Bouteillepleine/FuckDevicePolicy](https://github.com/Bouteillepleine/FuckDevicePolicy).

---

### 简体中文

该模块可以让 App 在你自己的设备上看到「没有任何设备策略限制」。它挂钩客户端的
`DevicePolicyManager` 与 `UserManager` 的限制检查，按类别返回「无限制」的结果，并带有一个总开关。
这些策略一般由「设备管理员」App 或「工作配置文件」设置（例如 Microsoft Intune），
即使在工作配置文件里设置、但会影响到整个安卓（你的个人空间）的全局策略也在覆盖范围内。

> 注意：大部分挂钩改变的是各进程中客户端所「看到」的策略。
> 请勿将 MDM App 本身（如 Intune / 公司门户）加入作用域。

> **4.1 新增：可以正常旁加载 APK 了。**「应用安装 / 卸载限制」与「开发者选项与 USB 调试限制」
> 两个类别直接挂钩 `system_server` 中的 `com.android.server.pm.UserManagerService`，
> 而不只是客户端的 `android.os.UserManager` 包装类——真正拒绝安装的代码（系统服务以及
> 安装器界面所在的进程）是直接询问该服务的，所以 4.0 及更早版本即使开启「用户限制」类别，
> 从 Telegram 等普通应用安装 APK 仍会被拦截。这两个类别对全设备生效，因此只针对各自的
> `DISALLOW_*` 键生效，并且需要 **系统框架**（`android`）作用域。

> [!WARNING]
> **4.0 需要支持新版 Xposed API 的框架**（libxposed，`minApiVersion=101` /
> `targetApiVersion=102`），目前只有 **LSPosed 2.x 分支** 实现。**原版 LSPosed 1.9.x
> 无法加载此版本**，模块将不会生效；若仍在使用 1.9.x，请继续使用 **3.1**。
> 升级后**设置会被重置**（新版模块没有全局可读的配置重定向），恢复为默认值（全部开启）；
> **作用域会保留**。
