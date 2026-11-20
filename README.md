# 网易云音乐api · MyGo Music 融合插件

基于 [sogud/mygo-music-netease](https://github.com/sogud/mygo-music-netease)（AGPL-3.0）
的**融合版**：在它的全部功能基础上，加入了**会员曲解灰**能力。一个插件、一个音源，
`id=netease`，打开即用，不依赖任何独立后端进程。

## 与上游的差别

| 项 | 上游 sogud | 本融合版 |
| --- | --- | --- |
| 会员/无版权曲 | 按 `privilege` 标灰、不可点 | **解灰**：按歌曲 id 去公开匹配服务取真实文件 |
| 解灰设置 | 无 | 9 个设置项（见下） |
| 其它功能 | 全部 | 全部保留，并已同步到上游 v0.1.14 |

## 安装

把整个文件夹放到 MyGo Music 的 `plugins/netease/` 目录（或从 GitHub 安装本仓库），
重启应用。`id=netease`，所以 `settings.json` 的 `source` 不用改，打开就是它。

> 若你之前装的是官方 sogud 插件，本目录里会有一个 `.origin.json` 指向 sogud 仓库，
> 应用会自动更新并**把融合版覆盖回官方原版**。本仓库已把该文件改名为
> `.origin.json.disabled` 停用自动更新；上游更新请走下面的 git 同步流程。

## 会员曲解灰

`stream()` 取流顺序：官方接口 → 解灰 → 外链。

官方接口（`eapi /song/enhance/player/url/v1`）失败时（无版权 / 会员曲 / 460），
按歌曲 id 请求公开匹配服务 `music-api.gdstudio.xyz`，拿回网易云 CDN 上的真实文件。
**只发歌曲 id 和音质档位，不发账号、cookie、听歌记录。**

## 设置项（全部留空 = 零配置）

| 设置项 | 默认 | 作用 |
| --- | --- | --- |
| `UNBLOCK` | `auto` | `auto` 官方播不了才解灰 / `always` 每首都先解灰 / `off` 关闭。开关写法也认 `true`/`1`/`on` = `always`，`false`/`0`/`off` |
| `MATCH_API_URL` | 内置 gdstudio | 换自建服务，支持 `{id}` `{br}` 占位符 |
| `ENABLE_FLAC` | `true` | 无损是否要 flac（`br=740`） |
| `SELECT_MAX_BR` | `false` | `true` 不理会应用音质，一律最高 |
| `FOLLOW_SOURCE_ORDER` | `true` | `false` 解灰优先 |
| `ENABLE_PROXY` / `PROXY_URL` | `false`/空 | 音频地址走代理 |
| `ENABLE_RANDOM_CN_IP` | `false` | 请求加随机中国 IP 头 |
| `NETEASE_COOKIE` | 空 | 会员 Cookie，填了官方接口直接给会员曲地址 |

## 同步上游更新

见 [SYNC_UPSTREAM.md](./SYNC_UPSTREAM.md)。核心：本仓库用 git 跟踪上游，
上游更新时 `git fetch upstream && git merge upstream/main`，三方合并自动解决
大部分差异，只在真正冲突的函数上定夺。合并后务必检查两处易被上游改回的修复：
`.mgg` 加密地址拒绝、无损码率 kbps。

## 许可证

AGPL-3.0。上游 sogud/mygo-music-netease 的归属见 THIRD_PARTY_NOTICES.md。
