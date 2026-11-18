# TOOLS.md - Local Notes

Short, durable notes that make external tools work reliably in this
particular setup: device nicknames, host aliases, preferred voices, and
environment-specific quirks. Skills describe how tools work in general; this
file holds only what is unique here.

## 平板：赵小喵的专属平板

- ZTE W202DS，Android 13，Tailscale 设备名 w202ds，ADB 无线调试（5555）远程接管；已装 ADB Keyboard、Appium、ToDesk、Tailscale；Tailscale 常驻在线、永不息屏。是赵小喵的专属平板；竖屏+自动旋转关闭（accelerometer_rotation=0、user_rotation=0），不要动自动旋转按钮。
- 平板连接：other-TCP（Direct network protocols）总开关不是连上平板的原因，开关一直关着；真正起作用的是平板 IP 100.68.162.89 的一条单独长期放行规则。该设置在用户手机 Muse App（设置 → Permissions → Direct network protocols），位于“管理权限”列表下方，需往下滚动找。
- 应用下载：用系统自带的应用商店，不用 Google Play、不用第三方（Aurora Store 打不开、Google Play 会崩）。例外：大肥鱼桌宠是侧载的 GitHub APK，不是常规路子。
- 金铲铲之战已安装（包名 com.tencent.jkchess；启动以桌面图标 (687,650) 上半部分点按或 query-activities 为准，pm list 不可靠）。游戏横屏 1920×1200，dump 只有 'Game view' 节点，按钮须看截图按 1920×1200 坐标点按；文字输入弹原生 EditText 悬浮条（广播 --es msg，须先清默认文本），点悬浮条“确定”提交、再点游戏内金色“确定”。
- 金铲铲细节：游戏 ID 赵小喵；赛季 S18「自然之力」；双排传送只能传一星棋子、装备不跟随，传送道具除固定回合发放外有时可在商店 D 出来买。

## 大肥鱼桌宠

- 2026-11-09 装好：gqpG10/dsh-pet-android v1.0.0.1，包名 com.dsha.pet，悬浮窗常驻桌面，106 段动画、点按 Q 弹、拖拽甩飞，点按冒气泡回话。纯离线动画版，不聊天、不联网；DSH 显示“未连接”。
- 装机核实：掉线报错不等于没装上，事后须 pm list 核实；APK 放 /data/local/tmp。让鱼像小喵一样刷抖音/B 站很难，暂不做。

## 操作要点

- 平板报故障或“device offline”先动手验证还能不能控制，再分析逻辑，不要只凭错误文本下结论。
- 关机/重启后 5555 监听消失（tcpip 模式不持久），重建流程（2026-11-18 实测）：平板开无线调试 → 配对对话框取配对端口+配对码 `adb pair`（配对一次永久有效）→ 主页面「IP 地址和端口」是连接端口，与配对端口不同、配对对话框一关配对端口就没了 → `adb connect` 连接端口 → `adb tcpip 5555` → 恢复正常流程。隧道通路探针：8000 端口 CONNECT 返回 200 即通路正常。配对失败超三次就停（老大定的）。
- Tailscale 显示在线可能是过期状态：2026-11-18 平板实际关机时 Tailscale 仍显示 Connected，ADB 连不上时要把关机列为可能原因，别只信 Tailscale 状态。
- adb server 常驻共享，禁止 kill-server；每次 exec 的 HTTPS_PROXY 令牌现签不同，常驻 forwarder 不可行；forwarder 须等端口真正监听才连。
- forwarder 经 hatch-egress-proxy:3130 用 HTTP CONNECT 连平板 100.68.162.89:5555，3130 即 Tailscale 隧道专用口——现在走的已经是平板 VPN 通道。日志里的“3130 代理断连”断的就是 VPN 隧道本身，约每轮一两次，等 60 秒重连即恢复，平板侧不用动。
- tablet_fast.py setup 重试：forwarder/建连失败时等 60 秒、换新 forwarder 重试，最多 3 次；setup 阶段尚未对平板做任何操作，所有模式都可重试；TABLET_NO_RETRY=1 关闭，TABLET_RETRY_SLEEP 改等待秒数。只读模式中途断连暂不自动重试（靠子任务手动重 dump），看后续日志再决定要不要加。
- 截图走机内 screencap + adb pull；dump 主路径 exec-out，播放视频时 dump 必失败须先识别视频态、直接跳过、单次不重试；设备端 JPEG 失败回落 VM 侧。
- 看视频多帧规则见抖音指南（单帧看不懂视频讲什么，要多截几帧看懂再评）。
- 快操作脚本 ~/workspace/tablet-audit/tablet_fast.py 与 seq.py。
- ADB_INPUT_TEXT 广播键名：`am broadcast -a ADB_INPUT_TEXT` 必须用 `--es msg`，错用 `--es text` 会静默失败（什么都不打进去也不报错）；输入框未聚焦时广播同样静默失败，须先点输入框再广播。tablet_fast.py 的 type_and_send 已按这个写（先点 EditText 再广播 --es msg），手写广播时别抄错；打完截图确认文字确实落进去了。
- 搜索框占位符陷阱：视频页顶部搜索框有占位符文本时，直接点“搜索”会按占位符搜；点搜索前务必先点搜索框、清空占位符、再广播关键词，广播后截图确认搜的词对不对。
- 上滑起手点须在内容区中部（底部 y=1400/1500 起手失效），配方见抖音指南。
- 消息页详情页不上滑：从消息页点开的视频详情页上滑无反应，须先退回首页再刷。
- exec 超时：一次 exec 里 sleep+多次 tablet_fast 会超 120 秒被 abort。长等待拆成多次短 exec，或走后台，别把几十秒等待和几次 tablet_fast 塞进同一个 exec。
- 切应用别用 monkey：monkey 被禁（MonkeyMonitorThread not allowed），切应用改走 `am start -n <包名>/<Activity>`，如抖音 `am start -n com.ss.android.ugc.aweme/.splash.SplashActivity`（应用已在后台时 intent 送达即切回）。
- 抖音暂停视频：视频在播时 keyevent 85 有时不暂停，点画面中央暂停更稳。
- 返回键：tablet_fast.py 没有 `key` 子命令，`key back` 无效；返回须走 `run "input keyevent 4"`。
- tablet_fast.py 的模式：只有 run / tap / tap_text / type_send / pxcal / shot（默认 dump）；没有 shot_jpeg 这个命令行模式（shot_jpeg 只是内部函数名，shot 模式内部调的就是它），传 shot_jpeg 只会走 dump、不截图。截图就用 shot。
- 传大文件上平板：adb push 大文件上行会 EOF 断连、平板只留 0 字节空文件；改走 adb shell cat >> 断点续传（128KB 片，先 stat 读已到大小、只追加缺失尾巴），全部落袋后设备端 md5sum 核对一致再媒体扫描。stat 失败绝不能当成 0 字节处理（会往已有文件头重复追加）。平板 wget 拿远程存储会 TLS 握手失败，平板无 curl、无浏览器，别走这条备选路。
- 发布关键流程防干扰：并发任务点按干扰发布流时，用 ~/workspace/tablet-audit/drv_repl.py 一次性持有 adb 锁独占平板（约 5.5 分钟），在锁内完成选片→预览→编辑→文案→发布→验证；对方调用锁超时跳过，不会误操作。发布完干净退出释放锁。
- 发作品文案陷阱：别点标题 EditText（20 字上限会截断文案），先清空标题、点描述框再广播完整文案；话题推荐面板会盖住底部发作品按钮，先点一下标题行移开焦点（别点进标题输入框）、等面板消失再点发作品。发作品按钮 dump bounds [646,1830][1187,1902]，点中心 (916,1866)；截图目测的 (900,1825) 落在 bounds 外、点空过一次，关键按钮一律以 dump 为准。
- 简介里的 @ 须走选择器：广播直发“@名字”只是黑色纯文本，不会变链接；先广播 @ 前的文字，再 `input keyevent 77` 触发“添加用户”选择器，点选正确的人（核对头像/抖音号），@ 段会变蓝、自动带空格；编辑页的标签点开也是同一个选择器。
- 设备端 JPEG 可能返回 stale 帧：单张直调设备端 JPEG 必须验新鲜度（capture_burst.py 的新旧帧校验 guard 工作正常，12 张全新鲜）。
- B 站暂停方法：keyevent 85 暂停可靠（播放态发 85，约 13.5 秒后视频区 mean absdiff 0.0 真冻结）、恢复不可靠（暂停态再发 85 不恢复；但视频结束后发 85 会恢复播放——暂停须在播放态发）；控制条暂停键（设备坐标约 (40,682)）双向可靠（点画面中央调出控制条、1 秒内点 (40,682)，约 14 秒后 diff 0.0；同一按钮恢复播放）；单点画面中央只显隐控制条、不切换播放/暂停。网页预览播放器页除外：该页控制条暂停键 (40,682) 无效，用 keyevent 85；视频播完会自动连播下一条，写评前须先暂停。合集 76 秒短视频连播快于 exec 周期，切集/搜到后立刻 keyevent 85 暂停再评论。
- tablet_fast.py type_send 已有双层 guard 拦截 `--` 开头参数（见 tablet_fast.py）；type_bili.py 仍无 --help，别对它用 `--help`（会被当正文打进评论框）。
- 简报写法：B 站指南路径是 bilibili-browsing-and-commenting/files/（browsing-and-language-learning 是抖音的）；events.py 的 init-round 只接平台名（douyin|bilibili），round_id 由工具生成，check 接 <platform> <author> <title>，简报里别把 round_id 当参数传。
- 输入法绑定陈旧：切平台后广播打字失效，先查 `dumpsys input_method` 的 packageName 是不是还绑在旧应用；ime set 重设无效，force-stop 目标应用重启可恢复。点按坐标务必用 dump 的 bounds 中心，目测截图坐标×1.5 易错。

## WorkBuddy

- 模型是 DeepSeek-V4.1-Flash，不要切换；发大段代码要一次发完或接受转附件（ADB Keyboard 广播按光标位置插入，会打乱顺序）。DeepSeek 回复生成时 dump 会失败（UI 忙），属正常，改截图验证。

## 其他

- ToDesk 保留。勿删 w202ds、15、本机。Windows PC 端无任务。
- 画画：不在平板上画（效率太慢）；VM 上可用代码落笔真画，不是模型生成图。
