# 截图、dump 与输入

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
