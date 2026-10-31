# 連接 / 斷聯 Bili23 的 MCP / Connecting & disconnecting the Bili23 MCP

> 補上 tool-first 流程裡缺的「最後一環」：如何**靈活**地連接 / 斷開 Bili23 的 MCP，而不是每次啟動 Claude CLI 都連帶把 Bili23 桌面應用一併拉起。
> This closes the last link in the tool-first workflow: how to **on-demand** connect / disconnect the Bili23 MCP, instead of having Claude CLI pull up the Bili23 desktop app every time it starts.

## 問題 / The problem

若把 Bili23 的 MCP 用 **stdio** 方式掛進 Claude Code（`command: Bili23.exe --mcp-stdio`），每次啟動 Claude CLI 都會**連帶啟動**整個 Bili23 桌面應用——只是想問點別的事，也被迫把下載器打開。

If you register Bili23's MCP as **stdio** (`command: Bili23.exe --mcp-stdio`), every Claude CLI start also **launches** the whole Bili23 desktop app — you're forced to open the downloader even when you just want to do something else.

## 解法：改用 HTTP 模式 / The fix: use HTTP mode

Bili23 的 MCP 本身就支援 HTTP（`http://127.0.0.1:<port>/mcp`，Bearer Token 鑑權）。改用 HTTP 後，**應用開不開由你決定**：

Bili23's MCP natively speaks HTTP (`http://127.0.0.1:<port>/mcp`, Bearer Token auth). With HTTP mode, **you decide whether the app runs**:

- **連接 / Connect**：先打開 Bili23 應用，再啟動 Claude CLI。Open the Bili23 app first, then start Claude CLI.
- **斷聯 / Disconnect**：直接關掉 Bili23 應用即可——應用不開，MCP 端點自然不回應。Just quit Bili23 — with the app closed, the MCP endpoint simply isn't there.
- **完全不啟動 / Stay off**：不開應用，Claude CLI 照常啟動，只是 bili23 工具不在。Don't open the app and Claude CLI starts normally without the bili23 tools.

### 1. 應用端：開啟 HTTP MCP / App side: enable HTTP MCP

在應用設定檔（Windows：`%APPDATA%\Bili23 Downloader\config.json`）確認 MCP 已啟用：

In the app config (Windows: `%APPDATA%\Bili23 Downloader\config.json`), make sure MCP is enabled:

```json
{
  "MCP": {
    "mcp_enabled": true,
    "mcp_port": 23330,
    "mcp_token": "<YOUR_MCP_TOKEN>"
  }
}
```

改完**重啟應用**。Restart the app after editing.

### 2. Claude Code 端：註冊為 HTTP MCP / Claude Code side: register as HTTP

在 `~/.claude.json` 的 `mcpServers`（或專案 `.mcp.json`）裡，把 `type` 設成 `http`：

In `~/.claude.json` `mcpServers` (or a project `.mcp.json`), set `type` to `http`:

```json
{
  "bili23": {
    "type": "http",
    "url": "http://127.0.0.1:23330/mcp",
    "headers": {
      "Authorization": "Bearer <YOUR_MCP_TOKEN>"
    }
  }
}
```

等價 CLI 命令 / Equivalent CLI:

```bash
claude mcp add --scope user --transport http bili23 http://127.0.0.1:23330/mcp --header "Authorization: Bearer <YOUR_MCP_TOKEN>"
```

> 憑證一律用佔位符 `<YOUR_MCP_TOKEN>`；不要把真實 token 提交進任何 repo。
> Always use the `<YOUR_MCP_TOKEN>` placeholder; never commit a real token to any repo.

## 對照：stdio vs HTTP / stdio vs HTTP at a glance

| | stdio | HTTP（推薦 / recommended） |
|---|---|---|
| 啟動 Claude CLI 是否拉起 Bili23 | 是 / yes | 否 / no |
| 連接 / Connect | 自動 / automatic | 先開應用，再開 Claude / open app then Claude |
| 斷聯 / Disconnect | 要改設定檔 / edit config | 關掉應用 / quit the app |
| 適合 / Best for | 想永遠掛著 / always-on | 想靈活開關 / on-demand |

## 相關 / See also

- [case-study.md](case-study.md) —— 驅動 MCP 完成批量下載的全流程 / The full batch-download workflow.
- [PITFALLS.md](PITFALLS.md) —— 踩坑清單 / The pitfall list.
