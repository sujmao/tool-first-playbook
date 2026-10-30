# 案例：驅動 Bili23-Downloader 的 MCP 完成批量下載

## 背景與目標

- **目標**：下載某個 B 站空間（`space.bilibili.com/<UID>`）的全部投稿影片，
  要求「下載前先估算總大小」「檔案名包含發布時間 + 標題」。
- **可用工具**：已安裝的開源桌面應用
  [Bili23-Downloader](https://github.com/ScottSloan/Bili23-Downloader)，它內建一個 MCP server。

## 工具盤點

Bili23-Downloader 的 MCP：

- **傳輸**：HTTP，`http://127.0.0.1:<port>/mcp`，Bearer Token 鑑權，JSON-RPC 2.0。
- **開啟方式**：改應用設定檔（應用標準設定目錄下的 `config.json`）——
  `MCP.mcp_enabled = true`，設定 `mcp_port` 與 `mcp_token`，重啟應用。
- **工具清單**：`parse_url` / `get_episodes` / `create_download` / `list_tasks` /
  `get_task_status` / `get_login_status` / `pause_task` / `resume_task` / `cancel_task`。

## 為什麼「不直接打 B 站 API」

初步嘗試證明裸打 API 很痛苦：

- 空間列表 `x/space/wbi/arc/search` 需要 wbi 簽名，匿名請求又撞 HTTP 412，
  要求 `dm_*` 反爬參數。
- 而應用的 MCP 內部已經持有登入態與反爬處理，等於把最難的部分免費繼承了。

於是分工變成：

| 職責 | 用甚麼 |
|---|---|
| 登入 / 反爬 / 下載 / 合併 | 應用的 MCP |
| 拿發布時間、檔案大小（做估算） | 匿名 `x/web-interface/view` + `x/player/playurl` |
| 重命名 / 統計 / 報告 | 自寫編排腳本（薄膠水） |

## 全流程

1. **列舉**：`get_episodes` 拿到空間全部條目（bvid / 標題）。
2. **估算大小**（下載前）：匿名呼叫 `x/player/playurl?fnval=0` 取 720p 的
   `durl[].size` 累加，作為**下界**；再依「登入後實際畫質（1080p）」給出一個**區間**，
   而不是單點精確值。
3. **逐個下載**：空間條目帶 `NEED_PARSE_BIT`，不能直接下，必須
   `parse_url(bvid)` → 取 `episode_id` → `create_download({episode_ids:[id]})`。
4. **輪詢**：`get_task_status({task_id})` 直到 `status == completed`。
5. **重命名**：外部腳本把產物改成 `{發布日期}_{標題}.mp4`（MCP 無法改輸出目錄 / 命名，
   命名規則一律跟應用設定走）。

## 成果（匿名化）

- 全部投稿下載完成、統一重命名。
- 估算區間與實際落地大小相符（下界為 720p，實際因登入按 1080p 下載而落在區間上沿）。
- 全程未逆向任何反爬協議，僅寫了一層編排 + 幾個純取數的匿名 API 呼叫。
