# TOOLS.md — 本机工具笔记

记录这套环境特有的设备、参数与稳定做法，通用技能说明留在对应技能中。

## 平板

- 设备为 ZTE W202DS，Android 13，Tailscale 名称 `w202ds`，ADB 无线调试端口 5555。已安装 ADB Keyboard、Appium、ToDesk 和 Tailscale。
- 屏幕保持竖屏，自动旋转关闭，`accelerometer_rotation=0`、`user_rotation=0`。
- 网络放行依赖平板 IP `100.68.162.89` 的独立长期规则，位置在 Muse App 的 Direct network protocols 管理权限列表下方。
- 应用优先通过系统应用商店安装。大肥鱼桌宠使用已验证的 GitHub APK 侧载。
- 金铲铲和 PUBG Mobile HD 已于 2026-11-20 卸载，平板空闲约 63G/108G。

## 分包 APK 与大文件

1. 解包 `.apks`，取得 `base.apk` 和 `split_*.apk`。
2. 使用 `adb shell cat >>` 按 128KB 分片断点续传，先 `stat` 确认已接收字节，再追加缺失部分。`stat` 异常时先排查连接，保留现有文件。
3. 使用平板本地 `pm` 会话执行 `install-create`、`install-write`、`install-commit`。`install-write` 使用清单中的 split 名，例如 `config.arm64_v8a`。
4. `commit` 空输出表示已交接系统安装器，继续观察安装器 UI 并点击允许安装。
5. 文件写完后在设备端执行 `md5sum`，与源文件核对一致，再触发媒体扫描。
6. 大包安装优先使用上述分片方法，降低隧道断线风险。

## 连接与恢复

- 报故障或 `device offline` 时，先测试实际控制，再结合错误文本分析。
- 关机或重启后重建 ADB：开启无线调试，读取配对端口和配对码并执行 `adb pair`，再从主页面读取连接端口执行 `adb connect`，最后执行 `adb tcpip 5555`。
- 配对最多尝试三次，仍未成功时暂停并向老大反馈。
- 隧道探针检查 8000 端口，CONNECT 返回 200 表示通路正常。
- ADB 连接异常时同时检查平板电源；Tailscale 的 Connected 可能保留过期状态。
- adb server 作为共享常驻服务保留，单任务异常通过重建 forwarder 处理。
- forwarder 经 `hatch-egress-proxy:3130` 使用 HTTP CONNECT 连接 `100.68.162.89:5555`。3130 日志断连通常对应 VPN 隧道，等待 60 秒后换 forwarder 重试，最多三次。
- `TABLET_NO_RETRY=1` 控制重试开关，`TABLET_RETRY_SLEEP` 设置等待秒数。只读任务中途断连时重新 dump 当前状态。

## 截图、dump 与输入

- 截图使用机内 `screencap` 加 `adb pull`，dump 优先 `exec-out`。视频播放时先识别播放态并直接采用多帧方案，设备端 JPEG 异常时回落 VM 侧。
- 设备端 JPEG 使用前验证帧新鲜度。
- ADB 输入广播使用 `am broadcast -a ADB_INPUT_TEXT --es msg`。先点击并聚焦 EditText，再广播，完成后截图核对实际文本。
- 搜索时先点输入框，清空占位内容，广播关键词并核对，再点搜索。
- 上滑从内容区中部起手，底部 `y=1400/1500` 容易失效。
- 从消息页进入的视频详情页，需要返回首页后再上滑浏览。
- 长等待拆成多个短 exec，或转为后台任务，让每次 exec 控制在超时范围内。
- 切应用使用 `am start -n <包名>/<Activity>`，例如抖音 `am start -n com.ss.android.ugc.aweme/.splash.SplashActivity`，B 站 `am start -n tv.danmaku.bili/.MainActivityV2`（2026-11-22 实测：`monkey -p tv.danmaku.bili` 缺事件数切不过去，补 `-c android.intent.category.LAUNCHER 1` 才成）。
- 抖音暂停优先点画面中央；B 站播放态可用 `keyevent 85`，通用双向控制使用控制条暂停键。
- 返回使用 `tablet_fast.py run "input keyevent 4"`。
- `tablet_fast.py` 支持 `run / tap / tap_text / type_send / pxcal / shot`，截图统一调用 `shot`。
- 搜索、评论和发送等关键坐标以最新 dump 的 bounds 中心为准，截图坐标先按分辨率换算。

## 应用与发布

- 发布期间若受到并发点按干扰，使用 `drv_repl.py` 持有 adb 锁，在锁内完成选片、预览、编辑、文案、发布和验证，结束后退出并释放锁。
- 发作品时清空标题，在描述框输入完整文案。话题推荐面板出现后，点击标题行移开焦点并等待面板收起，再点发作品。
- 发作品按钮以 dump bounds 为准，已验证中心约 `(916,1866)`。
- 简介 @ 使用用户选择器：先输入 @ 前文字，通过 `input keyevent 77` 打开选择器，核对头像和抖音号后选中，蓝色高亮表示链接生效。
- B 站合集短视频切换较快，进入后立即暂停，评论前核对当前集标题，发后按评论面板计数与「刚刚」确认落点。
- `type_bili.py` 直接接收正文，参数以正文处理；需要查看用法时阅读脚本说明。
- `events.py init-round` 接收平台名 `douyin|bilibili`，`round_id` 由工具生成；`check` 使用 `<platform> <author> <title>`。
- 输入失效时检查 `dumpsys input_method` 的 `packageName`，通过 force-stop 并重启目标应用刷新 IME 绑定。

## 大肥鱼桌宠

- 安装包为 `gqpG10/dsh-pet-android v1.0.0.1`，包名 `com.dsha.pet`，APK 暂存 `/data/local/tmp`。
- 安装后使用 `pm list` 核实包名，再判断安装结果。
- 当前版本为纯离线动画桌宠，支持点按、拖拽、气泡和 106 段动画，DSH 显示「未连接」属于当前形态。

## WorkBuddy

- 模型固定使用 DeepSeek-V4.1-Flash。
- 大段代码一次发送，或转为附件，保证 ADB Keyboard 按光标插入时顺序稳定。
- DeepSeek 生成期间 UI 忙碌，使用截图核对状态。

## 其他

- ToDesk 保留 `w202ds`、`15` 和本机配置。
- 绘画在 VM 上使用代码逐笔完成，平板用于浏览和展示。
