# lmentor-plain — 猫源引擎明文版（Lmentor 9.23）

> **准则：只翻译，不重写。**
> 这份代码不是「照着原版重写一遍」，而是**把原版 index.js 原封不动地摊开成明文**：
> 词元从原文件切片复制，只在空隙里补换行与缩进；面板 HTML、45 个内置站源、接口返回全部与原版一致。
> 在此基础上做了两件事：**加中文注释**、**删掉引擎里那份永远不执行的旧弹幕库**。

---

## 一、它是什么

| 项目 | 值 |
|---|---|
| 插件形态 | CatPaw / 猫源插件（`index.js` + `index.config.js`，`module.exports = { start, stop }`） |
| 引擎入口 | `/website`（面板）／`/config`（全量配置）／`/torrent/*`、`/spider/*`、`/backup` 等 |
| 源码规模 | **291,743 行 / 15.8 MB** |
| 与原版关系 | 同一份代码，**只有一处刻意删减**（旧弹幕库，见第七节），其余逐字符相同 |
| 第三方依赖 | 无。原版把 axios / undici / cheerio / webdav / webtorrent 等全部内联，这里同样内联 |

体积变化的三步，方便对账：

```
原版（压缩产物）      38,274 行 / 13.5 MB
摊开成明文            316,869 行 / 16.1 MB
+ 中文注释            321,676 行 / 16.3 MB
− 旧弹幕库 v1.21.2    291,743 行 / 15.8 MB   ← 现在这份
```

原版那 228 行「一行就是几万字符」的代码，本质是打包器输出（esbuild）。本包把它们展开成正常缩进的代码，**逻辑一个字都没动**。

---

## 二、怎么证明「没改坏」

三道校验（行为级 / 深度回归 / 词法级），全部可复跑（见 `tools/`），报告存在 `reports/`。

### 1. 行为级（最直接）——`tools/verify-http.mjs`

把原版与被测版**先后**跑在 9988 端口（引擎把端口写死为 9988），对 15 条路径比对「状态码 + 字节数 + sha256」，结果 **15/15 完全一致**：

| 路径 | 状态 | 字节 | sha256 前 12 位 |
|---|---:|---:|---|
| `/website` | 200 | 544,648 | `c6ed536ed693` |
| `/config` | 200 | 11,973 | `9d0595c69ede` |
| `/website/torrent/admin` | 200 | 18,946 | `9ecbc35f5973` |
| `/website/torrent/config` | 200 | 275 | `addc05bef391` |
| `/version`、`/api/*`、`/danmu/setting` 等 11 条 | 404/200 | 一致 | 一致 |

> `/config` 里有引擎自动生成的站源 key（含 `Date.now()` 与随机串），两次启动天然不同。
> 所以校验宿主 `tools/host-run-verify.mjs` 会在加载引擎前冻结时间与 `Math.random`，
> 只有这样才可能逐字节比对。正式运行用 `tools/host-run.mjs`，不做任何冻结。

### 2. 深度回归——`tools/verify-deep.mjs`

把 `/website` 面板里出现的路径 **加上源码里注册过的全部路由**（共 70 条）逐条比对，
另外单独比对弹幕子服务 `127.0.0.1:9321`：

- 65 条**完全一致**（状态码 + 字节 + sha256）
- 5 条有差异，全部是「返回值本身带时间/内存/上游网络结果」的接口：
  `diagnostics`、`concurrency/stats`、`inflight/stats`、`probe/worker-threads`、`115·百度二维码`
- 原版**自己和自己比**也会变：`diagnostics`、`worker-threads` 两次启动的哈希就不同；
  干净启动单独复测时 `concurrency/stats`、`cache/stats`、`http/agent-stats` 与新版本字节完全相同

弹幕子服务：`/`（管理页 470,544B）与 `/api/v2/search/anime` 两版**哈希完全相同**；
`/api/v2/search/episodes` 因为每次向上游取的内容不同，长度和哈希都会变（原版自比同样会变）。

### 3. 词法级——`tools/verify-tokens.mjs` 与 `tools/verify-diff-scope.mjs`

`verify-tokens.mjs` 只在「不做删减、只加注释」时才应当 ✅（那时 3,887,741 个词法单元与原版逐个相同）。
**本文件已删减旧弹幕，跑它必然报差异**，属于预期。

真正该看的是 `verify-diff-scope.mjs`：它回答「除了删掉的那块，别处是不是一个 token 都没动」。
做法是先按同一规则把两边 CRLF 归一成 LF（否则原版的 CRLF 会被跨行模板字面量带进 token 原文，
在第一个跨行模板处造成假差异——实测第 44,889 个 token），再做分段 diff：

| 读数 | 值 |
|---|---|
| 原版 / 明文 token 数 | 3,887,741 / 3,690,263（差 −197,478）|
| 删除窗 | 20 段，共 197,844 token |
| 插入窗 | 5 段，共 366 token（其中 14 token 是把旧分支改成抛错的 IIFE）|
| 净差 | 197,844 − 366 = **−197,478**，与 token 计数**完全吻合** |
| 未对齐尾部 | **0** |
| 窗口位置 | 全部落在原版 **L23569–L37173**（也就是 danmu 那一段），别处无窗口 |

结论：**没有被窗口覆盖的部分，两文件逐 token 完全相同**——删减只发生在那一条带里。

---

## 三、目录

```
lmentor-plain/
├── index.js                 明文引擎（291,743 行，含中文注释）—— 维护就改这个文件
├── index.config.js          插件默认配置（与原版逐字节相同）
├── index.js.md5             与 index.js 对应的 md5（改名/改内容后需要重算）
├── index.config.js.md5
├── CODE-MAP.md              代码地图：42 段导读 + 1,751 条模块路标 + 19 处作者横幅
├── README.md
├── removed/
│   └── danmu-v1.21.2.removed.js   删掉的旧弹幕库原文（551 个模块，想恢复就贴回原位）
├── reports/
│   ├── diff-scope-report.json  分段差异报告（证明改动只落在一个区间）
│   ├── token-report.json    词法单元比对报告
│   ├── http-report.json     HTTP 响应比对报告
│   └── prune-report.json    删减报告（删了哪些模块、多少行、自检结果）
└── tools/
    ├── plainify.mjs         从原版 index.js 生成明文（可复现）
    ├── annotate.mjs         往明文里插中文注释（只插整行注释，附安全检查）
    ├── annotations.data.mjs 注释内容本体：区块导读 + 函数一句话说明 + 目录
    ├── inventory.mjs        一次性导出结构清单 .tmp/inv.json（后续工具都读它）
    ├── danmu-closure.mjs    判定旧弹幕库哪些模块是死代码（引用闭包 + 外部引用保护）
    ├── prune-danmu.mjs      执行删减（含语法自检 + 悬空引用检查 + 归档）
    ├── make-code-map.mjs    生成 CODE-MAP.md
    ├── verify-tokens.mjs    词法单元级校验
    ├── verify-diff-scope.mjs 分段 diff：证明改动只落在删减区间
    ├── verify-http.mjs      黑盒 HTTP 校验（15 条）
    ├── verify-deep.mjs      深度回归（全部路由 + 弹幕子服务）
    ├── host-run.mjs         本地宿主（正式）
    └── host-run-verify.mjs  本地宿主（冻结时间/随机数，仅校验用）
```

---

## 四、注释与结构

在不改代码的前提下，插了 4 类纯注释：

| 类型 | 数量 | 长什么样 | 作用 |
|---|---:|---|---|
| 文件头 | 89 行 | `/* ═══ 这是什么 / 怎么验 / 怎么读 / 改的规矩 ═══ */` | 打开文件先看它，带目录与真实行号 |
| 区块导读 | 42 段 | `// 【07】Torrentio 站源 …` 多行说明 | 每个站源/面板/内核前说明「是什么、入口在哪、改它会怎样」 |
| 模块路标 | 1,751 条 | `// ▸ 模块 0123 · 145 行 · 近似 webtorrent` | 内联依赖的边界标记，滑到哪里都知道「又一个库开始了」 |
| 函数一句话 | 215 条 | `// ↓ 读取本源配置：…` | 只补作者原本没写注释的函数，避免重复 |

三条安全措施（`tools/annotate.mjs`）：

1. 只插**整行注释**，插在行首，不改动任何代码行；
2. 位置落在**模板字面量/跨行字符串内部**时自动跳过 —— 面板 HTML/CSS 是字符串，注释插进去会变成页面上可见的文字；
3. 插完用 `verify-http.mjs` 复验 15 条路径（15/15）。

> 想改注释内容，编辑 `tools/annotations.data.mjs` 后重新跑 `annotate.mjs`（它会先存 `.bak`）。
> 注释是按**行号锚定**的，代码改动多了以后锚点会漂，需要重新核对行号。

---

## 五、跑起来

```bash
cd lmentor-plain
node tools/host-run.mjs "$PWD/index.js"
# 浏览器打开 http://127.0.0.1:9988/website
```

放进 App：整个目录丢进猫源的插件目录（或按 App 的「导入插件」把 `index.js` + `index.config.js`
打包成 zip 导入）。插件会自行在运行目录下生成 `dudb/`、`js/`、`runtime/`、`test0.db.json` 等运行期数据，
这些都不是源码，别提交、别手改。

---

## 六、维护流程

**日常改功能**：直接编辑 `index.js`。它现在是唯一的源码，不存在「源码 → 打包」这一步，改完重启宿主即生效。

**改完自证没改坏**：

```bash
# 15 条路径逐字节比对（第二个参数是被测目录）
node tools/verify-http.mjs ../.tmp/v-orig .

# 全部路由 + 弹幕子服务
node tools/verify-deep.mjs ../.tmp/v-orig .
```

**上游出了新版本**（拿到了新的原版 index.js）：

```bash
node tools/plainify.mjs <新版index.js> ./index.js
node tools/inventory.mjs ./index.js                 # 重建结构清单
node tools/danmu-closure.mjs ./index.js             # 重新判定旧弹幕死代码
node tools/prune-danmu.mjs --write                  # 删减 + 自检 + 归档
node tools/verify-http.mjs <新版目录> .              # 必须 15/15
```

注释不会自动跟过去（按行号锚定）：按 `annotations.data.mjs` 里的标记在新文件里重新校准行号后跑 `annotate.mjs`。

`plainify.mjs` 的做法决定了它不会改坏东西：取 token 流 → 词元本体从原文件切片复制 → 只在空隙里补换行缩进 →
模板字面量整段原样复制（所以面板 HTML/CSS 的字节不受影响）。它不重排表达式、不改引号、不加删括号。

---

## 七、弹幕库：为什么曾经有两份，现在只剩一份

引擎里原来同时存在**两份完整的 danmu_api**：

| | 位置 | 版本 | 是否执行 |
|---|---|---|---|
| 旧 | 引擎包内部（内联在 esbuild 模块表里，还带它自己的管理页前端） | **v1.21.2** | ❌ 永不执行 |
| 新 | `【10】` 段，由 lmentor 更新脚本注入的独立整包 | **v1.21.6** | ✅ 实际用的就是它 |

弹幕运行时（`【13.5】`）是这样选择的：

```js
globalThis.__DANMU_V1216 ? { startDanmuApiServer: …, stopDanmuApiServer: … } : (pur(), Dqn)
```

注入包在文件加载时就把自己挂到 `globalThis.__DANMU_V1216`，所以三元表达式**永远走第一支**，
旧版那整棵子树从来没有被执行过 —— 纯死代码，但占着 3 万行、还让「改弹幕」这件事看着像有两套实现。

**2026-10-21 删掉了旧版**：551 个模块 / 29,961 行区间 / 1,258,998 字节（文件实际净减 29,747 行、1.26 MB）。

判定不是一个正则拍脑袋，而是三步：

1. **求引用闭包**：从旧版入口 `pur`/`Dqn` 出发，沿模块级引用图求可达集合（`tools/danmu-closure.mjs`，babel AST 的 Identifier，按父节点过滤掉成员属性名/对象键/声明位/参数位）；
2. **保护共用依赖**：闭包里只要被闭包外任何代码引用，就当共用依赖保留，并从它继续往外扩；
3. **删完自检**：删掉后重新解析整份文件，检查「被删掉的名字是否还被存活代码引用」—— 结果是 **0 处**（除了马上被改写的那行三元表达式）。

删除后的行为变化只有一处：注入包不存在时，旧版分支不再是「静默回退到 v1.21.2」，
而是抛一条明确错误 ——

```
[danmu] 内置旧版 danmu_api(v1.21.2) 已从本文件移除，缺少 v1.21.6 注入包（globalThis.__DANMU_V1216）
```

被删的代码原文在 `removed/danmu-v1.21.2.removed.js`，想恢复就贴回原位置。
上游仓库：<https://github.com/lugu123/danmu_api>（`huangxd-/danmu_api` 的 fork）。

---

## 八、已知边界（如实说明，不是缺陷）

1. **模板字面量内部不展开。** 面板的 HTML/CSS/JS 大量存放在模板字面量里（如 `websiteBundle` 的字符串数组）。
   为了 `/website` 输出**逐字节一致**，这些内容按原文保留——它们本来就是字符串数据，不是控制流。
2. **字符串里的 `\uXXXX` 保持转义形式。** 例如 `'\u5F39\u5E55'`（弹幕）。还原它会改写字符串原文，所以保留；阅读时按中文理解即可。
3. **`index.js` 是单文件。** 原版就是单文件插件，拆成多模块必然要改作用域与打包方式，属于「重写」，与准则冲突。
   定位靠 `CODE-MAP.md` + 编辑器搜索。
4. **端口固定 9988。** 这是原版行为（源码里 `p(9988)`），未做「优化」。
5. **旧弹幕已删，所以不再逐 token 等同于原版。** 差异只有这一处，位置已用 `verify-diff-scope.mjs`
   量出来（20 个删除窗 + 5 个插入窗，全在原版 L23569–L37173，未对齐尾部 0），被删代码有完整归档。
