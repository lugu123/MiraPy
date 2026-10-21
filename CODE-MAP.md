# 代码地图（lmentor-plain/index.js）

本文件 291,742 行 / 15.8 MB，其中约 19 万行是内联的第三方库，真正的项目代码只有 3 万多行。
加注释时插入了 **42 段区块导读**、**1751 条模块路标**、215 条函数一句话说明。
2026-10-21 又删掉了内联的旧弹幕库（danmu_api v1.21.2，551 个模块 / 29,961 行），
所以原来那段「弹幕管理面板前端 bundle」已经不在文件里了，见文末第四节。

下面用它们导航。

> 行号＝当前 index.js 的真实行号。改代码后行号会变，用编辑器全局搜索标记文字（如 `【06.2】`、`▸ 模块 0123`）更稳。

## 一、区块导读（手写，42 处）

| 行号 | 区块 |
|---:|---|
| 97 | 【01】启动外壳 · 第一段 |
| 125 | 【02】站源配置面板 lmAdminPage（youtube / torrentio / hdhub 三家共用） |
| 356 | 【03】YouTube 站源（Innertube） |
| 444 | ├ 配置存取：loadConfig / saveConfig / getConfig |
| 513 | ├ HTTP 层：axios / 代理 / 证书降级 |
| 643 | ├ Cookie：学习 · 清洗 · 注入 |
| 885 | ├ 客户端定义：UA / API Key / context |
| 1031 | ├ Innertube 请求：节流、缓存、客户端轮换 |
| 1291 | ├ 渲染树工具：从任意响应里挖条目 |
| 1689 | ├ 游标与链路日志 |
| 1830 | ├ 数据抓取 feedOnce：首页 / 分类 / 搜索共用一次 browse |
| 1861 | ├ 播放格式：DASH 合成与 HLS 逐画质 |
| 2351 | ├ 站源接口：home / homeVod / category / search / detail / play |
| 2754 | ├ PoToken：yt-pot / bgutil 两种本机服务 |
| 2960 | ├ 登录、诊断与自检 |
| 3544 | ├ 组装站源对象 makeSite |
| 3694 | └ 配置页与接口 configPlugin（/website/youtube/*） |
| 3989 | 【04】磁力下载配置页 lmTorrentAdmin |
| 4359 | 【05】备份扩展：把一个备份包打成 zip |
| 5001 | 【06】磁力内核 lmTorrent（WebTorrent + DHT + Tracker + Range） |
| 5022 | 【06.1】内联的 WebTorrent 全家桶（依赖，不是本项目代码） |
| 25815 | 【06.2】lmTorrent 自研实现 |
| 27429 | 【07】Torrentio 站源（Stremio addon + TMDB + 磁力） |
| 31166 | 【08】HdHub 站源（Stremio addon / 直链） |
| 33993 | 【09】弹幕库环境变量处理器（写 .env） |
| 34285 | 【10】弹幕库 danmu_api v1.21.6（上游整包，由更新脚本注入） |
| 85423 | 【10.1】danmu_api 自带管理页的样式（模板字符串里的 CSS） |
| 99508 | 【10.2】danmu_api 注入结束 → 回到 bootBaseDir 本体的 return |
| 99551 | 【11】面板前端 bundle 预加载 |
| 99603 | 【12】启动外壳 · 第二段 |
| 99658 | 【13】引擎本体（CatPawOpen 内核 + 全部内联依赖） |
| 99669 | 【13.1】Node 兼容垫片：WebAssembly / fetch / Headers |
| 99936 | 【13.2】语言/字符串垫片：Intl、toWellFormed、TextDecoder |
| 100142 | 【13.3】内联依赖模块区（从这里到 30 万行，9 成是第三方库） |
| 276459 | 【13.5】引擎拉起 / 停止弹幕子服务 |
| 277044 | 【13.6】弹幕面板静态资源与代理路由 |
| 284757 | 【13.7】内置站源注册表（41 个 match / 45 个站源） |
| 290747 | 【13.8】/website 面板路由插件（Oct） |
| 291247 | 【13.9】start / stop：HTTP 服务启停 |
| 291534 | 【14】插件导出：module.exports |
| 291546 | 【附】内联依赖的许可证清单（打包器生成，必须保留） |
| 291720 | 【15】收尾：把 module.exports 的键名写进启动日志 |

## 二、模块路标（自动生成，1751 条）

引擎本体里每个内联依赖模块前面都有一行 `// ▸ 模块 nnnn · 行数 · 线索`。
线索是自动识别的（库名 / 导出名 / 依赖的 node 模块），认不出来就只写行数 —— 它只是「又一个库开始了」的路标。

识别到线索的分布：

| 线索 | 模块数 |
|---|---:|
| （未识别） | 1503 |
| 近似 nanoid | 51 |
| 近似 m3u8 | 16 |
| 近似 form-data | 12 |
| 依赖 crypto | 12 |
| 近似 pino | 8 |
| 近似 fastify | 7 |
| 近似 webdav | 6 |
| 导出 default | 5 |
| 近似 iconv-lite | 5 |
| 依赖 node:events,node:util | 4 |
| 依赖 events | 4 |
| 导出 default,fastUri | 4 |
| 导出 Ajv | 4 |
| 依赖 node:assert | 4 |

前 40 条路标：

| 行号 | 模块 | 行数 | 线索 |
|---:|---:|---:|---|
| 99678 | 0001 | 254 | 近似 undici |
| 99942 | 0002 | 122 | — |
| 100086 | 0003 | 25 | — |
| 100152 | 0004 | 1850 | — |
| 102004 | 0005 | 27 | 依赖 node:process,node:stream/web,buffer |
| 102033 | 0006 | 22 | — |
| 102056 | 0007 | 82 | — |
| 102140 | 0008 | 24 | — |
| 102184 | 0009 | 57 | — |
| 102276 | 0010 | 67 | — |
| 102344 | 0011 | 101 | — |
| 102461 | 0012 | 102 | 导出 promise |
| 102565 | 0013 | 25 | 导出 default,createError |
| 102608 | 0014 | 53 | — |
| 102705 | 0015 | 75 | 依赖 node:events,node:util |
| 102834 | 0016 | 240 | 依赖 node:events,node:util |
| 103083 | 0017 | 39 | 导出 default,processWarning |
| 103124 | 0018 | 45 | 近似 fastify |
| 103184 | 0019 | 180 | — |
| 103366 | 0020 | 153 | 导出 createServer,compileValidateHTTPVersion |
| 103521 | 0021 | 102 | — |
| 103625 | 0022 | 27 | — |
| 103654 | 0023 | 66 | — |
| 103733 | 0024 | 32 | — |
| 103767 | 0025 | 28 | 近似 pino |
| 103797 | 0026 | 23 | — |
| 103822 | 0027 | 23 | — |
| 103847 | 0028 | 46 | 近似 pino |
| 103895 | 0029 | 30 | 近似 pino |
| 103960 | 0030 | 29 | — |
| 103995 | 0031 | 29 | — |
| 104026 | 0032 | 84 | — |
| 104112 | 0033 | 108 | — |
| 104222 | 0034 | 53 | — |
| 104293 | 0035 | 30 | — |
| 104332 | 0036 | 52 | 近似 pino |
| 104393 | 0037 | 51 | — |
| 104463 | 0038 | 217 | 依赖 fs,events,util |
| 104682 | 0039 | 48 | — |
| 104749 | 0040 | 28 | — |

## 三、作者自带的分区横幅（19 处）

这些是原作者写的 `/* ===== … ===== */`，用来划分站源模块；和上面的导读互相印证。

| 行号 | 横幅 |
|---:|---|
| 135 | ===== lmentor admin panel (shared config UI for youtube / torrentio / hdhub) ===== |
| 352 | ===== lmentor admin panel (end) ===== |
| 366 | ===== lmentor youtube site (innertube) ===== |
| 3985 | ===== lmentor youtube site (end) ===== |
| 3995 | ===== lmentor torrent admin panel（磁力下载 BT · 共用磁力播放的配置页） ===== |
| 4355 | ===== lmentor torrent admin panel (end) ===== |
| 4997 | ===== lmentor backup extension (end) ===== |
| 5011 | ===== lmentor torrent core (WebTorrent + DHT + Tracker + Range) ===== |
| 27425 | ===== lmentor torrent core (end) ===== |
| 27438 | ===== lmentor torrentio site (stremio addon + webtorrent) ===== |
| 31162 | ===== Lmentor Torrentio streaming site end ===== |
| 31174 | ===== lmentor hdhub site (stremio addon / direct http streams) ===== |
| 33989 | ===== lmentor hdhub site (end) ===== |
| 34281 | ===== 弹幕库 Node 环境变量处理器 (end) ===== |
| 34293 | ===== danmu_api v1.21.6（由更新脚本注入，失败可删除本段回退到内置旧版） ===== |
| 90758 | ===== \u80CC\u666F\u5F39\u5E55\u8BCD\u5E93 ===== |
| 90771 | ===== \u80CC\u666F\u5F39\u5E55\u7CFB\u7EDF ===== |
| 94995 | ===== \u989C\u8272\u6C60\u64CD\u4F5C\u51FD\u6570 ===== |
| 99514 | ===== danmu_api v1.21.6 注入结束 ===== |

## 四、已经删掉的一段：旧版弹幕库 danmu_api v1.21.2

引擎包里原本内联着完整一份 danmu_api **v1.21.2**（连它自己的管理页前端一起），
而 lmentor 的更新脚本又会把 **v1.21.6** 注入成一份独立整包，运行时通过
`globalThis.__DANMU_V1216` 顶替掉旧版 —— 旧版整棵子树永不执行。

2026-10-21 把旧版删了：**551 个模块 / 29,961 行 / 1.26 MB**，只留 v1.21.6。
判定依据：从旧版入口 `pur`/`Dqn` 求模块级引用闭包，闭包里只要被外部引用就算共用依赖留下，
其余删除；删完重新解析确认没有任何活代码还引用被删的名字（结果 0 处）。

- 被删代码原文：`removed/danmu-v1.21.2.removed.js`
- 删除工具：`tools/danmu-closure.mjs`（判定）+ `tools/prune-danmu.mjs`（执行，含自检）
- 原来的【13.4】弹幕管理面板前端 bundle（1 万多行模板字符串）就是旧版的管理页，一起删了
- 现在弹幕只有一份：【10】v1.21.6 整包 → 引擎在【13.5】里直接调用；缺包会抛明确错误

功能等价性：`tools/verify-http.mjs` 15/15 一致；`tools/verify-deep.mjs`
把面板和源码里出现过的 70 条接口逐条比对，除运行时统计/探针类接口外全部一致。
