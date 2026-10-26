# LORE

## 概觀

- TraceForge：基於 Windows 原生 GNU Emacs GUI 的輕量化程式碼閱讀環境，用來瀏覽 Repository、透過 LSP 追蹤 Symbol、檢視 Git 歷史，以及閱讀含 Mermaid 的渲染後 Markdown。
- 實作前必須閱讀 [README.md](README.md)；其中定義產品範圍、能力邊界、Roadmap 與驗收標準。

## 文件守則

- AI agent 文件必須維持雙語且內容等價 (無後綴即英文版)，例如 AGENTS.md=AGENTS_ch.md。
- AI agent 文件中的連結必須與文件語言對應：英文檔案連到英文目標，繁體中文檔案在有 _ch 目標時連到 _ch 目標。
- [README.md](README.md) 給人閱讀，以繁體中文撰寫。

## 範圍與平台

- TraceForge 必須在 Windows 原生 GNU Emacs GUI 上執行，且必須只針對它設計。
- MVP 嚴禁支援 WSL / WSL2、Linux、macOS、Terminal-only Emacs 或 Docker-based Emacs。
- 嚴禁以 WSL 作為核心功能的替代實作。
- MVP 嚴禁實作：Debugger、程式編譯與部署管理、完整 Java Refactoring Engine、Spring / Hibernate 專用 IDE 功能、Database Client、AI Coding Agent、Email / Calendar / Org-mode、複雜的 GUI Framework。

## Emacs 設定

- 功能應以獨立模組實作，避免互相耦合。
- 優先使用 Emacs 內建 API。
- 優先使用 Emacs 內建套件；非內建套件透過 `package.el` 與 `use-package` 管理。
- 嚴禁自行重寫成熟的 Emacs Package；優先使用成熟套件而非自行實作，且嚴禁自行實作新的搜尋引擎。
- 嚴禁引入不必要的 Framework。
- 保留 Emacs 原生快捷鍵；其他功能優先透過命名命令與 Magit 操作，不應過度修改原生 Keymap。
- 不應要求使用者熟悉大量 Emacs 指令才能完成基本操作。
- 狀態顯示 (Project、語言、Major Mode、LSP 狀態、Git Branch) 可利用 Mode Line；不需要建立自訂 Status Bar Framework。
- MVP 不需要完整的 Visual Project Tree。
- `early-init.el` 僅在實際啟動機制需要時使用；嚴禁假設由 `-Q` 載入 Bootstrap 後會自動處理它。
- 目錄可依實際需要精簡，避免建立沒有實際用途的空模組。

## Language Server

- Eglot 是唯一的 LSP Client；MVP 嚴禁同時引入 `lsp-mode`。
- 優先使用 Eglot 預設的 Language Server Mapping；只有在預設設定不足時，才修改 `eglot-server-programs`。
- 新增語言時，原則上應只增加對應的 Major Mode 與 Language Server 設定，不應修改核心 Navigation API。
- 生命週期：
  - 預設採用按需啟動 (Lazy Initialization)；初次操作時使用者可透過 `M-x eglot` 啟動。
  - 同一 Project、相同語言模式的 Buffer 應儘量共用 LSP Session；不同語言使用各自的 Session。
  - Language Server 異常時，嚴禁影響其他語言與一般檔案閱讀。
  - 自動啟動選項 `traceforge-lsp-auto-start` 預設為 `nil`，且嚴禁在 Emacs 啟動時載入所有 Language Server。
- 不同 Language Server 應可在相同的 Emacs GUI Session 內共存。
- 嚴禁因缺少某個 Language Server 而導致其他語言的導航無法使用；Language Server 必須可以分別安裝。
- Language Server 不支援特定功能時，應提供合理的提示，而不是讓整個導航功能失敗。
- Workspace Root：
  - 嚴禁假設 Git Repository Root 等於 LSP Workspace Root。
  - 不應將所有檔案強制指向最上層 Git Root。
  - 優先尊重 Language Server 自己的 Workspace Detection 與 Emacs Project API；必要時才加入可覆寫的 Root Resolver。
  - 必須避免因為不正確的 Root Detection，導致大型 Monorepo 被重複索引。
- Java (Eclipse JDT LS)：
  - 必須考慮 JDK Runtime 版本需求、jdtls 啟動參數、Windows 專用 Configuration、Maven / Gradle Project Import、Workspace Cache、多個 Java Project 的隔離，以及 Lombok 等額外相依設定。
  - 嚴禁假設所有 Windows 環境都有可直接執行的 `jdtls.exe`；必要時建立專用的 PowerShell Launcher 或 Emacs Lisp 啟動設定。
  - JDT LS Workspace Cache 應存放在 TraceForge Runtime Directory，並依 Project 區分。
- 嚴禁假設標準 LSP 能追蹤跨語言呼叫；跨語言 Call Graph、API Relationship 與 Distributed Service Trace 不納入 MVP。

## Git

- Git 必須透過 Windows 原生 Git for Windows 執行；嚴禁依賴 WSL 或 Git Bash 作為啟動環境。

## Markdown 與 Mermaid 預覽

- 優先使用 Windows 預設瀏覽器完成渲染；嚴禁要求使用者安裝特殊版本的 Emacs 或 WebKit。
- 應避免將 Repository 原始文件複製到公開伺服器。
- 嚴禁假設 EWW 支援完整瀏覽器的 JavaScript Rendering；必須明確區分 Emacs Buffer 內的基本渲染與瀏覽器中完整的 HTML / CSS / JavaScript 渲染。
- MVP 使用 Mermaid.js；渲染資源必須能離線使用已安裝的資源，並固定依賴版本。
- Ondine 必須是可選的 Renderer Backend；MVP 嚴禁強制依賴 Ondine。整合前應驗證其 Windows 可用性、Mermaid 語法相容性及所需 Runtime。

## 安裝與隔離

- 安裝不應直接覆寫使用者現有的 Emacs 設定。
- 必須避免覆寫 `~/.emacs` 與 `~/.emacs.d/`。
- 啟動應採用隔離的 Emacs 啟動方式，例如由 `runemacs.exe` 使用 `-Q` 搭配專用 Bootstrap，明確載入 TraceForge 設定。
- 初始化流程必須正確設定 `user-emacs-directory`、`package-user-dir`、`custom-file`、Cache Directory 與 Runtime Directory。
- 嚴禁假設 `--init-directory` 一定能避開所有既有 Emacs 初始化檔案。
- 每個外部依賴都必須在 [README.md](README.md) 記錄安裝方式。

## Runtime 資料

- 可變動資料建議放在 `%LOCALAPPDATA%\TraceForge\`。
- Git Repository 嚴禁儲存：Language Server Workspace Cache、Emacs 自動產生的暫存檔、Markdown Preview 暫存輸出、使用者本機絕對路徑，以及 Token、Credential 或其他機密資料。
- 所有外部工具應支援 Windows Unicode 路徑。

## 診斷

- 提供 `scripts/doctor.ps1` 與 `M-x traceforge-doctor`，檢查核心依賴與 LSP 狀態。
- 診斷應區分：Critical Error、Optional Dependency Missing、Language Server Missing、LSP Connection Failed、Workspace Initialization Failed。
- Language Server 不存在時，診斷必須提供可讀取的安裝建議。

## 安全與相容性

- 路徑必須測試：空白、Unicode / 繁體中文、Windows 路徑分隔字元、大小寫處理、可執行檔與 `.cmd` Launcher、PowerShell Argument Quoting。
- 嚴禁直接將未清理的使用者輸入拼接為 Shell Command；優先使用 Emacs Process API 或 PowerShell 的安全參數傳遞方式。
- 預覽不可信任的 Repository 時，必須避免：任意 JavaScript 執行、不必要的外部資源載入、任意本機檔案存取，以及將 Repository 文件上傳到外部服務。
- 若建立 Local Preview Server，必須：僅監聽 `127.0.0.1`、使用動態可用 Port、限制檔案存取範圍、防止 Path Traversal、提供停止機制，且不公開 `.git` 或使用者機密檔案。

## 開發流程

- 依照 Phase 1 至 Phase 6 順序實作；嚴禁讓 Phase 7 阻塞 MVP。
- 每個 Phase 都應有可執行的驗收方式。
- 每個 Phase 完成時，必須：
  1. 驗證 Emacs 設定可以載入。
  2. 執行適用的自動化測試。
  3. 更新 [README.md](README.md) 或相關文件。
  4. 在 [README Roadmap](README.md#roadmap) 該 Phase 的狀態記錄未完成與未驗證項目。
  5. 將可驗收功能獨立提交 Commit。
- 驗收必須使用實際 Windows 原生環境。Emacs Batch Mode 可以作為自動化測試，但嚴禁取代 GUI Smoke Test。
- 嚴禁因為無法在當前環境進行 Windows GUI 測試，就宣稱該項目已通過完整驗收。
- Commit 訊息使用 Conventional Commits 前綴 (`feat`、`fix`、`test`、`docs` 等)，後接繁體中文描述。
