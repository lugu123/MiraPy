# Lmentor 9.23（danmu 1.21.6）· YouTube 人机验证修复 + 磁力播放 + Torrentio / HdHub

在 **923 引擎**（`index.js`，danmu_api v1.21.6 版）上：

1. 修掉 **YouTube「请登录确认你不是机器人」（Sign in to confirm you're not a bot）**，并把 **账号登录（设备码）** 做进配置页；
2. 移植 902 的**磁力播放**整套能力，并做成**共用能力**：任何站源只要返回 `magnet:` 就能播；
3. 在「网盘」页加入 **磁力** 一行（与夸克等同一张表、可拖动排序）：**播放 ⇄ 转存** 一键切换、种子/播放计数、释放全部种子；
4. 按 YouTube 的形式新增 **Torrentio**、**HdHub** 两个站源（独立站源 + 独立配置页，首页/搜索走 TMDB、支持中文搜索）；
5. **不含 Stremio 通用插件桥**（按需求剔除）；
6. 其他：内置占位站源 **`20260923` → `20261019`**（就是「always last in the list」那个，名字和 KEY 一起改）、
   配置页移动端满宽 + 吸底栏、导出/WebDAV 备份带上插件配置与本地 js / 猫源 zip、
   第三方磁力站源（磁力熊 / 4K原盘）的 `require("util/torrent.js")` 兼容。

> 占位站源改名后 KEY 也变了，如果站源页原先勾选过 `20260923`，需要**重新启用一次 `20261019`**。

- 只替换 `index.js`；`index.config.js` 未改动（md5 `36c17715d0157884b6b0cddb5ab2f29a`）
- 新 `index.js` md5：`8299e6f73b968725691b17bf261d33f0`（14.03 MB，`node --check` 通过）

---

## 0-A. 磁力（网盘列表中的一行）· 播放 / 转存

「网盘」页里它就是**表格中的一行**（和夸克 / 天翼 / 移动 / P123 同一张表、同一套行样式）：

| 位置 | 表现 |
| --- | --- |
| 名称 | 「磁力」，不可改名（避免误触重命名） |
| 编辑（铅笔） | 打开配置页（`/website/torrent/admin`，iframe 嵌在同一个「修改配置」弹窗里） |
| 启用 | 开关直接写 `/torrent/config` 的 `enable`，关掉即回到 115 / 光鸭 / PikPak 转存路线 |
| 线路顺序 | 显示「播放/转存」 |
| **拖动排序** | 与其它网盘行在同一 dnd-kit 列表里，拖动即可换位置；位置记在本地 `localStorage.lmbtPos`，下次打开保持 |

点「保存」时会把这一行从 `PUT /pans/list` 里剔除，不会污染网盘自身的配置；「重置」后也会重新插回来。

详细页里的控件：

| 控件 | 作用 |
| --- | --- |
| 启用磁力下载 | 总开关；关闭后所有磁力都不拦截 |
| **磁力处理方式** | **播放（WebTorrent 边下边播）** / **转存（115 / 光鸭 / PikPak）** |
| 启动清空缓存 | 下次启动清空 torrent 缓存目录 |
| DHT 网络 | 关掉后只能靠 tracker 找源 |
| 最大连接 / 种子、空闲回收（秒）、常驻种子上限、上传限速 | 资源占用与回收策略 |
| 刷新状态 | 显示 种子 / 播放 / 串流 / 已下载 |
| 释放全部种子 | 立刻销毁全部 WebTorrent 实例，释放内存与句柄 |

关键对应关系（存盘字段仍是内核原有的 `intercept`）：

- `mode = play` ⇄ `intercept = true`（拦截磁力 → 边下边播）
- `mode = offload` ⇄ `intercept = false`（不拦截 → 恢复网盘转存路线）
- 页面上的「空闲回收（秒）」存盘为 `idleTTL`（毫秒），「上传限速（KB/s）」存盘为 `uploadLimit`（字节/秒），换算在服务端完成，配置文件格式不变。

### 0-A-0 修掉「Request has method 'GET' and cannot have a body」

配置页里的操作按钮（刷新状态 / PoToken 自检 / Cookie 自检等）走 GET，但前端 `api()` 之前不管方法都塞了 body，
浏览器直接抛 `TypeError: Request has method 'GET' and cannot have a body`。现在 GET/HEAD 一律不带 body，
只有 POST/PUT 才序列化 JSON。

### 0-A-1 手机端布局（本次一并改掉）

四张配置页（youtube / torrentio / hdhub / torrent）统一按移动端重排：

- **底栏从 `position:fixed` 改为 `position:sticky` 并移入文档流** —— 之前固定底栏盖住页面最后一块内容，这就是「滑不到最底下」的原因；
- 弹窗内容区 `max-height: min(72vh,760px)` → `min(78vh,760px)`（配合下方的负边距正好贴合，不再顶出屏幕）；
- iframe 左右用负边距（`0 -24px`）吃掉弹窗自带的 24px 内边距，宽度 `calc(100% + 48px)`；
  高度改成 `calc(78vh - 44px)`（= 内容区高度 − 上下 22px padding），这样 iframe 底边刚好落在可视区内，
  「保存配置 / 重新读取 / 连通性自检」三个按钮不会再被切掉；
- 页面 `.wrap` 改为满宽（`width:100%;max-width:none`），去掉左右大留白；
- `@media (max-width:760px)`：标题压到控件上方（不再用 210px 的左侧标签列挤占宽度）、输入框/下拉/文本域全部 `width:100%` 且字号 16px（避免 iOS 聚焦缩放）；
- 磁力面板也随这条规则重排，四个动作按钮在窄屏会自动换行。

> 另外修掉一个启动问题：磁力配置页原先用 `register(fn,{prefix:"/website/torrent"})` 注册，在你的引擎里会 404；
> 现在改成在 `lmTorrentInit` 之后**直接注册绝对路径**（与 `/torrent/*` 同一条启动路径），启动日志会打印
> `[torrent] 配置页已注册：/website/torrent/admin`，方便确认。

---

## 0-B. 共用磁力播放：任何站源给一条 magnet 就能播

新增公共对象 `globalThis.lmMagnet`（内核 `20_torrent_core` 导出），三种接入方式：

1. **零改动**：站点 `/play` 返回 `{ code:0, url:"magnet:?xt=urn:btih:..." }`
   → 引擎 `onSend` 钩子自动换成本机 `/torrent/stream` 地址（支持 `url` / `urls` / `magnet` / `extra.magnet` / `data.*`，也支持多线路数组和 `第1集$magnet:` 串）。
2. **主动调用**：`await globalThis.lmMagnet.play(magnet, request)` → `{ code, url, header, extra }`；
   只要地址不等探测时用 `lmMagnet.quick(...)`。
3. **站源包装**：`globalThis.lmMagnet.wrap(site)` —— 在 `api` 层再兜一层，钩子被别的插件干扰也不会漏。
   内置的 youtube / torrentio / hdhub 三个站源已在注册时自动套用。

猫源脚本（JS spider）：

```js
const t = __CAT_JS_UTIL.torrent;      // 完整内核
const r = await t.playUrl(magnet);    // → { url, header, extra }
__CAT_JS_UTIL.magnet.is(x);           // 同 lmMagnet：is / normalize / extract / play / quick
```

识别范围：`magnet:?xt=urn:btih:…`、40 位 hex infohash、32 位 base32、以及夹在文本/HTML 里的 magnet。

### 0-B-1 第三方磁力站源（猫源 / 本地 JS）兼容

像「磁力熊」「4K原盘」这类现成的磁力站源脚本，导入后常提示 **「该引擎内未配置磁力核心」**。原因是它们取内核的方式不止一种，而 923 的本地 JS 是 **data: URL 的 ES module**，环境差异让其中一种拿不到：

| 脚本里的写法 | 以前 | 现在 |
| --- | --- | --- |
| `require("util/torrent.js")`（磁力熊） | ESM 里没有 `require` → 直接抛错 → 判定「未内置」 | 引擎给脚本注入 `require` 兼容层，解析到 `__CAT_JS_UTIL` 里的 `torrent`（也支持 `core/xxx.js`、`axios`、`cheerio`、`crypto-js` 等） |
| `import t from "util/torrent.js"` | 已是可用路径 | 不变（`$mi` 重写为 `globalThis.__CAT_JS_UTIL["torrent"].default`） |
| `globalThis.__CAT_JS_UTIL.torrent`（4K原盘） | 已是可用路径 | 不变 |

另外把 `torrent.enabled()` 的语义改成 **「核心是否内置」恒为 true**：以前它跟着「磁力」总开关走，开关一关，站源就会误报「未内置磁力核心」。现在关掉开关时，播放会明确返回
**「磁力播放已关闭（在网盘页把「磁力」开关打开即可）」**，站源会带着这句原因回退成原始磁力链接；想看开关状态可以用 `torrent.featureOn()`。

脚本可用的内核方法：`enabled / featureOn / token / play / quickPlay / resolve / streamUrl / subtitleUrl / probe / stat / state / playUrl`。

> 注意：站源脚本内部会缓存「有没有内核」这个判断（拿到 null 就不再重试）。
> 所以替换 `index.js` 后**必须重启一次引擎**，脚本才会重新求值。

---

## 0. YouTube 人机验证怎么修的（本次重点）

### 0.1 先把问题定性

YouTube 的那道墙**不是 Cloudflare**，`cf_clearance` 这套（你给的 `cf-clearance-egern.js`）在这里用不上。
Google 判的是三件事：**IP 是否像机房/代理、会话是不是真浏览器、以及有没有绑定设备的 PoToken**。
命中后 `player` 接口返回：

```json
{"playabilityStatus":{"status":"LOGIN_REQUIRED","reason":"Sign in to confirm you're not a bot"}}
```

902 的做法是「撞墙了再想办法」，覆盖了三成场景；这次按 `cf-clearance` 的三段式结构（**学习 → 注入 → 失效检测**）重做，
并补上了它缺的两块：**PoToken 服务多协议探测** 和 **内嵌客户端兜底**。

### 0.2 学习：Cookie 能吃任何格式，且只留有用的

对应 `cf-clearance` 的「学习分支」。YouTube 侧的 token 是 `SID / SAPISID / HSID / SSID` 这一串，不是 `cf_clearance`。

- 支持**三种粘贴格式**：原始 `Cookie:` 头、浏览器插件导出的 **JSON 数组 / JSON 对象**、每行一个的文本；
- 只保留登录必需的名字（`SID/HSID/SSID/APISID/SAPISID/__Secure-*PSID/LOGIN_INFO/VISITOR_INFO1_LIVE/PREF…`），
  自动剔除 `_ga`、`_gcl_au` 这类纯追踪噪音 —— 对应 `scrubCookie()` 里剔除 `_ym_*` 的思路；
- 配置页只回显「识别到哪些字段 / 有没有 SAPISID」，不回显明文；
- `GET /website/youtube/cookie` 会真拉一次主页，读 `LOGGED_IN` 与 `accountName`，直接告诉你这个 Cookie 还能不能用。

### 0.3 注入：每个请求都带 SAPISIDHASH

对应「注入分支」。判断依据是 Cookie 里有没有 `SAPISID`：

- `SAPISIDHASH <ts>_<sha1(ts + " " + SAPISID + " " + origin)>` —— 网页登录态的官方鉴权头；
  有它 YouTube 才认这是「已登录网页会话」，机器人校验基本不再出现；
- 同时补 `X-Goog-AuthUser: 0`、`X-Origin`、`X-Goog-Api-Format-Version: 2`、`Sec-Fetch-Dest/Mode/Site`，
  把请求指纹补齐（`cf-clearance` 里排 header 顺序、补 `Sec-Fetch-*` 是同一个道理）；
- 有登录态时，**客户端顺序自动改成 web 优先**（web 带 Cookie 最稳），没有则仍旧 ios 优先；
- SHA1 用**纯 JS 实现**（不依赖 `require('crypto')` 或 crypto-js），保证在任何宿主里都算得出来，
  并已用标准向量校验：`sha1("abc") = a9993e364706816aba3e25717850c26c9cd0d89d`。

### 0.4 过盾：PoToken 服务两种协议都支持，还能自动探测

- **协议 A · bgutil-ytdlp-pot-provider**（推荐，Windows 可跑）：`POST /get_pot`，请求体 `{content_binding, client, video_id}`，
  返回 `{po_token, visitor_data}`；
- **协议 B · yt-pot**（Egern / Surge 模块）：`GET ?client=ios&videoId=…`，返回 `{ok, poToken, visitorData}`；
- 配置页留空时会依次探测：`127.0.0.1:4416/get_pot` → `127.0.0.1:8000/get_pot` → `127.0.0.1:4416` → `ytpot.token/token`，
  命中即写回配置，下次直接复用；
- 拿到的 token 分两种：**player pot**（绑定 videoId，塞进 `serviceIntegrityDimensions`）与
  **streaming pot**（绑定 visitor，拼在 googlevideo 直链后 `&pot=`），缓存 22~30 分钟；
- `GET /website/youtube/pot` 会逐个候选、逐个客户端跑一遍并给出结论与建议地址。

### 0.5 失效检测 + 兜底：内嵌客户端

对应「响应分支」。命中人机验证时：

1. 先**清掉 PoToken 缓存**（`invalidateCredentials()`），不拿旧 token 反复撞墙；
2. 有 PoToken 服务就刷新 token 重试一次；
3. 还不行 → 换成 **`tv_embedded`（TVHTML5_SIMPLY_EMBEDDED_PLAYER）+ `web_embedded`（WEB_EMBEDDED_PLAYER）**，
   带 `thirdParty.embedUrl: https://www.youtube.com/`。YouTube 对「网页内嵌播放」是另一套风控，很多 IP 在这里直接放行；
4. 全失败时返回的 `message` 会写清三条路（PoToken 服务地址 / Cookie / 换内嵌客户端），不再是一句「播放失败」。

### 0.6 你这边怎么操作（按成本从低到高）

| 方案 | 操作 | 适用 |
| --- | --- | --- |
| ① 换内嵌客户端（0 成本） | 配置页「播放客户端」选 `tv_embedded`，保存即可 | 大多数「偶发」人机验证 |
| ② 填 Cookie（1 分钟） | 浏览器登录 YouTube → 导出 Cookie（任格式）→ 粘贴到「YouTube Cookie」→ 保存 | 稳定、且能看年龄限制/订阅 |
| ③ 跑 PoToken 服务 | `docker run -d -p 4416:4416 brainicism/bgutil-ytdlp-pot-provider`，配置页「PoToken 服务地址」填 `http://127.0.0.1:4416/get_pot` | 不想用账号、又要长期稳定 |
| ④ 已有 Egern yt-pot 模块 | 「PoToken 服务地址」填 `http://ytpot.token/token` | iOS 侧已有该模块 |

验证入口：`/website/youtube/selftest`（逐项）、`/website/youtube/pot`（过盾服务）、`/website/youtube/cookie`（登录态）。

---

### 0.7 账号登录（设备码）已做进配置页

YouTube 配置页现在有四个动作按钮，对着 902 截图做的：

| 按钮 | 接口 | 说明 |
| --- | --- | --- |
| **账号登录（设备码）** | `POST /website/youtube/oauth/start` → 轮询 `/oauth/poll` | 页面直接显示 **代码** 与 **https://youtube.com/qr/activate/代码** 链接，并自动每 5 秒检查一次；也可以点「我已授权，检查结果」手动查 |
| 订阅 / 历史诊断 | `GET /feedtest` | 逐项跑 pageId、令牌续期、订阅/历史列表，输出错误明细 |
| PoToken 自检 | `GET /pot` | 逐个候选地址 × 逐个客户端跑一遍，给出可用地址建议 |
| Cookie 自检 | `GET /cookie` | 真拉主页读 `LOGGED_IN` / `accountName`，判断 Cookie 是否有效 |

登录成功后会自动 `bootstrapPageId()` 取账号页，订阅/历史随之可用；令牌过期会自动用 refresh_token 续期。

---

## 0-C. 备份：配置 + 本地 js + 猫源 zip 打成一个压缩包

「导出」现在直接给一个 **zip 压缩包**（不再是纯 JSON），「导入」也认 zip：

```
catpaw-backup-2026-10-19.zip
├── backup.json                       ← 原来的数据库配置
├── js/磁力熊.js                       ← 外部接口 → 本地 JS
├── js/4K原盘.js
└── catpaw_interfaces/
    └── cpi_xxxxxxxx/source.zip       ← 猫源接口的本地包
```

| 内容 | 来源 | 包内位置 |
| --- | --- | --- |
| youtube / torrentio / hdhub / 磁力配置 | 数据库 `/youtube`、`/torrentio`、`/hdhub`、`/torrent/config` | `backup.json` 顶层同名对象（缺失时主动补读一次） |
| 本地 JS | `<NODE_PATH>/js/*.js` | `js/*.js` |
| 猫源本地 ZIP | `<CATPAW_INTERFACE_CACHE_DIR 或 NODE_PATH>/catpaw_interfaces/**/*.zip` | 原目录结构 |

四条链路：

- **导出**：`GET /backup/` → `application/zip`，文件名 `catpaw-backup-日期.zip`；
- **导入**：`PUT /backup/` 接受 zip（前端把文件转成 base64 提交）或旧的纯 JSON，两种都能用；
- **WebDAV 快照**：手动 / 自动备份上传的也是 zip（`backup-日期-时间.zip` / `auto-日期.zip`）；
- **WebDAV 恢复**：按内容识别，zip 解包还原文件后写配置；旧的 `.json` 备份仍然可以恢复。

实现说明（为什么不是把文件塞进 JSON）：

- 之前是 `__lmFiles`（base64 数组）塞在 JSON 里，体积涨 33% 且只能靠程序解；
  现在换成标准 zip，**用系统解压工具也能直接打开看**；
- zip 写入用 STORE / DEFLATE 两种方式（能压就压），解包支持 STORE + DEFLATE，不依赖第三方库；
- 文件名带 UTF-8 标志位，中文名在 Windows / macOS 解压都不会乱码。

安全与容量：

- 单文件 ≤ 64MB、合计 ≤ 256MB（超出跳过，不影响其它条目）；
- 还原时只认 `js/*.js` 与 `catpaw_interfaces/**/*.zip`，其余（含 `backup.json`）不会被写到磁盘；
- 路径禁止 `..`、禁止绝对路径与盘符，写入统一以 `NODE_PATH` 为根。


---

## 1. 磁力播放（从 902 整块移植）

- **内核**：WebTorrent 2.8.5 + bittorrent-dht + bittorrent-tracker + ut_metadata，全部内嵌。
- **路由**：`/torrent/{stream,subtitle,stat,config,destroy,resolve,preload}`。
- **起播拦截**：`onSend` 钩子识别 `magnet:` / `.torrent`，自动建种、等元数据、边下边播；`/torrent/stat` 看进度/速率/做种数。
- **推送磁力**：push 站 `/play`、`/detail` 都已接入，详情页补体积、做种数、文件列表。
- **脚本侧**：`globalThis.__CAT_JS_UTIL.torrent` 可用。

## 2. Torrentio 站源

键名 `torrentio`，配置页 `/website/torrentio/admin`。首页 / 分类 / 搜索 / 详情走 **TMDB**（内置 Key），取流走 Torrentio 插件。

- 插件自带参数可直改：`providers / sort / language / excludeQuality / limit / sizeFilter / debridKey / debridOption`；
- 本机参数：内容过滤、体积上限、仅中日韩字幕、条数上限、tracker 来源；
- 详情按画质分组（4K / 1080P / 720P / 其他 / 自动选源），点击即交给内置磁力内核。

## 3. HdHub 站源

键名 `hdhub`，配置页 `/website/hdhub/admin`。首页 / 分类 / 搜索 / 详情走 **TMDB**，取流走 HdHub 插件。

- 插件自带参数可直改：`torbox / qualities / catalogs / content / sort / mq / mp / tb_only / tb_usenet`，改完自动重拼 base64 配置段；
- 资源四级分档：`磁力(4) > 直链(3) > 视频/HLS(2) > 普通 HTTP(1) > 网盘落地页(0)`，排序「先能播，再画质高」；
- 直链走 `/website/hdhub/stream` 代理（Range 透传 + mkv 补 `video/x-matroska`）；
- 首页对 manifest 最多等 5 秒，拉不到就先用默认目录顶上（实测首屏 23.3s → 5.0s）。

## 4. 中文搜索（两个站源都支持）

1. `/search/multi?language=zh-CN` → 2. 拆 `movie` / `tv` 各查一次 → 3. 去标点空格再查 → 4. 片名精确 + 热度排序 →
5. TMDB 整体不可达才退 IMDb 联想（Torrentio）/ Cinemeta（HdHub）。

另：`api.themoviedb.org` 国内多数网络不可达，已优先用官方别名 **`api.tmdb.org`**，失败再回落，并记住可用域名。

实测：流浪地球 7 条、甄嬛传 3 条、蜘蛛侠 20 条、隐秘的角落 2 条；Torrentio 对「流浪地球」返回 4K(8) / 1080P(12) / 自动 三组共 19 条。

---

## 5. 安装

1. 关闭引擎；
2. 用本包 `index.js` 覆盖，`index.js.md5` 一并覆盖（内容 `8299e6f73b968725691b17bf261d33f0`）；
3. `index.config.js` 不用替换；
4. 启动后进「站源」，可见 `YouTube / Torrentio / HdHub`，点「配置」打开各自页面。

## 6. 已知限制

1. **YouTube 无法在本机做真机验证**：当前环境访问 youtube.com 直接超时，因此用**全离线 mock** 覆盖逻辑链路
   （`build/test_youtube.js`：18 项全过，含「ios 撞墙 → 探测 PoToken → 内嵌客户端放行 → 直链带 pot」）。
   Cookie 是否有效、PoToken 服务是否可连，请用 `/website/youtube/cookie` 与 `/website/youtube/pot` 在你机器上确认。
2. **HdHub 部分落地页失效**（`hubdrive.pics` 返回 File not found），这类线路被评为不可播并排到最后。
3. **Torrentio 无 CDN**：国内直连首包 10~25 秒，`timeoutMs` 不要调太小。
4. **引擎整体启动**在本机卡在启动阶段（原始 923 同样如此，与补丁无关），故采用模块级单测 + `node --check` + bundle 校验。
