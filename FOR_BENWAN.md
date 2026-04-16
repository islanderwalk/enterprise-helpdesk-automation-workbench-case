# 5 題的直接回答 + 我補的第 6 題

> 給本萬有限公司 hiring manager。您 04/14 列的 5 題按順序答在下方；第 6 題是您沒問、但通常決定合作三個月後會不會變成技術債的那一題，也一併回。
> 完整證據鏈見 `CASE_STUDY.md`；可執行 demo 在 `../enterprise-provisioning-workbench-demo/`。
>
> **公開性聲明**：以下是企業內部 IT 自動化實務的去識別化重述。公司名、內部主機名、客戶、員工資料、token、cookie 全部移除，以通用佔位詞替換。同一套架構的可執行公開版本（含 sample 輸入輸出、unittest、Mermaid）在姐妹 repo 裡跑得起來。

---

## 1 · 原本遇到的實際問題是什麼

企業內部 IT 團隊在支援數千位員工時，最耗時間的不是「故障排除」，而是**每週每月都一模一樣、但每次都要手開十幾個視窗**的工作：

- **帳號生命週期**：SSL-VPN、SVN、FTP、ITS、產品包授權、Ivanti、WPM 每週會議 — 七套系統，七套表單，七套信件範本，七次手動貼帳號／密碼／PDF。
- **信件觸發流程**：每張票都手寫 email、手拖 PDF、手編 BCC 名單。
- **收件匣分類**：每天幾百封退信、訓練完成通知、稽核要求 — 誰看完、誰沒看完全靠個人記憶。
- **終端修復**：Teams 登入卡、Pulse Secure 卡 cache、OneDrive 政策切換、ESET 殘檔 — 每一題都是某一個人腦裡的 runbook。

**症狀**：處理不一致、記錄零碎、經驗綁在單點、換人交接要重學、出事之後無法還原做了什麼。

---

## 2 · 我怎麼分析與拆解流程

我刻意把所有流程壓進**同一個五階段骨幹**，每個階段都留下可檢查的產物：

| 階段 | 輸入 | 輸出 | 為什麼這樣切 |
|---|---|---|---|
| **Intake** | Outlook folder / EIP2 form / Excel / Cerberus CSV | 原始 row | 讓 "來源" 這件事單一可查 |
| **Normalize** | 原始 row | 統一 row shape `(user, system, action, source)` | 消除「為了七套系統寫七種處理 code」 |
| **Merge** | 去重、記下合併原因 | 唯一 row + `merge_reason` | 絕對不允許同一人被兩條路重複處理 |
| **Decide** | `CREATE / REVIEW / SKIP` + reason string | 操作員只碰 REVIEW 那一批 | 把例外和標準流程切開 |
| **Execute + Verify** | Adapter 寫入 → readback → telemetry | 執行 summary + 單筆 audit | 每個外部寫入都有「我寫進去了沒？」的回讀 |

旁邊再掛三條 **assist-only** 的支路（不碰外部寫入）：
- AI 退信分類（firewall-immunity regex + OpenAI few-shot vs golden samples）
- OCR + fuzzy-match（訓練證書圖 → 名單行）
- Claude RAG helpdesk chat（員工自助，解不出再建票）

**關鍵設計規則**：AI 永遠只產生「標籤 / 行 / 答案字串」，**絕對不在 Decide 和 Execute 之間**。這條線被我當作不可動搖的安全邊界。

---

## 3 · 最後實際交付了什麼

七個模組，都是真的跑過、有 git history、有測試。括號內是 e-commerce SMB 語境下最對得上的 equivalent，供貴司參考。

1. **InQuire 3.0** — PySide6 桌面外殼，一個 app 同時跑四個 domain workflow（產品包 ETL、SSL-VPN 批次、Ivanti 建/改帳號、WPM 會議排程）。90+ integration tests。私有 source；姐妹 demo 用 sample data 跑得起來。
   *（→ 對應：自建系統 + 多後台整合的桌面操作員工作台）*

2. **EIP2_report 5 階段 pipeline** — `1.fetch_cerberus_list.py` → `2.fetch_EIP2_FSR_list.py` → `3.process_ftp_requests.py` → `4.create_mail_drafts.py` → `5.sign_off_forms.py`。憑證走 env-var（已完成外洩稽核 + 輪替）。
   *（→ 對應：訂單資料+後台資料合併 → 客服回覆草稿 → 批次確認）*

3. **AI Email Classification Engine** — PowerShell MAPI export + Python few-shot 分類器 + Strict Mode。把退信分到 `DELETE / REVIEW / KEEP` 並附理由。
   *（→ 對應：客服訊息自動分類、常見問題 AI 草回）*

4. **outlook_file_download** — Tkinter + Tesseract OCR + Outlook COM，圖片證書 → 人員課程表自動填入 Excel。中英混雜、多閾值 fuzzy match（0.78 / 0.85 / 0.95）。
   *（→ 對應：商品資訊確認 / 圖片 / PDF 資料拆出進後台）*

5. **VBA 確認信套件（7 套）** — FTP/InQuire/ITS/Product Package/SVN/VPN_Internal/WPM，HTML 模板、簽名保留、多 PDF 附件迴圈、200-per-draft BCC chunking（避開 SMTP cutoff）、DST-aware 會議排程。
   *（→ 對應：訂單確認信、出貨通知、批次客戶通訊）*

6. **Windows 急救工具組** — `Teams_Full_Reset.bat`、`fix_pulse_secure.bat`、`OneDrive_off.bat`、`eset_cleanup_deep.bat`。把腦裡 runbook 變成一鍵 script（附 log + exit code）。
   *（→ 對應：後台 / POS / 印表機常見卡關的 SOP 自動化）*

7. **enterprise-provisioning-workbench-demo**（**公開可執行**）— MIT license，Python stdlib 無第三方依賴，`python -m demo_app --show-stages` 直接跑，sample JSON in / run-report JSON out，Mermaid 架構圖、unittest、interview-story。
   *（→ 這就是您要的「公開或去識別化的案例」原型）*

---

## 4 · 成效或改善點是什麼

我不想捏造數字。以下是**可觀察的結構性改變**，每一項都有對應的 log / report / 測試輔證：

- **帳號開通**：從「每張票 ~20 個手動觸點」變成「一次批次 run，出來一份 `create / review / skip` 表 + 已草好的信件草稿」。操作員只讀 REVIEW 那一格。
- **退信分類**：從未被 scope 變成每週一份 risk-classified 表，降低誤刪合法帳號（firewall-immunity 把 DNS 失敗 / 垃圾郵件誤判 / IP block 類型強制保留）。
- **訓練證書登錄**：從「人工開 100 封信貼 Excel」變成「選日期範圍、按 Run」。中英混雜課名 fuzzy-match 命中率由 ~60%（純字串比對）升到 >95%（評估樣本 300 張）。
- **端點急救**：從「每次個人現場重診」變成雙擊一個 `.bat` + log file 自動歸檔。
- **確認信批次**：200-per-draft 之後沒有再撞過 SMTP cutoff。BCC / PDF 附件 / 簽名保留的錯誤率降到接近 0。
- **AI helpdesk 聊天**：長尾「怎麼做 ...」問題在 RAG 階段就攔下，進 ticket 的比例下降。

**規範保證層面**（對後續維護比數字更關鍵）：
- 104 side `pytest tests --tb=short -q` **587/587 green**（截至 2026-04-02）。
- InQuire 3.0 **90+ integration tests** + brainstorm/plan/implementation docs per milestone。
- 2026-04-15 完成一輪 **憑證外洩稽核 + 輪替**：OpenAI key（已輪替）、EIP2 / Ivanti 明碼密碼（5 個檔案改 `_credentials.py` + `.env`），同日修復 WPM outlook_gateway.py 的 SSR (Strip-Send-Restore) 重複寄信 bug（11/11 regression tests green）。

---

## 5 · 這個案例後續如何維護、交接或延續使用

我從一開始就把這些工具設計成「**比我活得久**」：

1. **Docs 不是事後補的，是 commit 之前就寫的**：每個模組都有 `README.md` + `*.brainstorm.md` + `*.plan.md`；非瑣碎改動走「brainstorm → plan → 實作」的 loop，`why` 留在 repo 裡可搜尋。
2. **測試就是 release gate**：改動要跑 `pytest` 到綠；InQuire 3.0 / 104 side / 姐妹 demo 三套 suite 各自獨立。
3. **憑證唯一路徑是 env-var**：沒有「暫時先寫死」例外。本次稽核把殘留的 6 個硬編碼點全部消除。
4. **Telemetry 存 dated JSONL**：上週四那個 run 的回放是機械動作，不是考古。
5. **公開 ↔ 私有分離**：`InQuire 3.0/` 是內部 source；姐妹 `enterprise-provisioning-workbench-demo/` 是公開鏡像架構。談外部合作時，談姐妹 demo 不會洩露內部。
6. **第一週閱讀清單**：新手進來先讀 `README` → `SYSTEM_OVERVIEW` → `CASE_STUDY` → 模組自己的 README → 最新一對 `brainstorm.md` / `plan.md`。跑 `python -m demo_app --show-stages`，看到 Action Board 輸出。**不需要 shadow 任何人**。

換句話說 — 這整套的交接成本不是「跟我排時間」，是「你給我兩天，這份 repo 我讀完」。

---

## 6 · 這套自動化在什麼條件下我會主動建議**暫停使用、回到人工處理**

您沒問這一題，但這題通常決定合作三個月後會不會變成技術債。

三個情境我會直接講「不要自動化」：

1. **標的系統的 UI 或 API 每季都在變、且沒有穩定 contract** — 這時自動化的維護成本會超過節省的人工，不如讓熟悉的人繼續點擊，把工程時間投到更穩定的前置或後置環節。
2. **下游動作是高度信任的**（金流、對客信件、庫存異動）**而業務端還沒準備 double-check 機制** — 這時堅持 human-in-the-loop。自動化可以產草稿、送審、記 audit trail，但送出按鈕留給人。
3. **日常操作員人數 ≤ 2 且他們每天都親自看到流程** — 這時 SOP + manual runbook 的交接成本比自動化低；對新人，讀三頁 SOP 比讀三個 repo 好上手。

這三題的共同結構：自動化的價值不是「能不能做」，是「做了之後整體系統的熵會降還是升」。若合作開始，我通常會先花半天做這個判斷，不是先動手寫 code。

---

## 可驗證的公開材料

- **本 portfolio**（這份 `.md` 所在的 repo）：<https://github.com/islanderwalk/enterprise-helpdesk-automation-workbench-case>
- **姐妹可執行 demo**：<https://github.com/islanderwalk/enterprise-provisioning-workbench-demo>
  - README 含架構 Mermaid
  - `python -m demo_app --show-stages` 直接跑
  - `python -m unittest discover -s tests` 跑過
  - `docs/interview-story.md` / `docs/architecture-overview.md`
- **Mermaid 流程圖**（before / after / AI-assist / handoff）：[`docs/workflow-diagram.md`](docs/workflow-diagram.md)
- **去識別化 sample 輸入 / 輸出**：[`examples/sanitized_input/requests.json`](examples/sanitized_input/requests.json) + [`examples/sanitized_output/run-report.json`](examples/sanitized_output/run-report.json)

### 同一個 GitHub profile 上的相關 repo

![GitHub profile — islanderwalk](assets/screenshots/00_github_profile_repos.png)

- `enterprise-provisioning-workbench-demo` — 本案姐妹公開 demo
- `InQuire-3.0` — 本案私有 source（architecture 在公開 demo 裡描述）
- `Outlook-Email-Behavior-Classification-Engine` — AI 退信分類模組
- `BMC_BIOS_1213`、`EnglishAI-Toolkit`、`ai-short-video-mvp`、`islanderwalk/islanderwalk` — 其他 side project / profile README

---

## 我在過去 24 小時順手做的兩件事（時間訊號）

整理這份公開包的同一時段（2026-04-14 至 04-15），我在私有端完成了兩件和本案主題直接相關的工作，列在這裡是因為它們本身就是「後續維護與優化」的證據：

- **一輪憑證外洩稽核** — 過去某個 POC 期留下的 OpenAI API key 明碼、以及 EIP2 / Ivanti 的硬編碼密碼共 6 處，全部輪替並改用 `_credentials.py` + `.env` pattern；`.env` 已納入 `.gitignore`。發現即修復，同日完成。
- **修復一個 Outlook meeting 的 SSR (Strip-Send-Restore) 重複寄送 bug** — 路徑在 `InQuire 3.0/jobs/wpm/outlook_gateway.py`；restore 階段和主流程的 send 疊加會雙寄。加了一個 `continue` 把 restore 之後的 send 路徑切掉，11/11 regression tests 全綠。

這兩件事不是特別為您做的 —— 是同一套標準在跑，恰好今天輪到它們。我寫出來，因為它是「實際落地經驗、後續維護與優化」這件事最誠實的版本：自己過去的設計錯誤，自己抓出來、自己修。

---

## 協作模式（事實陳述，非提議）

以遠端、專案型或顧問 retainer 為主。新竹現場本身我沒在排除，但不是主軸。具體 scope 對齊前，價碼先不談 — 先看合作方向是否對得上。

---

## 和 04/13 訊息的連續性（你問「最接近營運流程優化／資料整理整合／自動化落地」的案例）

我 04/13 提到四類場景，都能找到 repo 裡的具體對應：

| 04/13 說過的場景 | 這份 portfolio 的對應 |
|---|---|
| 商品 / 訂單 / 客服資料自動擷取與同步 | Intake layer（`EIP2_report/fetch_*.py`、`outlook_file_download/outlook_bot.py`）；normalize → merge 的結構 |
| 報表自動生成、異常自動提醒 | `Ivanti_Auto_Report/`、`EIP2_monthly_report/`、telemetry JSONL 聚合 |
| 常見問題自動分類、AI 輔助回覆、聊天機器人 | AI Email Behavior Classification Engine（firewall-immunity + few-shot）；InQuire 3.0 `services/helpdesk_service.py`（Claude + chromadb RAG）|
| SOP 流程自動檢核與知識文件化 | 每模組的 `README` / `brainstorm.md` / `plan.md` 寫法；InQuire 3.0 的 AGENTS.md / CLAUDE.md 雙層指引；確認信 VBA 的 HTML template + 簽名保留作為「SOP 即程式碼」 |

如有需要，我可以針對貴司自建系統的具體場景（例如某個後台 API 不穩、某個客服分類任務、某份 Excel 需要跟後台對帳）先畫 1–2 張小型 before/after Mermaid，再評估合作。
