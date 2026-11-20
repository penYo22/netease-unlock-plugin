# 同步上游更新

本插件是 [sogud/mygo-music-netease](https://github.com/sogud/mygo-music-netease)
（AGPL-3.0）的融合版：在它的基础上加了会员曲解灰、9 个设置项。上游持续更新，
这里记录把上游新代码合进来的标准流程。

## 仓库结构

- `fused`（或 `main`）—— 你的融合版分支，上传到你自己的 GitHub
- `upstream/main` —— sogud 官方上游，只读跟踪

## 每次上游更新后的同步流程

### 1. 拉取上游最新

```bash
cd netease-amll-plugin
git fetch upstream
git log --oneline upstream/main --not HEAD   # 看上游新增了什么
git diff HEAD..upstream/main -- index.js      # 看具体改动
```

### 2. 三方合并

```bash
git merge upstream/main
```

- 无冲突 → 直接第 3 步。
- 有冲突（历史上总是出在 `plugin.json` 的 name/version，偶尔在 `stream()`/`lyrics()`）：
  - `plugin.json`：保留我们的 `name`（网易云音乐）、`version`（我们的版本号），
    丢弃上游的 0.1.x。
  - 解灰相关（`gdStream`/`matchBitrate`/`neteaseCDN`/`loadStream` 兜底顺序）→ 保留**我们的**。
  - 上游纯新增功能 → 采用**上游的**。

### 3. 检查「我们修过的 bug 是否被上游改回去」（必做）

1. `.mgg` 加密地址：确认取流处仍有「按 URL 文件扩展名拒绝非明文格式」，
   拒绝后自动走解灰兜底。
2. 无损码率：确认匹配服务返回的 `br` 直接用（kbps），不再被 `>1000` 除以 1000。

### 4. 回归

```bash
cd .scratch
NOLOGIN=1 SET_UNBLOCK=true node pagecheck.mjs      # 页面 12 项
NOLOGIN=1 SET_UNBLOCK=true node playlistcheck.mjs  # 歌单 0 灰标 + 可播
```

### 5. 提交并升版本号

```bash
# 改 plugin.json 的 version（如 0.8.0 -> 0.8.1）
git add -A
git commit -m "sync upstream v0.1.x"
git push origin main
```

## 首次上传到自己的 GitHub

```bash
cd netease-amll-plugin
git remote add origin https://github.com/<你>/<仓库名>.git
git push -u origin main
```

上传后，可以把插件目录里的 `.origin.json` 的 `repo` 改成你自己的仓库，
这样应用内的「更新」就指向你的融合版了。

## 为什么用 merge 而不是直接复制上游文件

直接复制会把解灰、设置项这些改动全丢掉。三方 merge 能自动合并上游没碰到的部分，
只在真正冲突的函数上让你定夺。
