# 平板连接与 ADB

## 平板

- 设备为 ZTE W202DS，Android 13，Tailscale 名称 `w202ds`，ADB 无线调试端口 5555。已安装 ADB Keyboard、Appium、ToDesk 和 Tailscale。
- 屏幕保持竖屏，自动旋转关闭，`accelerometer_rotation=0`、`user_rotation=0`。
- 网络放行依赖平板 IP `100.68.162.89` 的独立长期规则，位置在 Muse App 的 Direct network protocols 管理权限列表下方。
- 应用优先通过系统应用商店安装。大肥鱼桌宠使用已验证的 GitHub APK 侧载。
- 金铲铲和 PUBG Mobile HD 已于 2026-11-20 卸载，平板空闲约 63G/108G。

## 连接与恢复

- 关机或重启后重建 ADB：开启无线调试，读取配对端口和配对码并执行 `adb pair`，再从主页面读取连接端口执行 `adb connect`，最后执行 `adb tcpip 5555`。
- 配对最多尝试三次，仍未成功时暂停并向老大反馈。
- 隧道探针检查 8000 端口，CONNECT 返回 200 表示通路正常。
- adb server 作为共享常驻服务保留，单任务异常通过重建 forwarder 处理。
- forwarder 经 `hatch-egress-proxy:3130` 使用 HTTP CONNECT 连接 `100.68.162.89:5555`。3130 日志断连通常对应 VPN 隧道，等待 60 秒后换 forwarder 重试，最多三次。
- `TABLET_NO_RETRY=1` 控制重试开关，`TABLET_RETRY_SLEEP` 设置等待秒数。只读任务中途断连时重新 dump 当前状态。
