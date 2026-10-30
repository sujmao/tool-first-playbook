# 方法論：工具優先，不重複造輪子 / Methodology: Tool-First, Don't Reinvent the Wheel

## 核心問題 / The core problem

接到一個任務（例如「批量下載某平台內容」），第一反應往往是「我去查它的 API、自己寫個腳本」。但在很多場景下，這條路會讓你一頭撞進：反爬、簽名、風控、格式轉換、合併邏輯……而這些，都是成熟工具早就解決過的問題。

Given a task (e.g. "batch-download a platform's content"), the first instinct is often "I'll look up its API and write a script." But that path frequently leads straight into anti-bot measures, request signing, risk control, format conversion, muxing logic — all problems mature tools have already solved.

## 決策框架（四步）/ Decision framework (four steps)

### 1. 先盤點，再動手 / Inventory first, act second

動手前先問：目標對象是否已經提供——MCP server（越來越多的桌面應用內建）、CLI / 命令列工具、官方或第三方 SDK、現成的外掛 / 下載器 / 匯出功能。

Before acting, first ask whether the target already provides: an MCP server (more and more desktop apps ship one), a CLI, an official or third-party SDK, or an existing plugin / downloader / export feature.

如果答案是「有」，**優先驅動它**，而不是自己重新實現。

If the answer is "yes", drive it first — don't reimplement.

### 2. 分工：讓工具做難的，自己做薄的 / Split the work: the tool does the hard part, you do the thin part

| 難的部分（交給工具）/ Hard part (the tool) | 薄的部分（自己寫膠水）/ Thin part (your glue) |
|---|---|
| 登入 / Cookie / 鑑權 · Login / cookie / auth | 讀取結果、解析結構化輸出 · Read results, parse structured output |
| 反爬 / 簽名 / 風控 · Anti-bot / signing / risk control | 輪詢狀態、錯誤重試 · Poll status, retry on error |
| 分片下載、合併、轉碼 · Segmented download, mux, transcode | 重命名、統計、估算 · Rename, tally, estimate |
| 邊界條件（多 P、清晰度、會員）· Edge cases (multi-part, quality, members) | 生成報告 · Generate reports |

### 3. 只造「膠水」，不重造「核心」/ Build glue, not core

自己寫的代碼應該是一層薄薄的**編排層**：呼叫工具的介面 → 讀結果 → 做下一步。一旦發現自己開始重新實現「簽名算法」「協議解析」「字幕轉 ASS」「封面轉碼」，**停下來，回去找現成工具**。

Your code should be a thin orchestration layer: call the tool's interface → read the result → take the next step. The moment you find yourself reimplementing a signing algorithm, protocol parsing, subtitle-to-ASS conversion or cover transcoding, stop and go find the existing tool.

### 4. 判斷信號（該回頭找工具的時刻）/ Signals to turn back

- 你開始繞驗證碼 / 風控。You start working around CAPTCHAs / risk control.
- 你開始複製某個格式轉換的細節。You start reproducing the details of a format conversion.
- 你在重寫一個「別人明顯已經寫好」的功能。You're rewriting something clearly already written.
- 你在處理「登入態 / Cookie 過期」這類工具的內部狀態。You're managing a tool's internal state (login / cookie expiry).

任何一個信號出現，都先搜一遍：目標應用有沒有 MCP / CLI / 設定檔能直接達成。

When any signal appears, search first: does the target app have an MCP / CLI / config that achieves it directly?

## 本案例的具體應用 / How this case applied it

- **目標 / Goal**：批量下載 B 站空間全部投稿。Batch-download every upload in a Bilibili space.
- **直覺做法 / Naive path**：逆向 B 站 Web API → 撞上 wbi 簽名 + HTTP 412 風控 + `dm_*` 反爬。Reverse-engineer the web API → hit wbi signing + HTTP 412 risk control + `dm_*` anti-bot.
- **工具優先做法 / Tool-first path**：驅動已安裝的桌面應用 Bili23-Downloader 的內建 MCP——應用內部已經處理好登入、Cookie、反爬與合併，MCP 只暴露「解析 / 建立下載 / 查狀態」。Drive the installed Bili23-Downloader app's built-in MCP — the app already handles login, cookies, anti-bot and muxing; MCP just exposes "parse / create download / query status".
- **自己只寫 / All you write**：逐個 bvid 呼叫 MCP 的編排腳本 + 事後重命名 + 事前用匿名 API 估算大小。An orchestration script that calls MCP per-bvid, plus post-rename and an upfront size estimate via anonymous API.

結果：全程沒有逆向任何反爬協議，只寫了一層編排，卻完成了 40+ 檔案的批量下載、估價與重命名。

Result: without reverse-engineering any anti-bot protocol, a single orchestration layer completed the batch download, size estimate and rename of 40+ files.
