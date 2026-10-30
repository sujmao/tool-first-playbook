# 工具優先 / Tool-First

充分利用現有工具的優勢，不重複造輪子。
Leverage the strengths of existing tools — don't reinvent the wheel.

> 一份「經驗總結遷移」repo —— 把一次真實的批量自動化任務中「用對工具、少走彎路」的方法論與踩坑記錄沉澱下來，供其他人複用。
> A knowledge-transfer repo distilling the methodology and pitfalls from one real batch-automation task, so others can reuse the playbook and skip the detours.

## 一句話主旨 / One-line thesis

先盤點現有工具的能力，讓成熟的工具做它最擅長的事（鑑權、反爬、下載、合併），自己只寫一層薄薄的膠水，而不是重新造輪子。

Inventory existing tools first; let the mature tool do what it does best (auth, anti-bot, download, muxing) and write only a thin glue layer yourself — don't reinvent the wheel.

## 這個 repo 是甚麼 / What this repo is

- **一次案例 / A case study**：驅動 [Bili23-Downloader](https://github.com/ScottSloan/Bili23-Downloader) 自帶的 MCP 介面，批量下載某個 B 站空間的全部投稿，並在事前估算總大小、事後統一重命名。
  Driving the built-in MCP interface of Bili23-Downloader to batch-download every upload in a Bilibili space, estimating total size upfront and renaming the results afterward.
- **一套方法論 / A methodology**：甚麼時候該「用現成工具」、甚麼時候才「自己動手」，以及中間的判斷信號。
  When to "use the existing tool" vs. "build it yourself", and the signals in between.
- **一份踩坑清單 / A pitfall list**：把實際撞過的坑逐條記錄，附「原因 + 避開方式」，讓下一個人不用再踩。
  Every trap actually hit, each with its cause and how to avoid it.

## 目錄 / Index

| 文件 / File | 內容 / Contents |
|---|---|
| [PRINCIPLES.md](PRINCIPLES.md) | 方法論：工具優先的決策框架 / Tool-first decision framework |
| [case-study.md](case-study.md) | 案例：Bili23-Downloader MCP 批量下載全流程 / Full workflow |
| [PITFALLS.md](PITFALLS.md) | 踩坑清單（核心，逐條可查）/ Pitfall list (core reference) |

## 適用讀者 / Who this is for

- 想做「自動化下載 / 批量處理」，但不想從零逆向目標平台的人。
  Anyone automating downloads or batch jobs who doesn't want to reverse-engineer a platform from scratch.
- 面對一個「已有桌面應用 / 已有 CLI / 已有 MCP」的任務，猶豫該不該自研的人。
  Anyone hesitating to self-build when a desktop app / CLI / MCP already exists.
- 想把手上的排程任務從「人肉點擊」變成「腳本編排」的人。
  Anyone turning a manual click-through task into scripted orchestration.

## 隱私說明 / Privacy

本 repo 所有範例均已**匿名化**：具體 UID、本機路徑、Token、Cookie 值一律用佔位符取代，只保留可複用的方法與通用路徑（例如應用標準設定檔位置）。請勿把真實憑證提交進任何 repo。

All examples are anonymized: concrete UIDs, local paths, tokens and cookie values are replaced with placeholders, keeping only reusable methods and generic paths. Never commit real credentials to any repo.
