# rizsim
rizsim律动空间-一款rizline html模拟器
# RizSim · 律动空间网页模拟器

1:1 风格复刻 [Rizline（律动轨迹）](https://baike.baidu.com/item/律动轨迹) 玩法的**单文件 HTML 模拟器**（粉丝学习项目，应用名 RizSim / 律动空间，与原版游戏无关）。
按官方谱面格式与还原引擎实测数据实现：判线移动、全屏触摸判定、Tap / Catch / Hold（无严格尾判）、
多段 Riztime 主题切换、相机移动、达成率（减法）/ 得分 / 连击倍率 / 血量。

- 纯前端，**无需安装、无需联网**（无任何 CDN 依赖）
- 支持导入电脑上的 **zip 谱面包**（谱面 JSON + 曲绘 + 音乐）
- 支持 `file://` 直接打开（Chrome / Edge / Firefox / Safari 现代版本均可）
- **全屏游玩**：右上角 ⛶ 全屏按钮；手机浏览器自动铺满屏幕并适配多点触摸 / 阻止长按菜单
- **全屏判定**：点击屏幕任意位置（包括画面边缘）均可判定，无需精确对准音符
- 纯静态 HTML，直接打开即用（演示谱面已移除，请导入自己的谱面包）



## 使用方法

1. 用浏览器打开 `index.html`（手机 / 电脑均可）。
2. 点 **📦 导入谱面 ZIP**，选择你的谱面包；或点 **🗂 导入文件** 单独选 json / 音乐 / 曲绘。
3. 在选曲页确认曲绘与音乐，选择难度谱面，点 **▶ 开始**。
4. 音符（线点）从屏幕上方沿移动的「线」滑向判线，**在音符与判定环重合时点击屏幕**（任意位置均可）：
   - **Tap**：重合时点一下（比 Drag 大 1.5 倍，便于区分）
   - **Drag（Catch）**：任意按着的手指都能命中——正在按 Hold 的手指也不用松开，无需重新按下
   - **Hold**：重合时按住。**没有严格尾判**——按到结尾自动判第二次 Hit，不用掐时间松手；提前松手只会让 Hold 变暗并丢失第二次 Hit。长条永远竖直（平行于 y 轴），只跟随线的 x 轴平移
5. 结算页显示得分 / 达成率（减法，满分 100% 封顶）/ 最大连击 / 判定统计。

**游玩界面**：顶部中央横置大血条（主题 note 色边框 + 背景色内部，Riztime 跟随变色）；左上角实时 acc（主题色大号数字，**倒扣制：从 120% 起，每个 Miss 扣 2%、Bad 扣 1%，完美游玩保持 120%**）；右上角 combo（主题 note 色、无阴影）；不显示分数与 Grade 豆豆。结算页为官方 120% 达成率（0.0000%~120.0000%）。右上角小按钮（暂停/全屏，无背景模板）位于 combo 下方。

> 没有谱面？导入rizline制铺器随附的三款铺面即可体验。

### 关于 zip 结构

任意目录结构均可，模拟器会按内容自动识别：

- **谱面 JSON**：`.json`，需包含 `lines` 数组与 `bPM` 字段（官方/社区 rizline 谱面格式）
- **音乐**：`.mp3` `.ogg` `.wav` `.m4a` `.aac` `.flac` `.opus`
- **曲绘**：`.png` `.jpg` `.jpeg` `.webp` `.gif`

多个 JSON = 多个难度，在选曲页选择；多张图/多首音乐会提供下拉切换。
支持普通 ZIP 与 UTF-8 文件名（deflate / 存储两种压缩方式），支持大文件（deflate 的 ogg/mp3 等）。

### 官方编辑器谱面

直接支持 [Rizline Editor](https://www.rizwiki.cn/index.php?title=Rizline_Editor) 导出的谱面（含 `chartDelayMs` 字段，已自动作为谱面偏移处理），
把编辑器导出的 json（或连同音乐、曲绘打成 zip）导入即可。

### 兼容性

- 解压优先使用浏览器原生 `DecompressionStream`（10 秒超时），失败/不支持时自动回退**内置纯 JS inflate 解压器**（零依赖，保证任何现代浏览器都能打开）
- 音频支持 `.mp3 .ogg .wav .m4a .aac .flac .opus`（Safari 不支持 ogg 时请改用 mp3/wav/m4a）

### 快捷键（电脑端）

- `Space` / `Esc`：暂停 / 继续
- `R`：重新开始
- 鼠标点击 = 触摸

## 谱面格式支持

严格按社区反编译的官方格式解析（与 [rizlib](https://docs.rs/rizlib) 一致）：

- 根字段：`fileVersion` `songsName` `themes` `challengeTimes` `bPM` `bpmShifts` `offset` `lines` `canvasMoves` `cameraMove`
- `LinePoint`：`time`(tick/拍) `xPosition`(-0.5~0.5 屏幕宽) `color` `easeType`(0~18) `canvasIndex` `floorPosition`(米)
- `Note`：`type` 0=Tap 1=Catch 2=Hold，`time`，`floorPosition`，`otherInformations`（Hold 尾部 time/canvas/floorPosition）
- `CanvasMove`（speedKeyPoints 直接读取预计算的累计 floorPosition）、`CameraMove`（缩放/横移关键帧）
- `judgeRingColor` / `lineColor`（按官方混色规则，线身与线点颜色完全取自 JSON）
- 支持 BPM 变速（bpmShifts）、**多段 Riztime**（challengeTimes 逐段对应主题、独立半圆过渡动画、重叠时上层优先）

坐标模型：虚拟画布 540×960，判线 y=720；米→像素换算采用社区实测公式
`960 × (215/32 + 流速) × 10/129`。




## 参考资料

- [Rizline:Chart - Rizline Wiki](https://rgwiki.stary.pc.pl/wiki/Rizline:Chart)（谱面 JSON 结构）
- [Rizline:Scoring system - Rizline Wiki](https://rgwiki.stary.pc.pl/wiki/Rizline:Scoring_system)（计分/达成率/连击）
- [Rizline 谱面格式说明 - Lchzh Docs](https://docs.lchzh.net/learning/rizline/)（坐标换算实测公式）
- [rizlib - docs.rs](https://docs.rs/rizlib)（Rust 谱面解析库，字段定义）
- [Sonolus-Rizline-Engine](https://github.com/LBO44/Sonolus-Rizline-Engine)（官方判定/音符机制还原参考）
- [Rizline - 萌娘百科](https://zh.moegirl.tw/Rizline)（判定/血量/连击机制）

> 本模拟器为粉丝学习项目，与 Rizline 官方及 Pigeon Games 无关联。请仅使用你拥有权利的谱面与音乐。
