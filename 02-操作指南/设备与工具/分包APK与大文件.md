# 分包 APK 与大文件

1. 解包 `.apks`，取得 `base.apk` 和 `split_*.apk`。
2. 使用 `adb shell cat >>` 按 128KB 分片断点续传，先 `stat` 确认已接收字节，再追加缺失部分。`stat` 异常时先排查连接，保留现有文件。
3. 使用平板本地 `pm` 会话执行 `install-create`、`install-write`、`install-commit`。`install-write` 使用清单中的 split 名，例如 `config.arm64_v8a`。
4. `commit` 空输出表示已交接系统安装器，继续观察安装器 UI 并点击允许安装。
5. 文件写完后在设备端执行 `md5sum`，与源文件核对一致，再触发媒体扫描。
6. 大包安装优先使用上述分片方法，降低隧道断线风险。
