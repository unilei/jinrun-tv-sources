# JinRun TV 源包（sources bundle）

这个仓库只放**一个文件**：`sources.json`。它是 JinRun TV 的订阅入口——
App 里填**一个**链接，就能加载下面全部直播源。

## 怎么用

App → 直播源 → 「订阅源包」，粘贴：

```
https://github.com/unilei/jinrun-tv-sources/blob/main/sources.json
```

> **本仓库是私有的，别用 `raw.githubusercontent.com` 那条链接。**
> `raw.githubusercontent.com` 对私有仓库一律返回 **404**，而且**不认令牌**——实测过。
> 用上面的 `github.com/.../blob/...` 形式：App 检测到配了令牌时，会自动把它改写成
> `https://api.github.com/repos/unilei/jinrun-tv-sources/contents/sources.json?ref=main`
> 并带上 `Accept: application/vnd.github.raw`。三种写法都认：
>
> - `github.com/{owner}/{repo}/blob/{ref}/{path}`
> - `raw.githubusercontent.com/{owner}/{repo}/{ref}/{path}`（公开仓库用）
> - `api.github.com/repos/{owner}/{repo}/contents/{path}?ref={ref}`
>
> 所以 App 里还要在「私密参数」里配一个 GitHub 令牌（classic PAT 勾 `repo`，
> 或 fine-grained PAT 只给这一个仓库的 `Contents: Read`）。令牌只存在设备本地。

之后改源只需要改这个仓库并 push，App 下次启动自动生效，**不需要重新装 APK**。
热更新走的是条件请求：带上次的 `ETag`，源包没变时 GitHub 回 `304`（几十字节），
不会把整个 JSON 重下一遍。

## 格式

```jsonc
{
  "schema": 1,              // 格式版本。App 遇到不认识的版本会拒绝而不是猜着解析
  "name": "JinRun TV 精选直播源",
  "updatedAt": "2026-10-05T03:16:27Z",   // 变更检测用；也显示在 App 里
  "sources": [
    {
      "id": "hyqyk",        // 长期标识，见下
      "name": "虎牙一起看",
      "url": "https://sub.ottiptv.cc/huyayqk.m3u",
      "userAgent": "okHttp/Mod-1.5.0.0"  // 可选
    }
  ]
}
```

### `id` 是长期标识，不要随手改

App 用 `id` 认一个源。用户在这个源上设过的 UA、以及「当前选中了哪个源」都挂在它上面。
**改了 `id`，等于在用户那里删掉旧源、加一个新源**，这些设置会静默丢失。

特别地：**`id` 不要由 URL 派生**。这些上游源经常换域名（`sub.ottiptv.cc` 这类），
URL 一变 id 就变，用户设置全丢。id 应该是你手写的一个短名。

### 可选字段

| 字段 | 作用 |
|---|---|
| `userAgent` | 拉这个源的清单时用的 UA。部分订阅源不带指定 UA 会 403 |
| `enabled` | 设 `false` 可**临时下线**一个源而不删掉它（保留 id 与用户设置） |

## 凭证不要写进这个文件

`sources.json` 是要 push 到 GitHub 的。**任何带 `sign` / `auth_token` / `password`
之类的订阅链接都不要填真值**——公开仓库等于把这些凭证公开。

用 `{{参数名}}` 占位，真值存在 App 本地：

```json
"url": "https://live.ottiptv.cc/iptv.m3u?userid={{userid}}&sign={{sign}}&auth_token={{auth_token}}"
```

## 这个文件是怎么生成的

不要手改 `sources.json`——用脚本从设备上导出现有源，避免手抄错 URL：

```bash
cd ../android
python tools/export_sources_bundle.py --from-device -o ../sources-bundle/sources.json
```

脚本会**自动把疑似凭证的查询参数替换成占位符**，所以默认产出是安全可公开的。
`--keep-secrets` 会保留真值，**只在确认本仓库为私有时才用**。
