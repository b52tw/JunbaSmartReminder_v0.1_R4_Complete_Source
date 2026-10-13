# Junba 智慧提醒 v0.1 R4 — 恢復功能版（Android 8.0+）

這是 **完整 Android 原始碼及 GitHub Actions APK 編譯專案**，不是現成 APK。目標裝置 vivo V2413 / Android 16。版號 `0.1.4-R4`，`applicationId=tw.junba.reminder` 保持與 R3 相同。

## R4 新增／恢復

- **本月／今日簡易日曆**：上／下月、上／下日、日期格點選，黃色選取日；**灰點**＝未連動、**紅點**＝已提出原廠建立請求待核對、**藍點**＝本人手動確認原廠設定。日曆展示事件及 App 自有每日／每週／每月記事。
- **完整日曆帳號／來源勾選**：讀取手機 `CalendarContract` 已授權的日曆，依完整帳號名稱與日曆名稱逐一勾選。只有手機已同步的 Google 日曆／Gemini 語音建立日曆活動可顯示；**無法**從 Gemini API 讀取助理歷史。
- **每筆日曆活動 → vivo 原廠時鐘**：選提前分鐘、呼叫 `AlarmClock.ACTION_SET_ALARM`，**由使用者到原廠時鐘確認**。記事也可逐筆送交原廠時鐘、編輯、加入 Google 日曆（開啟編輯畫面待儲存）。提供原廠 2 分鐘測試入口及 App 1 分鐘通知測試，兩者清楚區分。
- **連動紀錄**：記錄送交請求與「本人已在 vivo 原廠確認」，重開 App 後保留；**不冒充原廠鬧鐘清單**。
- **Gemini API Key**：設定區可展開／收合；一鍵開啟 https://aistudio.google.com/apikey 取得／管理金鑰；金鑰採 Android Keystore AES/GCM **本機加密儲存**，可更新／刪除及測試連線。AI 查詢明確由使用者勾選，才會傳送當次記事及已授權日曆摘要。
- **提醒管理**：App 自有記事可指定年月日、提前提醒（0/5/10/30/60 分／1 天）、每日／每週／每月重複、編輯／取消，開啟 App 或重開機、調整時間／時區時重排程。精確排程前檢查 Android 特殊權限；App 自有通知**不是**原廠鬧鐘鈴聲。
- **備份／還原**：App 自有記事與連動紀錄可匯出／匯入 JSON；**不包含 API Key**。R3 舊版未有備份 UI，Windows 有 `tools/Backup_R3_on_Windows.ps1` 供具備 ADB 與 USB 偵錯的使用者嘗試備份 debug 版的記事，請勿在備份前解除安裝 R3。

## 系統限制（不應誤認為已實現）

Android 公開 API **沒有**提供全部 vivo 原廠鬧鐘的逐筆查詢與編輯功能，亦無法取得原廠稍後提醒的真實次數及間隔；`getNextAlarmClock()` 只提供下一個系統鬧鐘。

`ACTION_SET_ALARM` 可送時、分、名稱及部分週重複，**沒有完整年月日欄位**；原廠可能建立在「下一次同時分」，使用者必須核對當日／重複／啟用狀態。App 的「本人已核對」只代表使用者手動標記，不代表 App 自動讀回原廠設定。原廠鬧鐘無法由 App 自動全量匯入，語音建立的原廠鬧鐘也一樣。

## 注意：覆蓋安裝與資料安全

GitHub Actions 的 debug APK 可能在不同 Runner 上產生**不同簽章**，即使 applicationId 一樣也可能無法覆蓋舊版。**不要為了安裝新版直接解除安裝舊版**，否則舊 App 的記事與設定會遺失（`android:allowBackup=false`）。

如你已安裝 R3 **debug APK**，可在 Windows 上連接已開啟 USB 偵錯的手機，安裝 Android platform-tools（adb），使用 `powershell -ExecutionPolicy Bypass -File tools/Backup_R3_on_Windows.ps1`，確認得到 `Junba_R3_to_R4_backup.json`。R4 安裝後到「設定 → 匯入備份」選取此檔案。若原版非 debug 或 `run-as` 失敗，請停止移除舊版，資料可能無法備份。

正式長期更新應使用由使用者自己保管、**每次不變的正式簽章金鑰**；絕對不要把正式私鑰直接放到公開 GitHub Repository。R4 此測試專案尚未配置正式發布簽章。

## GitHub Actions 編譯 APK

將壓縮檔內所有內容放在 Repository 根目錄，包括隱藏的 `.github`，避免再套一層資料夾。只保留一份 `.github/workflows/build-apk.yml`，請移除任何名稱不同且過時的舊 Workflow。

進入 **Actions → Build Android APK → Run workflow** 或提交到 main。成功後，點選本次執行 → **Artifacts → JunbaSmartReminder-v0.1-R4-debug-APK**，解壓縮取得 `app-debug.apk`。GitHub Actions 設定採 R3 已通過的 Java 17／SDK 35／Gradle 8.10.2 流程，只更新產出名稱並新增專案檔案檢查。

## 驗收順序

1. 複查 App 是否能重啟、顯示既有記事與帳號／日曆勾選（手機需允許日曆及通知權限）。
2. 到日曆的今日／本月查看同步事件，點選事件後使用「建立 vivo 原廠鬧鐘」並在原廠畫面**確認**；回到連動紀錄手動標記。
3. 使用「vivo 原廠鬧鐘測試」核對真正原廠鈴聲與貪睡；**App 通知測試**僅測試自有通知，不可混淆。
4. 到「設定 → Gemini API Key」，點選取得 Key，存入並測試（需 Google AI Studio 有效 Key、網路與配額）。
5. 測試更新、時間改動、重開機及 vivo 背景省電管理設定；在真機測試實際響鈴。

## 實際驗證狀態

- 已用 Java 編譯器語法分析器檢查所有 Java 檔的**語法**，並檢查 Manifest XML、Actions YAML 及 ZIP 完整性。
- **尚未使用 Android SDK 實際編譯 APK、也未在 vivo 真機驗證**。這份是 R4 候選版完整程式碼，不能宣稱已交付「可安裝且已實測成功的 APK」。
