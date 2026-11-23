# 我的影视仓配置（GitHub 托管版）

放这两个文件到仓库根目录即可，文件名**不要改**（`config.json` 里用相对路径引用 `spider.jar`）：

| 文件 | 说明 |
|---|---|
| `config.json` | 4 个秒播站点（师兄/文才/闪电/瓜子）+ 3 组 IPTV，`spider` 写成 `./spider.jar` |
| `spider.jar` | 冻结的壳，md5 `924547ec8b148216d7636beb3213b60e`（1 049 542 B） |
| `lite.json` | 可选。纯 JSON 备用配置（5 个采集站，零代码），设备上不需要 jar |

## 为什么 `spider` 用相对路径

```json
"spider": "./spider.jar;md5;924547ec8b148216d7636beb3213b60e"
```

这样**配从一个镜像拉，jar 就从同一个镜像拉**。以后想换加速前缀（甚至换到 Gitee），
只改客户端的配置地址，不用动 `config.json`。

如果某个客户端不认相对路径，把它改成绝对地址即可，例如：

```json
"spider": "https://v4.gh-proxy.org/https://raw.githubusercontent.com/<用户名>/<仓库>/main/spider.jar;md5;924547ec8b148216d7636beb3213b60e"
```

## 各镜像地址（本机实测，2026-11-24）

把 `<用户>/<仓库>` 换成你的：

| 方式 | 地址 | 实测 |
|---|---|---|
| gh-proxy（推荐，国内快） | `https://v4.gh-proxy.org/https://raw.githubusercontent.com/<用户>/<仓库>/main/config.json` | 200 / 921 ms |
| ghfast.top | `https://ghfast.top/https://raw.githubusercontent.com/<用户>/<仓库>/main/config.json` | 200 / 1352 ms |
| jsDelivr | `https://cdn.jsdelivr.net/gh/<用户>/<仓库>@main/config.json` | 200 / 1278 ms |
| 官方 raw | `https://raw.githubusercontent.com/<用户>/<仓库>/main/config.json` | ❌ 本网络直连不通 |

同一个仓库在不同设备上可以使用不同镜像，因为是相对路径引用，互不影响。

## 上传步骤（网页版，不需要装 git）

1. github.com → New repository → 名字随意（如 `tvbox-config`）→ **Public** → Create
2. `Add file` → `Upload files` → 把 `config.json` 和 `spider.jar` 拖进去 → Commit
3. （可选）再上传 `lite.json`
4. 复制 `config.json` 的 raw 链接，前面套上 `https://v4.gh-proxy.org/`

## 必须知道的三件事

1. **仓库必须是 Public**。私有仓库的 raw 需要 token，影音客户端带不了。
2. **公开 = 任何人都能用你的这两个文件**，包括那个来源不明的二进制。介意的话就别公开。
3. **更新即全设备生效**。改了 `spider.jar` 或 `config.json`，所有填了这个地址的设备下次启动都会跟着变——这正是你要的便利，但也就没有"版本锁"了（本地化版本才有锁）。

## `snapshot/` 目录：siteconfig 存档（**不是配置，别当配置用**）

`siteconfig.json` 是壳（`libwexproxy.so`）从服务端拉下来、写在 App 私有目录的**站点域名表**：
`{"shandian":["http://shandian.blog"], "wogg":[...], ...}`，共 16 个 key。

**它不能被当成配置托管**，原因：

1. **没有读取方**。TVBox 配置格式里没有这个字段，壳只读本地固定路径
   `/data/data/com.fongmi.android.tv/files/AiWex/siteconfig`。放 GitHub 上没有任何客户端会去拉。
2. **它是派生缓存，不是源**。壳启动/定时会重写它，你放一份进去要么被覆盖、要么和壳打架。
3. **日志里有 `signature validation succeeded`**，手改的内容可能直接判无效。
4. **内容不完整**：16 个 key 里只有 `shandian`（闪电）在表内，
   师兄（AppV7）、文才、瓜子 都不在这张表里，它们走别的通道。即使能注入也只对 1/4 有效。

那它放这里有什么用？三个正当用途：

| 用途 | 说明 |
|---|---|
| **变更监控** | 定期抓一份做 diff，谁换了域名一眼可见（站点挂掉前的早期信号） |
| **灾难恢复** | 服务端不可用、域名又全挂时，可把快照推回本地路径救急（需 root，且可能被壳覆盖） |
| **排障参考** | 判断某站"打开是空的"到底是域名挂了还是接口改了 |

当前快照：`f806dd54bd8d57a4675e9dd52fea150e`（1231 B，2026-11-24 01:43 抓取）

抓取命令：

```bash
powershell.exe -File ../../adb.ps1 -Command "su 0 cat /data/data/com.fongmi.android.tv/files/AiWex/siteconfig" > snapshot/siteconfig.json
```

## 以后更新 jar 的步骤

1. 换掉仓库里的 `spider.jar`
2. 算出新 md5：`certutil -hashfile spider.jar MD5` 或 `md5sum spider.jar`
3. 把 `config.json` 里 `;md5;` 后面那串改成新值
4. commit，全设备下次启动自动生效
