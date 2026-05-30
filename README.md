# sports-video-mixer

`sports-video-mixer` 是一个用于体育短视频二创混剪的 Codex Skill。它适合把已经下载到本地文件夹里的体育视频素材，自动筛选、分组、混剪成竖屏短视频，并加入字幕、目标语言配音、宣传图小卡片和导出校验。

这个 Skill 的重点不是爬取视频，而是“把素材剪好”。推荐工作流是：先用其他下载/爬取类 Skills 从 TikTok、Instagram、Facebook、YouTube 等平台收集一批相关体育视频到本地文件夹，再用 `sports-video-mixer` 从这批素材里挑选画面干净、动作清楚、字幕较少的视频进行混剪。

## 适合做什么

- NBA、足球、拳击、电竞等体育高光混剪
- 10-15 秒竖屏短视频
- 菲律宾语、英语、中文等目标语言配音短视频
- 带品牌宣传图、活动海报、优惠图的小卡片混剪
- 从 50-100 个素材视频中筛选可用片段
- 生成用于审核的抽帧图和 contact sheet

## 前置条件

本机需要能运行：

- `ffmpeg`
- `ffprobe`
- Python 3
- 可选：`edge-tts`，用于生成菲律宾语等配音

如果没有 `edge-tts`，Skill 会倾向于创建本地 Python 虚拟环境安装，而不是污染系统 Python。

## 推荐前置 Skills

在使用 `sports-video-mixer` 之前，建议先使用下载/爬取类 Skills 准备素材。例如：

- `scrapecreators-api`：用于搜索和抓取社交平台公开内容数据
- `yt-dlp`：用于下载视频、提取音频或批量保存公开视频
- 其他浏览器自动化或社交媒体采集 Skills：用于根据关键词、热度、发布时间筛选素材

一个比较稳定的流程是：

1. 先搜索你要的主题，例如“最近 NBA 东决西决高赞视频”“NBA conference finals highlights”“Knicks Thunder Spurs Wemby SGA playoffs”等。
2. 从 TikTok、Instagram、Facebook、YouTube Shorts 等平台筛选高赞、高清、字幕较少的视频。
3. 先下载一批素材到同一个本地文件夹，建议一次准备 50-100 个视频。
4. 再调用 `sports-video-mixer`，让它从这批素材里筛选、混剪、配音和导出。

## 推荐素材规模

不要只给 3-5 个视频就让它批量生成很多条短视频。这样会导致画面重复、节奏单一、可选动作不够。

更推荐：

- 先爬取或下载 100 个相关视频到一个文件夹
- 让 Skill 生成候选 contact sheet
- 从 100 个里优先选择动作强、字幕少、人物和主题匹配的素材
- 每条 10 秒短视频通常只使用 2-3 个源视频
- 如果要生成 20 条短视频，建议至少准备 50 个以上可用素材

## 安装方式

如果你使用 `npx skills add` 安装 Skills，可以使用 GitHub 仓库地址安装：

```bash
npx skills add duxinkang/sports-video-mixer
```

如果你的 Skill 管理器需要完整 GitHub 地址，可以使用：

```bash
npx skills add https://github.com/duxinkang/sports-video-mixer
```

## 基本用法

准备好视频素材文件夹后，可以这样对 Codex 说：

```text
用 sports-video-mixer，把 /path/to/downloads/nba-conf-finals-100 这个文件夹里的视频混剪成 20 条菲律宾语配音 10 秒短视频，并加入 /path/to/promo-images 里的宣传图。
```

也可以指定更细的剪辑要求：

```text
用 sports-video-mixer，从 /path/to/videos 里挑字幕少、动作强的 NBA 高光，做 10 条 10 秒竖屏短视频。每条最多混剪 3 个源视频，同一条里只使用一种分屏模式，宣传图用小卡片弹出，不要全屏结尾广告。
```

## 和下载类 Skills 的配合示例

第一步：先下载素材。

```text
用 scrapecreators-api 和 yt-dlp，帮我找 100 个最近 NBA 东决西决的 TikTok/Instagram/Facebook 高赞、高清、字幕少的视频，下载到 /Users/me/downloads/nba-conf-finals-100。
```

第二步：再混剪。

```text
用 sports-video-mixer，把 /Users/me/downloads/nba-conf-finals-100 里的视频混剪成 20 条菲律宾语配音 10 秒短视频，并把 /Users/me/promo-images 里的宣传图以小卡片形式加入视频中。
```

## 默认剪辑偏好

这个 Skill 会优先遵守以下剪辑习惯：

- 输出竖屏 `1080x1920`
- 优先选择低字幕、动作清晰的素材
- 避免用大块黑色遮罩盖住原视频字幕
- 对底部字幕较多的视频，优先使用轻微上移取景或裁切
- 10 秒短视频中尽量只使用 2-3 个源视频
- 同一条短视频只使用一种分屏或小窗模式
- 不要每 1-2 秒频繁换分屏结构
- 让关键动作看完整，不要刚到爽点就切走
- 宣传图以 220-340px 宽的小卡片形式弹出
- 宣传图尽量出现在动作低强度时刻，不遮挡扣篮、投篮、封盖等关键画面
- 配音音量高于原视频环境声，原声保留为氛围
- 导出后用 `ffprobe` 和抽帧图检查结果

## 输出内容

通常会生成：

- `final/`：最终成片
- `review_frames/`：审核抽帧
- `review_frames/contact_sheet_midpoints.jpg`：中点总览图
- `manifest.json`：导出文件和媒体信息
- `edit_plan.json`：每条视频使用了哪些素材、时长和模板
- `voiceover/`：生成的配音音频
- `captions/`：生成的字幕卡片

## 注意事项

- 请只下载和处理你有权使用或符合平台规则的视频素材。
- 这个 Skill 不能保证自动绕过平台限制或下载私密内容。
- 如果源视频本身有很大的烧录字幕，Skill 会尽量筛选和裁切，但最好在素材准备阶段就优先选择字幕少的视频。
- 批量生成短视频前，最好先做 1-3 条样片确认风格，再批量跑 20 条、50 条或更多。
