# 動態排程工具 / Dynamic Scheduler

一個單一 HTML 檔案的「動態排程」原型：輸入任務、資源與目前狀態，系統用規則式演算法（非 AI）自動排出時程；狀況改變時可重新自動排程，也可以直接手動拖曳覆蓋結果。中／英文介面、深／淺色主題皆可切換。

A single-file HTML prototype for dynamic scheduling: enter tasks, resources, and current status, and a deterministic rule-based algorithm (not AI) lays out a timeline. Re-run it whenever things change, or override the result by dragging. Bilingual (Traditional Chinese / English) UI, dark/light theme.

**線上試用 / Live demo:** https://sinliongtoo.github.io/dynamic-schedule-tool/

## 執行方式 / Running it

不需要安裝、不需要伺服器——直接用瀏覽器開啟這個檔案即可：

No install, no server — just open the file directly in a browser:

```
index.html
```

資料會存在瀏覽器的 localStorage（同一台電腦、同一個瀏覽器下次開啟會記得資料）。想要備份或搬到別台電腦，用右上角「匯出」存成 JSON。

Data persists in the browser's localStorage (same computer, same browser). To back up or move to another machine, use "Export" (top right) to save a JSON file.

## 主要功能 / Features

- **總覽儀表板**：一打開就看到——任務統計、逾期風險任務、資源負載熱點、即將開始的任務。
  Overview dashboard on landing: task counts, at-risk tasks, resource load hotspots, and what's starting soon.
- **自動排程**：依前置任務、優先權、截止日自動安排時程，支援共用資源池、換線／緩衝時間、跳過週末。
  Auto-schedule by dependencies → priority → deadline, with shared resource pools, setup/changeover buffers, and optional weekend-skipping.
- **甘特圖**：拖曳（滑鼠或觸控皆可）調整時間與資源、今天標記線＋一鍵跳至今天、相依關係箭頭、關鍵路徑標示、時間軸縮放（60%–250%）、右上角「⛶ 放大」可全螢幕檢視。
  Gantt chart: drag (mouse or touch) to reschedule, a "today" marker with jump-to-today, dependency arrows, critical-path highlighting, zoom (60%–250%), and a fullscreen expand view.
- **看板 / 資源負載**：依狀態分欄檢視（含彩色徽章）；各資源使用率一覽。
  Kanban board by status (with colored badges); per-resource utilization view.
- **任務／資源設定**：可直接編輯的表格——欄位改完自動存檔，不必開彈出視窗；搜尋、篩選（狀態／資源）、排序，逐列快速鎖定／複製／刪除。
  Tasks/Resources: directly-editable tables — fields save automatically, no popup needed; search, filter (status/resource), sort, and per-row quick actions (lock, duplicate, delete).
- **匯入／匯出**：右上角選單可選 JSON（完整備份，匯入會取代目前資料）或 CSV（任務清單，匯入會與現有任務「同步」——名稱相同就更新、不同就新增、未知資源自動建立）。
  Import/Export: pick JSON (full backup, import replaces current data) or CSV (task list only, import syncs by task name and auto-creates unrecognized resources).
- **排程異動比較**：每次自動排程後跳出視窗列出哪些任務被搬動。
  A diff dialog after every auto-schedule run shows what moved.
- **復原／重做**：右上角「↶」「↷」或 Ctrl+Z／Ctrl+Y，可復原、重做幾乎所有操作（拖曳、表格編輯、自動排程、新增／刪除、匯入等），最多 50 步。
  Undo/redo: "↶"/"↷" or Ctrl+Z/Ctrl+Y undoes and redoes almost any action (dragging, table edits, auto-schedule, add/delete, imports), up to 50 steps.

完整使用說明請點應用程式右上角「❓」。

For full usage instructions, click the "❓" button inside the app.

## 檔案結構 / Files

```
index.html          整個應用程式（HTML + CSS + JS，單一檔案）
.claude/skills/run-dynamic-scheduler/SKILL.md   給開發者／AI 助理的技術筆記（如何開啟測試、已知的疑難雜症）
README.md                            本檔案
CHANGELOG.md                         修改紀錄
```

## 開發備註 / For developers

沒有建置流程，也沒有測試套件——整個程式都在一個 `<script>` IIFE 裡。若要用瀏覽器自動化工具（例如 Playwright）驅動測試，請先看 [.claude/skills/run-dynamic-scheduler/SKILL.md](.claude/skills/run-dynamic-scheduler/SKILL.md)，裡面記錄了幾個容易踩到的坑（下載事件的擷取方式、CSV 的 UTF-8 BOM、甘特圖任務條的 z-index 疊層問題等）。

No build step, no test suite — the whole app lives in one `<script>` IIFE. Before driving it with browser automation (e.g. Playwright), see [.claude/skills/run-dynamic-scheduler/SKILL.md](.claude/skills/run-dynamic-scheduler/SKILL.md) for gotchas already hit (capturing download events, the CSV UTF-8 BOM requirement, the taskbar z-index stacking issue, etc.).
