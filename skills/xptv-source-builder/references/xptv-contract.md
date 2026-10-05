# XPTV 接口约定

这是接口工作摘要，不是客户端兼容性保证。新源制作前核对原始文档：

- 指南：https://meteor-lemongrass-68b.notion.site/b60fcb3db53841229f9ec7352c5fda26
- 指南链接的开发者文档：https://gist.github.com/occupy-pluto/25222e89b8d0fc8409b79a377dda3f4c

摘要依据已分析的接口；重新访问失败不能写成“已核验最新文档”。

## 运行时

使用 `$fetch.get(url, {headers: ...})`，读取响应 `data`。接口示例使用 `argsify` 解析输入、`jsonify` 序列化输出；等效 JSON 解析和 `JSON.stringify` 需验证。`createCheerio()` 可用于文档支持的 HTML 解析。Node 的 fs、vm 等仅用于本地开发，不放入交付脚本。

## 数据传递

```text
getConfig.tabs[].ext
 → getCards(ext).list[].ext
 → getTracks(ext).list[].tracks[].ext
 → getPlayinfo(ext)
```

示例地址仅用于说明结构，实际源应使用已验证地址。各接口返回序列化后的 JSON。

`getLocalInfo()`：本地识别信息。

```js
{ ver: 1, name: "源名称", api: "csp_SourceName" }
```

`getConfig()`：分类入口。

```js
{ ver: 1, title: "源名称", site: "https://example.com",
  tabs: [{ name: "最近更新", ext: { id: "recent" } }] }
```

`getCards(ext)`：分页在 ext.page，筛选在 ext.filters。

```js
{ list: [{ vod_id: "42", vod_name: "作品标题", vod_pic: "图片URL",
           vod_remarks: "12集", ext: { id: "42" } }],
  filter: [{ key: "sort", name: "排序", init: "latest",
             value: [{ n: "最新", v: "latest" }] }] }
```

filter 仅在实际支持筛选时返回；key 对应后续 ext.filters[key]。缺省或非法值使用定义的默认值。

`getTracks(ext)`：播放分组和剧集。

```js
{ list: [{ title: "默认线路", tracks: [
  { name: "第1集", pan: "", ext: { id: "42", episode: 1 } }
] }] }
```

`getPlayinfo(ext)`：选中集的媒体 URL。headers 与 urls 数组对应。

```js
{ urls: ["https://media.example.com/episode-1.m3u8"],
  headers: [{ Referer: "https://example.com/watch/42/1" }] }
```

没有地址返回 urls: [] 并提供诊断，不编造可播放 URL。

`search(ext)`：通常读取 ext.text 和 ext.page，按卡片格式返回 {list: [...]}，不假定所有客户端都有其他关键词字段。

## 订阅 JSON

```json
{
  "sites": [{
    "name": "源名称",
    "type": 3,
    "api": "csp_SourceName",
    "ext": "https://你的托管地址/source.js"
  }]
}
```

api 与本地识别信息一致。type: 3 不会让 TVBox 或 ForwardWidget 自动兼容 XPTV，接口和运行时均需适配。

`$config_str` 可用于脚本配置，但不能假定某个客户端版本有对应编辑界面。源码可以提供非敏感默认配置，秘密不得随公开脚本发布。
