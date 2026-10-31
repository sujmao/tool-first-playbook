# 案例：驅動 Bili23-Downloader 的 MCP 完成批量下載 / Case Study: Driving Bili23-Downloader's MCP for Batch Downloads

## 背景與目標 / Background & goal

- **目標 / Goal**：下載某個 B 站空間（`space.bilibili.com/<UID>`）的全部投稿影片，要求「下載前先估算總大小」「檔案名包含發布時間 + 標題」。
  Download every upload in a Bilibili space (`space.bilibili.com/<UID>`), with "estimate total size first" and "filename contains publish time + title".
- **可用工具 / Available tool**：已安裝的開源桌面應用 [Bili23-Downloader](https://github.com/ScottSloan/Bili23-Downloader)，它內建一個 MCP server。
  The installed open-source desktop app Bili23-Downloader, which ships a built-in MCP server.

## 工具盤點 / Tool inventory

Bili23-Downloader 的 MCP：

- **傳輸 / Transport**：HTTP，`http://127.0.0.1:<port>/mcp`，Bearer Token 鑑權，JSON-RPC 2.0。
- **開啟方式 / How to enable**：改應用設定檔（應用標準設定目錄下的 `config.json`）——`MCP.mcp_enabled = true`，設定 `mcp_port` 與 `mcp_token`，重啟應用。
  Edit the app config (`config.json` in the app's standard settings dir): set `MCP.mcp_enabled = true`, set `mcp_port` and `mcp_token`, then restart.
- **連接 / 斷聯 / Connect & disconnect**：見 [MCP-CONNECT.md](MCP-CONNECT.md)——用 HTTP 模式，開應用即連、關應用即斷，避免 Claude CLI 連帶啟動 Bili23。See MCP-CONNECT.md: HTTP mode — open the app to connect, quit to disconnect, so Claude CLI never auto-launches Bili23.
- **工具清單 / Tools**：`parse_url` / `get_episodes` / `create_download` / `list_tasks` / `get_task_status` / `get_login_status` / `pause_task` / `resume_task` / `cancel_task`。

## 為什麼「不直接打 B 站 API」/ Why not hit the Bilibili API directly

初步嘗試證明裸打 API 很痛苦：

An initial attempt showed hitting the raw API is painful:

- 空間列表 `x/space/wbi/arc/search` 需要 wbi 簽名，匿名請求又撞 HTTP 412，要求 `dm_*` 反爬參數。
  The space-list endpoint `x/space/wbi/arc/search` requires wbi signing, and anonymous requests hit HTTP 412 demanding `dm_*` anti-bot params.
- 而應用的 MCP 內部已經持有登入態與反爬處理，等於把最難的部分免費繼承了。
  The app's MCP already holds the login state and anti-bot handling internally — the hardest part comes for free.

於是分工變成：

So the work split became:

| 職責 / Responsibility | 用甚麼 / Tool |
|---|---|
| 登入 / 反爬 / 下載 / 合併 · Login / anti-bot / download / mux | 應用的 MCP · the app's MCP |
| 拿發布時間、檔案大小（做估算）· Publish time & size (for the estimate) | 匿名 `x/web-interface/view` + `x/player/playurl` · anonymous API |
| 重命名 / 統計 / 報告 · Rename / tally / report | 自寫編排腳本（薄膠水）· a thin orchestration script |

## 全流程 / Full workflow

1. **列舉 / Enumerate**：`get_episodes` 拿到空間全部條目（bvid / 標題）。Get every space item (bvid / title).
2. **估算大小（下載前）/ Estimate size (before downloading)**：匿名呼叫 `x/player/playurl?fnval=0` 取 720p 的 `durl[].size` 累加，作為**下界**；再依「登入後實際畫質（1080p）」給出一個**區間**，而不是單點精確值。
   Sum the anonymous 720p `durl[].size` as a lower bound, then give a range based on the actual logged-in quality (1080p) — not a single point value.
3. **逐個下載 / Download one by one**：空間條目帶 `NEED_PARSE_BIT`，不能直接下，必須 `parse_url(bvid)` → 取 `episode_id` → `create_download({episode_ids:[id]})`。
   Space items carry `NEED_PARSE_BIT` and can't be downloaded directly: `parse_url(bvid)` → take `episode_id` → `create_download({episode_ids:[id]})`.
4. **輪詢 / Poll**：`get_task_status({task_id})` 直到 `status == completed`。Poll until `status == completed`.
5. **重命名 / Rename**：外部腳本把產物改成 `{發布日期}_{標題}.mp4`（MCP 無法改輸出目錄 / 命名，命名規則一律跟應用設定走）。
   Rename outputs to `{publish_date}_{title}.mp4` (MCP can't change the output dir / naming; naming always follows the app config).

## 成果（匿名化）/ Outcome (anonymized)

- 全部投稿下載完成、統一重命名。All uploads downloaded and uniformly renamed.
- 估算區間與實際落地大小相符（下界為 720p，實際因登入按 1080p 下載而落在區間上沿）。The estimate range matched the real size (lower bound at 720p; actual landed at the top of the range since a logged-in session downloads 1080p).
- 全程未逆向任何反爬協議，僅寫了一層編排 + 幾個純取數的匿名 API 呼叫。No anti-bot protocol was reverse-engineered — just one orchestration layer plus a few read-only anonymous API calls.
