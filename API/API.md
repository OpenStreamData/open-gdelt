# 全球舆情检索 API 说明

**Base URL：** `https://gdeltapi.openstreamdata.top`

**认证：** 所有请求需携带 API Key（请求头 `X-API-Key: 你的密钥`）

**申请 Key：** 见网页 `/api-key.html`，或发邮件至 openstreamdata@163.com

---

## 接口

| 接口 | 说明 |
|------|------|
| `GET /v1/meta` | 查询数据范围、总量、主题选项 |
| `GET /v1/search` | 条件搜索（JSON 分页） |
| `GET /v1/export` | 条件导出（CSV，最多 1 万条） |

---

## GET /v1/meta

无参数。

```bash
curl -H "X-API-Key: YOUR_KEY" \
  "https://gdeltapi.openstreamdata.top/v1/meta"
```

返回示例：

```json
{
  "total": 1461943,
  "min_date": "2026-06-17",
  "max_date": "2026-06-23",
  "min_datetime": "2026-06-17 00:00:00",
  "max_datetime": "2026-06-23 15:12:00",
  "last_update_label": "2026-06-23 15:24",
  "theme_options": [{"value": "MILITARY", "label": "军事 · MILITARY"}],
  "fts_ready": true
}
```

---

## GET /v1/search

### 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `date_from` | 是 | 开始日期，如 `2026-06-20` |
| `date_to` | 是 | 结束日期，如 `2026-06-23` |
| `keyword` | 否 | 关键词 |
| `theme` | 否 | GKG 主题筛选，见下方 [主题说明](#主题theme说明) |
| `page` | 否 | 页码，默认 `1` |
| `limit` | 否 | 每页条数，默认 `30`，最大 `100` |
| `datetime_from` | 否 | 开始时间，与 `datetime_to` 成对使用 |
| `datetime_to` | 否 | 结束时间，与 `datetime_from` 成对使用 |

**时间格式（任选）：** `2026-06-23T14:00:00`、`2026-06-23 14:00:00`、`20260623140000`

### 搜索形式

**1. 仅按日期**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23
```

**2. 日期 + 关键词（单词）**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&keyword=china
```

**3. 日期 + 关键词（多词，默认连续短语）**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&keyword=rare earth
```

匹配标题/实体/URL 中含 **连续** `rare earth`（如稀土），不会把 `Rare` 与 `Earth` 拆成两处单独命中。

**4. 日期 + 关键词（多词分别包含，用 AND 或 +）**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&keyword=rare AND earth
/v1/search?date_from=2026-06-20&date_to=2026-06-23&keyword=rare+earth
```

**5. 日期 + 主题**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&theme=MILITARY
```

**6. 日期 + 关键词 + 主题（组合）**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&keyword=china&theme=SANCTION
```

**7. 分页**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&page=2&limit=50
```

**8. 精确时间范围（如最近 30 分钟）**

```
/v1/search?date_from=2026-06-23&date_to=2026-06-23&datetime_from=2026-06-23T14:00:00&datetime_to=2026-06-23T14:30:00
```

> 仅返回 `record_time` 落在该区间内的记录。旧数据无时间戳时不会命中时间筛选；新入库数据均含完整时间。

**9. 时间范围 + 关键词**

```
/v1/search?date_from=2026-06-23&date_to=2026-06-23&datetime_from=2026-06-23T14:00:00&datetime_to=2026-06-23T14:30:00&keyword=china
```

> 关键词匹配：标题、实体/人名、来源 URL。中文需 URL 编码。

### 返回字段（items 每条）

| 字段 | 说明 |
|------|------|
| `date_label` | 日期 |
| `time_label` | 时间（`YYYY-MM-DD HH:MM:SS`） |
| `record_time` | 原始时间戳（`YYYYMMDDHHMMSS`） |
| `page_title` | 标题 |
| `document_identifier` | 来源 URL |
| `themes` | 主题 | GKG 原始主题串（V2Themes），见 [主题说明](#主题theme说明) |
| `all_names` | 实体/人名 |
| `organizations` | 组织 |
| `locations` | 地点 |
| `tone` | 语调（负偏消极，正偏积极，0 中性） |

### 示例

```bash
curl -G -H "X-API-Key: YOUR_KEY" \
  --data-urlencode "date_from=2026-06-20" \
  --data-urlencode "date_to=2026-06-23" \
  --data-urlencode "keyword=china" \
  --data-urlencode "theme=MILITARY" \
  "https://gdeltapi.openstreamdata.top/v1/search"
```

---

## GET /v1/export

参数与 `/v1/search` 相同（无 `page`、`limit`），返回 CSV 文件。

```bash
curl -L -H "X-API-Key: YOUR_KEY" \
  -o export.csv \
  "https://gdeltapi.openstreamdata.top/v1/export?date_from=2026-06-20&date_to=2026-06-23&keyword=china"
```

CSV 列：日期、时间、标题、来源URL、实体/人名、组织、地点、主题、语调

---

## 主题（theme）说明

### 数据来源

每条记录的 `themes` 字段来自 GDELT GKG 2.0 的 **V2Themes** 列，入库时**原样保存**，不做翻译或裁剪。

**原始格式：**

```
主题代码,字符偏移;主题代码,字符偏移;...
```

**示例：**

```
MILITARY,812;ARMEDCONFLICT,812;GOV_POLITICS,1204;ECON_TRADE,980
```

| 部分 | 含义 |
|------|------|
| `MILITARY` | GDELT 主题代码 |
| `812` | 该主题在文章文本中首次出现的字符位置（偏移量） |
| `;` | 多条主题之间的分隔符 |

一篇文章通常有**多个**主题标签；返回结果里的 `themes` 是完整列表，不是单个值。

---

### `theme` 参数怎么筛

`/v1/search` 和 `/v1/export` 的 `theme` 参数对 `themes` 字段做 **子串匹配**（相当于 SQL `LIKE '%你填的值%'`）。

| 规则 | 说明 |
|------|------|
| 不传或传空 | 不限制主题 |
| 一次只能传 **一个** `theme` | 不支持多个 theme 参数组合 |
| 匹配方式 | 只要 `themes` 里**任意一段**含有该子串即命中 |
| 与关键词 | 可与 `keyword` 同时使用（先按日期/时间，再主题，再关键词，逻辑 AND） |

**示例：**

| 请求参数 | 能匹配到的 themes 示例 |
|----------|------------------------|
| `theme=MILITARY` | `MILITARY,800`、`USPEC_MILITARY,400`（含 MILITARY 子串即可） |
| `theme=SANCTION` | `SANCTION,600`、`ECON_SANCTION,900` |
| `theme=ECON_` | `ECON_STOCKMARKET,100`、`ECON_INFLATION,200`、`ECON_TRADE,300` 等所有 `ECON_` 开头代码 |
| `theme=GOV_` | `GOV_POLITICS,500`、`GOV_ELECTION,700` 等 |
| `theme=PROTEST` | `PROTEST,300`（**不会**匹配 `PROTESTER`，除非 themes 里确实出现 PROTEST 子串） |

带 **末尾下划线 `_`** 的预设值（如 `ECON_`、`GOV_`）适合筛**一整类** GDELT 主题；不带下划线的多为**精确或较窄**代码。

---

### 系统预设主题（`/v1/meta` → `theme_options`）

Web 下拉框与 API 文档使用同一套预设，定义在服务端 `config.py`。调用 `/v1/meta` 可获取最新列表。

| `theme` 参数值 | 中文说明 | 典型用途 |
|----------------|----------|----------|
| *(不传)* | 全部主题 | 不做主题限制 |
| `MILITARY` | 军事 | 军队、国防、军事行动相关 |
| `TERROR` | 恐怖 | 恐怖主义、极端暴力 |
| `EXPORT` | 出口 | 出口管制、贸易出口 |
| `SANCTION` | 制裁 | 经济/政治制裁 |
| `ARMEDCONFLICT` | 武装冲突 | 战争、交火、冲突升级 |
| `PROTEST` | 抗议 | 示威、罢工、社会运动 |
| `ECON_` | 经济（前缀） | 宏观经济、市场、通胀、贸易等 `ECON_*` 类 |
| `GOV_` | 政府（前缀） | 政治、选举、政策等 `GOV_*` 类 |
| `CYBER_` | 网络（前缀） | 网络安全、黑客、数字基础设施等 |
| `USPEC_` | 美国特定（前缀） | 美国国内政治/社会相关 `USPEC_*` 类 |
| `WB_` | 世界银行（前缀） | 世界银行及发展金融相关 |
| `CRISISLEX` | 危机 | 危机应对、灾害、紧急状态 |
| `KILL` | 杀戮 | 凶杀、死亡、暴力致死 |
| `REFUGEES` | 难民 | 难民、流离失所、边境人道危机 |

> 上表为系统**推荐快捷项**，并非 GDELT 全部主题。GDELT 官方有数千个主题代码；只要知道代码或前缀，都可以直接填入 `theme` 参数尝试匹配。

---

### 请求示例

**按军事主题：**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&theme=MILITARY
```

**经济类（所有 ECON_ 开头）：**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&theme=ECON_
```

**制裁 + 关键词：**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&theme=SANCTION&keyword=china
```

**自定义 GDELT 代码（预设表里没有也可以）：**

```
/v1/search?date_from=2026-06-20&date_to=2026-06-23&theme=ENV_MINING
```

---

### 使用建议

1. **不确定用什么代码**：先不带 `theme` 搜索，看结果里 `themes` 字段有哪些代码，再拿其中一段作下次筛选。
2. **想覆盖一大类**：优先用带 `_` 的前缀项（`ECON_`、`GOV_`、`CYBER_`）。
3. **主题 + 关键词**：主题缩小领域，关键词缩小实体/事件，组合更精准。
4. **不要依赖 offset 数字**：`,812` 这类偏移量仅作 GDELT 原始信息，查询时只用主题代码部分。

**GDELT 官方主题参考：** [GDELT GKG Themes 说明](http://data.gdeltproject.org/api/v2/guides/LOOKUP-GKGTHEMES.TXT)

---

## 限流

- 每个 API Key：**3 秒 1 次**
- 超限返回 HTTP **429**，请稍后重试

---

## 错误说明

| 情况 | 处理 |
|------|------|
| 缺少日期 | 返回 `"error": "请选择开始日期和结束日期"` |
| 时间格式错误 | 返回 `"error": "datetime_from 格式无效..."` |
| Key 无效 | HTTP 401 |
| 请求过快 | HTTP 429，间隔 3 秒再试 |

---

## 多语言调用模板

将 `YOUR_KEY` 替换为你的 API Key。每次请求间隔至少 **3 秒**。

### Python

```python
import requests
import time

BASE = "https://gdeltapi.openstreamdata.top"
API_KEY = "YOUR_KEY"
HEADERS = {"X-API-Key": API_KEY}


def meta():
    r = requests.get(f"{BASE}/v1/meta", headers=HEADERS, timeout=60)
    r.raise_for_status()
    return r.json()


def search(date_from, date_to, keyword="", theme="", page=1, limit=30,
           datetime_from="", datetime_to=""):
    params = {
        "date_from": date_from,
        "date_to": date_to,
        "page": page,
        "limit": limit,
    }
    if keyword:
        params["keyword"] = keyword
    if theme:
        params["theme"] = theme
    if datetime_from:
        params["datetime_from"] = datetime_from
    if datetime_to:
        params["datetime_to"] = datetime_to
    r = requests.get(f"{BASE}/v1/search", headers=HEADERS, params=params, timeout=120)
    r.raise_for_status()
    data = r.json()
    if data.get("error"):
        raise RuntimeError(data["error"])
    return data


def export_csv(date_from, date_to, keyword="", theme="", out_path="export.csv"):
    params = {"date_from": date_from, "date_to": date_to}
    if keyword:
        params["keyword"] = keyword
    if theme:
        params["theme"] = theme
    r = requests.get(f"{BASE}/v1/export", headers=HEADERS, params=params, timeout=300)
    r.raise_for_status()
    with open(out_path, "wb") as f:
        f.write(r.content)


if __name__ == "__main__":
    info = meta()
    print(info["min_date"], info["max_date"], info["total"])

    time.sleep(3)  # 限流：每 Key 3 秒 1 次

    result = search("2026-06-20", "2026-06-23", keyword="rare earth", theme="MILITARY")
    for item in result["items"]:
        print(item["date_label"], item["page_title"], item["tone"])
```

依赖：`pip install requests`

---

### JavaScript（Node.js / 浏览器 fetch）

```javascript
const BASE = "https://gdeltapi.openstreamdata.top";
const API_KEY = "YOUR_KEY";
const headers = { "X-API-Key": API_KEY };

async function meta() {
  const res = await fetch(`${BASE}/v1/meta`, { headers });
  const data = await res.json();
  if (!res.ok) throw new Error(data.error || res.statusText);
  return data;
}

async function search({ dateFrom, dateTo, keyword = "", theme = "", page = 1, limit = 30 }) {
  const params = new URLSearchParams({
    date_from: dateFrom,
    date_to: dateTo,
    page: String(page),
    limit: String(limit),
  });
  if (keyword) params.set("keyword", keyword);
  if (theme) params.set("theme", theme);

  const res = await fetch(`${BASE}/v1/search?${params}`, { headers });
  const data = await res.json();
  if (!res.ok || data.error) throw new Error(data.error || res.statusText);
  return data;
}

async function exportCsv({ dateFrom, dateTo, keyword = "", theme = "" }) {
  const params = new URLSearchParams({ date_from: dateFrom, date_to: dateTo });
  if (keyword) params.set("keyword", keyword);
  if (theme) params.set("theme", theme);

  const res = await fetch(`${BASE}/v1/export?${params}`, { headers });
  if (!res.ok) throw new Error(await res.text());
  return res.blob(); // 浏览器可 URL.createObjectURL(blob) 下载
}

// 示例
(async () => {
  const info = await meta();
  console.log(info.min_date, info.max_date, info.total);

  await new Promise((r) => setTimeout(r, 3000));

  const result = await search({
    dateFrom: "2026-06-20",
    dateTo: "2026-06-23",
    keyword: "rare earth",
    theme: "MILITARY",
  });
  result.items.forEach((item) => console.log(item.date_label, item.page_title));
})();
```

---

### C#

```csharp
using System.Net.Http.Headers;
using System.Text.Json;

const string Base = "https://gdeltapi.openstreamdata.top";
const string ApiKey = "YOUR_KEY";

using var http = new HttpClient();
http.DefaultRequestHeaders.Add("X-API-Key", ApiKey);

// meta
var metaJson = await http.GetStringAsync($"{Base}/v1/meta");
using var metaDoc = JsonDocument.Parse(metaJson);
var root = metaDoc.RootElement;
Console.WriteLine($"{root.GetProperty("min_date")} ~ {root.GetProperty("max_date")}");

await Task.Delay(3000); // 限流

// search（多词 keyword 用 Query 自动编码）
var query = new Dictionary<string, string>
{
    ["date_from"] = "2026-06-20",
    ["date_to"] = "2026-06-23",
    ["keyword"] = "rare earth",
    ["theme"] = "MILITARY",
    ["page"] = "1",
    ["limit"] = "30",
};
var qs = string.Join("&", query.Select(kv =>
    $"{Uri.EscapeDataString(kv.Key)}={Uri.EscapeDataString(kv.Value)}"));
var searchJson = await http.GetStringAsync($"{Base}/v1/search?{qs}");
using var searchDoc = JsonDocument.Parse(searchJson);
foreach (var item in searchDoc.RootElement.GetProperty("items").EnumerateArray())
{
    Console.WriteLine($"{item.GetProperty("date_label")} {item.GetProperty("page_title")}");
}

// export
await Task.Delay(3000);
var csvBytes = await http.GetByteArrayAsync(
    $"{Base}/v1/export?date_from=2026-06-20&date_to=2026-06-23&keyword={Uri.EscapeDataString("china")}");
await File.WriteAllBytesAsync("export.csv", csvBytes);
```

---

### Java

```java
import java.net.URI;
import java.net.URLEncoder;
import java.net.http.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;

public class GdeltApi {
    static final String BASE = "https://gdeltapi.openstreamdata.top";
    static final String API_KEY = "YOUR_KEY";

    static HttpClient client = HttpClient.newHttpClient();

    static HttpRequest req(String url) {
        return HttpRequest.newBuilder(URI.create(url))
            .header("X-API-Key", API_KEY)
            .GET()
            .build();
    }

    public static void main(String[] args) throws Exception {
        var meta = client.send(req(BASE + "/v1/meta"), HttpResponse.BodyHandlers.ofString());
        System.out.println(meta.body());

        Thread.sleep(3000);

        String qs = "date_from=2026-06-20&date_to=2026-06-23"
            + "&keyword=" + URLEncoder.encode("rare earth", StandardCharsets.UTF_8)
            + "&theme=MILITARY&page=1&limit=30";
        var search = client.send(req(BASE + "/v1/search?" + qs), HttpResponse.BodyHandlers.ofString());
        System.out.println(search.body());
    }
}
```

---

### Go

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"net/url"
	"os"
	"time"
)

const base = "https://gdeltapi.openstreamdata.top"
const apiKey = "YOUR_KEY"

func get(path string, params url.Values) ([]byte, error) {
	u, _ := url.Parse(base + path)
	if params != nil {
		u.RawQuery = params.Encode()
	}
	req, _ := http.NewRequest(http.MethodGet, u.String(), nil)
	req.Header.Set("X-API-Key", apiKey)
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	return io.ReadAll(resp.Body)
}

func main() {
	body, _ := get("/v1/meta", nil)
	fmt.Println(string(body))

	time.Sleep(3 * time.Second)

	params := url.Values{
		"date_from": {"2026-06-20"},
		"date_to":   {"2026-06-23"},
		"keyword":   {"rare earth"},
		"theme":     {"MILITARY"},
		"page":      {"1"},
		"limit":     {"30"},
	}
	body, _ = get("/v1/search", params)
	fmt.Println(string(body))

	time.Sleep(3 * time.Second)

	csv, _ := get("/v1/export", url.Values{
		"date_from": {"2026-06-20"},
		"date_to":   {"2026-06-23"},
		"keyword":   {"china"},
	})
	os.WriteFile("export.csv", csv, 0644)
}
```

---

### PHP

```php
<?php
$base = "https://gdeltapi.openstreamdata.top";
$apiKey = "YOUR_KEY";

function gdeltGet(string $path, array $params = []): string {
    global $base, $apiKey;
    $url = $base . $path;
    if ($params) {
        $url .= "?" . http_build_query($params);
    }
    $ctx = stream_context_create([
        "http" => [
            "method" => "GET",
            "header" => "X-API-Key: $apiKey\r\n",
            "timeout" => 120,
        ],
    ]);
    return file_get_contents($url, false, $ctx);
}

$meta = json_decode(gdeltGet("/v1/meta"), true);
echo $meta["min_date"], " ~ ", $meta["max_date"], "\n";

sleep(3);

$result = json_decode(gdeltGet("/v1/search", [
    "date_from" => "2026-06-20",
    "date_to"   => "2026-06-23",
    "keyword"   => "rare earth",
    "theme"     => "MILITARY",
    "page"      => 1,
    "limit"     => 30,
]), true);

foreach ($result["items"] as $item) {
    echo $item["date_label"], " ", $item["page_title"], "\n";
}
```

---

### cURL

```bash
# meta
curl -s -H "X-API-Key: YOUR_KEY" \
  "https://gdeltapi.openstreamdata.top/v1/meta"

# search（推荐 -G --data-urlencode 自动处理空格和中文）
curl -s -G -H "X-API-Key: YOUR_KEY" \
  --data-urlencode "date_from=2026-06-20" \
  --data-urlencode "date_to=2026-06-23" \
  --data-urlencode "keyword=rare earth" \
  --data-urlencode "theme=MILITARY" \
  "https://gdeltapi.openstreamdata.top/v1/search"

# export
curl -L -H "X-API-Key: YOUR_KEY" \
  -o export.csv \
  "https://gdeltapi.openstreamdata.top/v1/export?date_from=2026-06-20&date_to=2026-06-23&keyword=china"
```
