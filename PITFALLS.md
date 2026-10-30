# 踩坑清單 / Pitfall List

> 每一條都附「現象 → 原因 → 避開方式」，方便直接查表。這些坑來自真實的一次 Bili23-Downloader MCP 批量下載任務。
> Each entry follows "Symptom → Cause → How to avoid", for quick lookup. These come from a real Bili23-Downloader MCP batch-download run.

## 1. 空間條目不能直接下載 / Space items can't be downloaded directly

- **現象 / Symptom**：對空間 URL `parse_url` 後，直接 `create_download` 被拒。After `parse_url` on a space URL, `create_download` is rejected directly.
- **原因 / Cause**：空間列表條目帶 `NEED_PARSE_BIT`，要求先單獨解析。Space-list items carry `NEED_PARSE_BIT` and must be parsed individually first.
- **避開 / Avoid**：**逐個 bvid** 走 `parse_url(bvid)` → 取 `episode_id` → `create_download`。Parse each bvid: `parse_url(bvid)` → `episode_id` → `create_download`.

## 2. MCP 無法改輸出目錄 / 檔案命名 / MCP can't change output dir or filename

- **現象 / Symptom**：想在 MCP 呼叫裡指定下載路徑或命名規則，發現沒這參數。No parameter to set the download path or naming rule in an MCP call.
- **原因 / Cause**：MCP 只跟應用設定檔走，不暴露這兩個選項。MCP only follows the app config and doesn't expose these.
- **避開 / Avoid**：下載完成後**外部腳本 `rename`**；或動手前先改應用設定檔的 `download_path` / 命名規則。Rename with an external script after download, or edit `download_path` / naming in the app config first.

## 3. 不要把會話 Cookie 寫進檔案 / Don't write session cookies to a file

- **現象 / Symptom**：想把自己登入態的 SESSDATA 存成檔案給腳本用。Wanting to save the logged-in SESSDATA to a file for a script.
- **原因 / Cause**：這類憑證落盤等於把帳號交出去，且其實**不必要**。Persisting such credentials is handing over the account, and it's unnecessary.
- **避開 / Avoid**：要麼讓應用 MCP 內部處理鑑權；要麼用匿名介面時，先從 `x/frontend/finger/spi` 拿**全新** `buvid3` / `buvid4`，用完即棄。Let the app's MCP handle auth; or fetch fresh anonymous `buvid3`/`buvid4` from `x/frontend/finger/spi` and discard them.

## 4. 指紋介面回傳的欄位名是 `b_3` / `b_4` / The fingerprint endpoint returns `b_3` / `b_4`

- **現象 / Symptom**：抓 `data.b3` 報 `KeyError`。Reading `data.b3` raises `KeyError`.
- **原因 / Cause**：實際欄位是 `b_3`、`b_4`（帶下劃線），不是 `b3`。The fields are `b_3`, `b_4` (with underscores).
- **避開 / Avoid**：直接讀 `data["b_3"]` / `data["b_4"]`。

## 5. 彈幕介面回傳「亂碼」/ The danmaku endpoint returns "garbage"

- **現象 / Symptom**：`x/v1/dm/list.so?oid=<cid>` 回傳內容解不出 XML。The response won't decode as XML.
- **原因 / Cause**：回應是 **raw deflate**（`Content-Encoding: deflate` 但沒有 zlib 頭）。It's raw deflate (`Content-Encoding: deflate` but no zlib header).
- **避開 / Avoid**：`zlib.decompress(raw, -15)` 再解碼；不要用 `gzip.decompress`。Use `zlib.decompress(raw, -15)`, not `gzip.decompress`.

## 6. 封面圖要 PNG 直接改 URL / Get a PNG cover by tweaking the URL

- **現象 / Symptom**：封面 URL 是 `.jpg`，想存 `.png` 還得自己轉碼。The cover URL is `.jpg`; saving as `.png` seems to need transcoding.
- **原因 / Cause**：B 站封面 URL 後綴 `@.png` 會直接回傳 PNG 位元組。Appending `@.png` makes Bilibili return PNG bytes directly.
- **避開 / Avoid**：`"{cover_url}@.png"` 直接下載即為 PNG。Download `"{cover_url}@.png"` — it's already PNG.

## 7. `create_download` 偶發回傳空 / `create_download` occasionally returns empty

- **現象 / Symptom**：偶爾 `create_download` 成功回應但 `structuredContent` 為空（「no tasks」）。Sometimes the response is successful but `structuredContent` is empty ("no tasks").
- **原因 / Cause**：多為應用下載佇列繁忙造成的瞬時失敗。Usually a transient failure from a busy download queue.
- **避開 / Avoid**：**讀完整回傳**（含 `content[].text` 的 `isError` 資訊）並**重試**，不要只看 `structuredContent`。Read the full response (including `isError` in `content[].text`) and retry; don't look only at `structuredContent`.

## 8. 進度 100% 卻 `status == failed` / 100% progress but `status == failed`

- **現象 / Symptom**：任務下到 100%、大小也對，狀態卻失敗，留下 `.m4s` / `.m4a` 分片。A task hits 100% with the right size yet fails, leaving `.m4s` / `.m4a` segments.
- **原因 / Cause**：**合併步驟失敗**（下載本身成功了）。The mux step failed (the download itself succeeded).
- **避開 / Avoid**：用應用內建 ffmpeg `ffmpeg -i video.m4s -i audio.m4a -c copy out.mp4` 直接補救，不必重新下載。Repair with the bundled ffmpeg `ffmpeg -i video.m4s -i audio.m4a -c copy out.mp4` — no need to re-download.

## 9. 側車文件（彈幕 / 封面 / 元數據）跟著設定走 / Sidecar files (danmaku / cover / metadata) follow the app settings

- **現象 / Symptom**：下完影片，目錄裡還多出 `.ass` / `.json` / `.png`。After downloading videos, `.ass` / `.json` / `.png` files also appear.
- **原因 / Cause**：應用的「彈幕 / 封面 / 元數據」選項預設開啟。The app's danmaku / cover / metadata options are on by default.
- **避開 / Avoid**：動手前先確認是否要把側車文件納入交付，並對它們也做統一的命名 / 清理。Decide up front whether sidecars are part of the deliverable, and name/clean them consistently too.

## 10. 大小估算要給「區間」而非「單點」/ Estimate a range, not a point value

- **現象 / Symptom**：用匿名 720p 估出來的值，跟登入後 1080p 實際差很多。The anonymous 720p estimate differs a lot from the logged-in 1080p reality.
- **原因 / Cause**：匿名只能拿低畫質；登入後畫質更高、體積更大。Anonymous access gets lower quality; a logged-in session downloads higher quality and larger files.
- **避開 / Avoid**：用 720p 當**下界**，再依目標畫質給出**上界**，並明說這是估算區間。Use 720p as a lower bound, add an upper bound for the target quality, and state clearly that it's a range.

## 11. 併發下載時更容易觸發瞬時失敗 / Concurrency makes transient failures more likely

- **現象 / Symptom**：多個任務同時排隊時，某些 `create_download` 空回傳、狀態異常的機率變高。With many queued tasks, empty `create_download` responses and odd statuses become more frequent.
- **原因 / Cause**：應用的下載佇列與資料庫落盤在併發下更易出現競態。The app's download queue and DB writes race more under concurrency.
- **避開 / Avoid**：編排腳本**逐個或小批量**下載，並對每個失敗項做重試，而不是一口氣全塞進去。Download one at a time or in small batches, retry each failure, don't enqueue everything at once.
