# 修改紀錄 / Changelog

本檔案記錄 `index.html` 的修改歷史。格式大致依循 [Keep a Changelog](https://keepachangelog.com/)，但因為這是單一檔案、沒有版號的專案，改用日期分段。

This file tracks changes to `index.html`. Loosely follows [Keep a Changelog](https://keepachangelog.com/); dated sections instead of version numbers since this is an unversioned single-file project.

## 2026-09-15

### 新增 / Added

- 專案發布到 GitHub Pages：https://sinliongtoo.github.io/dynamic-schedule-tool/ （公開儲存庫 `SinLiongToo/dynamic-schedule-tool`）。主檔案由 `dynamic-scheduler (2).html` 更名為 `index.html`，讓網址不需要編碼空格/括號即可直接載入。
  Published the project to GitHub Pages: https://sinliongtoo.github.io/dynamic-schedule-tool/ (public repo `SinLiongToo/dynamic-schedule-tool`). Renamed the main file from `dynamic-scheduler (2).html` to `index.html` so the URL loads without needing to encode spaces/parentheses.
- 復原／重做：右上角「↶」「↷」（或 Ctrl+Z／Ctrl+Y）可復原、重做幾乎所有會改變資料的操作——拖曳排程、表格直接編輯、自動排程、新增／複製／刪除任務或資源、匯入 JSON／CSV、修改排程視窗設定等，最多可回溯 50 步。在文字輸入框內按 Ctrl+Z 仍是瀏覽器原生的欄位復原，不會被攔截。
  Undo/redo: "↶"/"↷" (or Ctrl+Z/Ctrl+Y) undoes and redoes almost every state-changing action — drag rescheduling, inline table edits, auto-schedule, add/duplicate/delete task or resource, JSON/CSV import, horizon/settings changes — up to 50 steps of history. Ctrl+Z inside a text field is left alone as the browser's native field-undo, not intercepted.

### 變更 / Changed

- 「範例資料」按鈕更名為「半導體晶片製程範例」，讓標籤明確反映內容本身就是晶圓廠前段製程（光罩→蝕刻→CMP→量測→爐管）；資料內容未變。同步更新「說明」(❓) 文字。
  Renamed the "Sample data" button to "Semiconductor Fab Process Example" to make the label explicit about what it already contains — a front-end wafer fab flow (photo → etch → CMP → metrology → furnace); the underlying data is unchanged. Updated the in-app Help (❓) text to match.

## 2026-09-09

### 新增 / Added

- 任務清單：搜尋框、狀態／資源篩選、排序（優先權／截止日／開始時間／名稱），每列可直接鎖定／解鎖 📌、複製 ⧉、刪除 🗑，不必開編輯視窗。
  Task list search, status/resource filters, sort (priority/deadline/start/name), and per-row quick actions (lock, duplicate, delete) without opening the edit dialog.
- 甘特圖：改用滑鼠＋觸控皆可用的拖曳方式（取代舊版僅支援滑鼠的 HTML5 drag-and-drop）；新增今天標記線與「📍 今天」跳轉按鈕；新增「顯示相依關係箭頭」，在甘特圖上以箭頭連線標出任務間的前置/後續關係；新增時間軸縮放（60%–250%，預設 125%），解決任務條過短、文字被壓縮看不清楚的問題；新增右上角「⛶ 放大」，可將整個甘特圖（含所有控制項）彈出全螢幕檢視。
  Gantt: switched task-bar dragging to pointer events (mouse + touch, replacing the old mouse-only HTML5 drag-and-drop); added a today marker line with a "📍 Today" jump button; added a "show dependency arrows" toggle connecting related tasks; added timeline zoom (60%–250%, default 125%) to fix cramped/unreadable labels on short task bars; added a top-right "⛶ Expand" button that pops the whole Gantt chart (controls included) into a fullscreen view.
- CSV 匯出（任務清單，含資源名稱與依賴關係，UTF-8 BOM 避免 Excel 開啟中文亂碼）。
  CSV export of the task list (with resource names and dependencies, UTF-8 BOM so Excel doesn't garble Chinese text).
- 匯入／匯出格式選單：右上角「匯出」旁可選 JSON（完整備份）或 CSV（任務清單）；匯入時依副檔名自動判斷格式。CSV 匯入會與現有任務「同步」：任務名稱相同就更新該任務（不覆蓋其已排定時間／鎖定狀態），不同則新增；resource 欄位若填入不存在的資源名稱會自動新增該資源。
  Import/Export format menu next to "Export" (JSON full backup or CSV task list); import auto-detects format by file extension. CSV import syncs by task name (updates in place without touching its schedule/lock, adds new names) and auto-creates unrecognized resource names.
- 新增 `.claude/skills/run-dynamic-scheduler/SKILL.md`：給未來開發／AI 助理的技術筆記（如何在沒有伺服器/建置流程下用 Playwright 開啟並操作這個檔案、已知的疑難雜症）。
  Added `.claude/skills/run-dynamic-scheduler/SKILL.md` — technical notes for future development/AI-assisted sessions (how to drive this build-less, server-less file with Playwright, known gotchas).
- 新增本 README、CHANGELOG。
  Added this README and CHANGELOG.
- 新增「總覽」儀表板分頁，設為預設進入畫面：任務統計（總數／待排程／進行中／已完成／逾期風險／過載資源，異常時數字轉紅）、逾期風險任務清單、資源熱點（負載最高的資源）、即將開始的任務，點清單項目可直接開編輯視窗。
  Added an "Overview" dashboard tab as the new default landing view: task counts (total/pending/in-progress/done/at-risk/overloaded resources, numbers turn red when non-zero), an at-risk task list, resource load hotspots, and what's starting soon — click any list item to open its edit dialog.
- 「任務清單」與「資源清單」改為可直接編輯的表格：欄位（名稱、工時、優先權、資源、截止日、狀態、進度、容量、換線時間等）可直接點擊修改，change 時即自動儲存並重新渲染，不必每次開彈出視窗；任務列保留 ✎ 按鈕開完整編輯視窗（用於前置任務等表格放不下的欄位）。
  "Tasks" and "Resources" are now directly-editable tables: fields (name, hours, priority, resource, deadline, status, progress, capacity, setup time, etc.) commit on change without opening a popup; a task row's ✎ button still opens the full edit dialog for fields that don't fit a table cell (like dependencies).
- 視覺調整：看板卡片、逾期風險清單改用彩色徽章（狀態／優先權／逾期）取代純文字；標頭右側控制項加上分隔線分組（語言/主題 · 匯出入 · 範例資料 · 自動排程）。
  Visual refresh: Kanban cards and the at-risk list now use colored badges (status/priority/at-risk) instead of plain text; header controls are grouped with dividers (language/theme · import/export · sample data · auto-schedule).

### 修正 / Fixed

- 修正甘特圖任務條（跨多個班次時會視覺上蓋過相鄰儲存格）因 CSS 疊層順序被右側儲存格蓋住，導致點擊/拖曳該任務條中段以後完全沒反應的問題（`z-index` 調整）。
  Fixed a stacking-order bug where a multi-slot task bar visually overflowed into neighboring grid cells but those cells painted on top of it — clicking/dragging anywhere past the first slot silently missed the bar (fixed via `z-index`).
- 修正「任務／資源設定」頁工具列按鈕（下載範本／匯出／匯入／新增）因版面 `flex:0` 設定被壓縮到只剩最小寬度、逐一換行變成直的一排，改為 `flex:0 0 auto`。
  Fixed the Tasks/Resources toolbar buttons collapsing to a narrow column and stacking vertically one-per-line, caused by `flex:0` shrinking the group to its minimum width; changed to `flex:0 0 auto`.
- CSV 匯出／範本下載補上 UTF-8 BOM，修正 Excel 開啟時中文欄位顯示亂碼的問題；CSV 匯入端同步補上防呆，去除可能殘留的 BOM 字元。
  Added a UTF-8 BOM to CSV export/template downloads to fix Chinese text garbling when opened in Excel; CSV import now also strips a stray leading BOM defensively.

### 變更 / Changed

- 移除原本分散在甘特圖頁與任務清單頁、各自獨立的「匯出 CSV」／「匯入 CSV」按鈕，整合進右上角統一的「匯出」／「匯入」（保留「下載 CSV 範本」，用途不同）。
  Removed the separate "Export CSV"/"Import CSV" buttons that were duplicated across the Gantt view and the task list view, consolidating them into the single header Export/Import controls (kept "Download CSV template" since it serves a different purpose).
- 更新內建「說明」(❓) 內容，涵蓋以上所有新功能，並新增「建議流程」：匯入／同步任務資料 → 自動排程 → 手動微調。
  Updated the in-app Help (❓) content to cover all of the above, including a new "Suggested workflow" note: import/sync data → auto-schedule → manual fine-tuning.
