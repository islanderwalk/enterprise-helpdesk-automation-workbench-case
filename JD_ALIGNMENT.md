# JD Alignment — 本萬有限公司 × 梁瀚文

> 來源：104 人力銀行 · 職缺 [`日班電商營運助理（上架／客服／社群支援）`](https://www.104.com.tw/job/8qf2c) （jobNo 14669508）+ 2026-04-13 ~ 04-14 的雙向訊息 (chatroom `8qf2c`)。訊息與 JD 原文存於私有 `docs/104_benwan_*.md`，已加入 `.gitignore`，**不會公開**。

---

## 這個對話的真實脈絡（不是應徵原職缺）

104 公開 JD 是 **日班電商營運助理（上架／客服／社群支援）** — 新竹市香山區現場、月薪 30–40k、主軸商品上架 + 客服訊息 + 社群支援 + 訂單處理。這不是我的主戰場，對方（本萬 hiring manager，非 HR）在 04/13 第一則訊息就直接點破：

> *「您的背景和我們目前這個電商營運助理職缺本身差異蠻大，不過也因為如此，讓我注意到您在系統、自動化與內部工具這塊的經驗其實相當完整。我們公司本身有自建系統，也一直有在思考找一位能協助 **技術盤點、流程整理與自動化方向** 的人才」*

**所以這個對話真正在談的是**：一個**有自建網站與系統**、主事者**自己也有設計與營運流程經驗**的新竹 SMB，想找一位**以專案型／顧問型／遠端支援**為主的人，處理 **營運流程優化／資料整理整合／自動化落地**。

對方 04/14 明確要求一份**公開或去識別化的案例**，按 5 題回答：
1. 原本遇到的實際問題是什麼
2. 您怎麼分析與拆解流程
3. 最後實際交付了什麼
4. 成效或改善點是什麼
5. 這個案例後續如何維護、交接或延續使用

**這份 portfolio 套件就是按這 5 題結構化的回答。**

---

## 對方評估重點 → 本 repo 的對應證據

對方訊息裡的**原文關鍵字**全部列出來，直接對映到這個 repo 或姐妹 demo repo 的哪個位置：

| 對方原話 | 他想看到什麼 | 本 repo / 姐妹 demo 的對應 |
|---|---|---|
| **實際落地經驗** | 不是 "我學過什麼技術"，是 "我真的跑過、踩過雷、修好了、留下來了" | `CASE_STUDY.md §9` 五個真實 challenge（firewall-immunity、credential 外洩、DST 衝突、公開安全重建、OCR 中英混雜），每一個都附上「遇到什麼 / 做了什麼 / 留下什麼 tradeoff」 |
| **營運現場** | 你有沒有站在每天跑這件事的人旁邊？ | `CASE_STUDY.md §2` Background（直接點出每週 / 每月 / 每天重複的事）；`docs/process-breakdown.md` 拆到 phase 級別 |
| **流程整合** | 多系統跨邊界，不是一個 script 搞定一切 | `CASE_STUDY.md §7.3` EIP2_report 5-stage pipeline（Cerberus FTP → EIP2 form → merge → Outlook draft → 批次簽核）；`SYSTEM_OVERVIEW.md` 五階段主幹 |
| **後續維護與優化** | 跑起來之後怎麼辦 | `CASE_STUDY.md §11` Maintenance and handoff；`docs/handoff-notes.md` 第一週閱讀、最常壞的 8 件事、pre-publish checklist |
| **公開或去識別化的案例** | 不是把公司內部檔案 zip 給我，是敢公開的正式對外版本 | 姐妹 repo `../enterprise-provisioning-workbench-demo/` — 可執行 Python CLI、sample JSON、MIT license，跟這份 portfolio 平行發布 |
| **營運流程優化** | 把重複、碎片化的人工流程變成可跑的流程 | `SYSTEM_OVERVIEW.md` before/after 兩張 Mermaid（手動多視窗 → 批次單一 run）|
| **資料整理整合** | 不同源頭的資料能合成一張表 | `CASE_STUDY.md §7.3` Cerberus CSV + EIP2 form + Excel vpnlist 合併去重；`docs/process-breakdown.md` Phase 2–3（normalize + merge with reason string）|
| **自動化落地** | 不是 POC，是真的在跑 | InQuire 3.0 — 90+ integration tests、production-hardened、`jobs/` 四個 domain workflow；104 side — `pytest tests --tb=short -q` 587/587 green（截至 2026-04-02）|
| **公開連結、簡報、流程圖、demo** | 一定要能點開看 | README + CASE_STUDY + 四張 Mermaid（`docs/workflow-diagram.md`）+ 跑得起來的姐妹 demo + 截圖 index（`SCREENSHOT_INDEX.md`）|

---

## 我在 04/13 已經跟對方講的東西（portfolio 必須和這份說法一致）

我 04/13 的訊息已經鎖定這些位置，**portfolio 不能和它們矛盾**：

- 主專案名稱：**104 IT Automation Suite**
- 核心技術：Python、Playwright、requests、OpenAI GPT、SQLite、自製 TF-IDF、claudecode CLI
- 四類落地場景（全部都對應 e-commerce SMB 的日常）：
  1. 商品 / 訂單 / 客服資料**自動擷取與同步**（能串 Google 試算表、Excel、後台系統）
  2. **報表自動生成、異常自動提醒**
  3. 常見問題**自動分類、AI 輔助回覆、聊天機器人**
  4. SOP 流程**自動檢核與知識文件化**
- 協作模式：以遠端、假日彈性為主，專案型／顧問型／遠端支援

這份 portfolio 的 `CASE_STUDY.md §7` 七個 deliverable **每一個都能對映回這 4 類場景**（見 `FOR_BENWAN.md` 的對映表）。

---

## JD 條件欄位對照（104 原文）

| JD 欄位 | 本人條件 | 註記 |
|---|---|---|
| 職務類別（社群行銷／網站行銷企劃／電商行銷） | 非直接本行 | 04/13 對方已同意不走這個路線 |
| 工作地點（新竹市香山區牛埔路 151 號） | 目前遠端 | 04/13 已說明「遠端、假日彈性、專案／顧問／遠端支援」，對方未反對 |
| 工作技能（網路活動規劃、電商營運、社群經營） | 非主力 | 對方已經轉向技術盤點 + 自動化方向 |
| 擅長工具（Excel、Word、打字 20–50、FB/IG/Line） | 全部 ✓ | 門檻類條件，無壓力 |
| 學歷（高中以上）、經歷（不拘）、薪資（30–40k） | 門檻全過 | 薪資是現場職缺基準，顧問型合作需另議 |
| 加分條件（電商平台經驗、客服 / 社群經驗） | 無 | 用 IT automation 背景補位；對方已知 |

---

## 下一步建議

對方 04/14 要求後到今天（04/15）才回，延遲 1 天，**回覆內容要高品質抵銷時差**：

1. 直接貼 `FOR_BENWAN.md` 的 5 題答案（一個 message 就全吃）
2. 附上姐妹 repo `enterprise-provisioning-workbench-demo` 的 GitHub link（推上去後）+ 這份 portfolio 的 link
3. 主動點出「如果您方便，我也可以先針對貴司自建系統的具體場景給 1–2 個簡短的流程圖切入建議，評估看看是否真的對得上」— 把主導權再丟回去

完整回覆稿已擬好在 `docs/reply_to_benwan.md`（已加入 .gitignore，不會外流）。
