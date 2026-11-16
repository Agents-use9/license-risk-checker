# 授權檢查器（License Risk Checker Skill）v1.1

> **專為開發者、產品經理與架構師打造的開源授權商用風險檢查 Skill**  
> 一個為 Claude Code、Codex 及各類 LLM Agent 設計的開源工具，輸入專案網址或本地路徑，自動產出繁體中文商用風險評估報告。

---

## 為什麼需要這個專案？（誠實定位與差異化）

市場上已有許多優秀的免費開源掃描工具（例如 `license-checker`、`cargo-deny`、`pip-licenses`、`ScanCode Toolkit`，以及 `FOSSA` / `Snyk` 的免費版本）。這些工具已經能非常精準地**提取 SPDX 識別碼並列出相依套件授權清單**。

**本專案絕對不是「唯一能檢查授權的工具」，也無意重複發明輪子。**

本 Skill 的核心定位與專注賣點在於傳統掃描工具容易遺漏的四大「深水區」：

1. **中文白話商用風險判斷與源代碼可用限制**：
   - 清楚告知閉源專案或 SaaS 雲端架構下是否會觸發 AGPL-3.0 第 13 條網路傳染強制開源（如 `omnidiff` 案例）。
   - 識別源代碼可用協議（Source-Available / Fair-code），明確告知其「僅限內部業務目的（internal business purposes）」之限制，禁止將軟體作為雲端服務託管銷售（如 `n8n` 案例，深度對比 AGPL 與 SSPL 之機制差異）。
2. **AI 模型特有授權地雷**：
   - **代碼與權重授權脫鉤**：GitHub 代碼寫著寬鬆的 Apache-2.0，但權重實質採用禁止商用的研究協議（如 `Qwen-Image-2.1` 案例）。
   - **地域排除條款（Geographic Exclusions）**：如 `MiniMax H3` 社群協議明文排除美國、歐盟、英國、南韓四地，在該區域運行即構成侵權。
   - **規模與營收門檻**：年營收超過 2,000 萬美元（MiniMax H3 條款）、月活躍用戶超過 1 億（Qwen 商業版）或 7 億 MAU 需另行取得授權（如 `Meta Llama 3.1` 案例，包含 AUP 禁區與發布模型開頭冠上 "Llama" 之命名義務）。
   - **模型權重 vs 輸出物商用權限分離**：如 `FLUX.1 [dev]` 案例，權重本身嚴禁商業部署，但協議明文放行產出之圖片（Outputs）用於商業目的，同時禁止輸出物用於蒸餾競品模型。
3. **遙測、隱私與隱含條款坑（Default-On Telemetry）**：
   - 深入原始碼層檢查專案是否「預設開啟遙測（Default-On）」（如 `Next.js` 案例中 `storage.ts` 預設值為 `true` 向 `telemetry.nextjs.org` 回傳數據）。
   - 提供標準環境變數（`NEXT_TELEMETRY_DISABLED=1`）、CLI 關閉開關與本地端除錯審查模式。
   - 排查常見於大模型協議中的「單方反向賠償條款（Reverse Indemnity）」（如 Llama 3.1、FLUX.1 [dev]）。
4. **跨來源說法不一致（客觀揭露不選邊、突破自動化掃描盲區）**：
   - 揭露傳統 SCA / SPDX 工具只讀取根目錄 `LICENSE` 檔案所產生的「假綠燈盲區」（如 `Diff-HMR` 案例：`LICENSE` 為 MIT 讓工具亮綠燈，但 `README.md` 卻手寫宣告 `CC BY-NC` 嚴禁商用）。
   - 堅持客觀並列雙方原文，不主觀替維護者選邊或找藉口，直接提示法律模糊性與處置行動。

---

## 零瞎編守則（Honesty First）

- **每條結論皆附來源**：每個風險評級與條款摘要，均附有具體檔案路徑或公開官網/Model Card 連結，並標明查核日期（2026-11-15）。
- **查不到就寫「未核實」**：凡未公開代碼、連結失效或缺少文本者，一律標註為「未核實」，絕不憑空捏造條文或數字。

---

## 目錄結構

```text
.
├── LICENSE                     # MIT 授權條款（由維護者 kang 決定與調整）
├── README.md                   # 專案介紹、安裝用法與免責聲明
├── SKILL.md                    # Claude Code / Codex 可直接載入之 Skill 主檔（v1.1）
├── 授權檢查清單.md              # 逐項勾選的互動式標準檢查清單（1:1 對應 SKILL.md v1.1）
├── 範例報告.md                  # 包含 8 個真實開源專案與模型之完整實測報告（3 基礎 + 5 擴充）
└── skills/
    └── license-risk-checker/
        └── SKILL.md            # 標準 Anthropic / Claude Code 規範路徑
```

---

## 安裝與使用方式

### 方式 1：在 Claude Code 中使用

將本倉庫 clone 至本機或放入專案的 skill 目錄：

```bash
# 複製至您的 Claude Code 配置目錄
git clone https://github.com/your-username/license-risk-checker.git
```

在 Claude Code 對話中直接調用：

```bash
# 檢查遠端 GitHub 專案
/license-risk-checker https://github.com/ivankovic/omnidiff

# 檢查工作流自動化專案之 Sustainable Use License
/license-risk-checker https://github.com/n8n-io/n8n

# 檢查前端框架之 Default-On Telemetry
/license-risk-checker https://github.com/vercel/next.js

# 檢查 Hugging Face 開源大模型（地域排除、MAU門檻、非商用權重與輸出分離）
/license-risk-checker https://huggingface.co/MiniMaxAI/MiniMax-H3
/license-risk-checker https://huggingface.co/meta-llama/Llama-3.1-8B
/license-risk-checker https://huggingface.co/black-forest-labs/FLUX.1-dev

# 檢查跨來源說法矛盾專案（README vs LICENSE）
/license-risk-checker https://github.com/hanbyel0105/Diff-HMR

# 檢查本機專案目錄
/license-risk-checker /path/to/your/project
```

### 方式 2：搭配其他 LLM Agent 或自訂 Prompt

直接將 `SKILL.md` 的內容作為 System Prompt 或 Context 提供給任何支援 Markdown 的大語言模型（如 Claude 3.5 Sonnet、Gemini 1.5 Pro、GPT-4o），並提供目標 Repo 連結即可開始分析。

### 方式 3：人工查核對照

若需進行人工或內部法務審計，可直接打開 [授權檢查清單.md](授權檢查清單.md)，依五大階段逐項勾選並記錄核查證據。

---

## 真實範例報告（收錄 8 個真實公開案例）

本專案收錄了 8 份真實公開專案的完整檢查報告，詳見 [範例報告.md](範例報告.md)：

### 基礎案例（v1.0）
1. **`ivankovic/omnidiff`**：Hacker News 熱議之 AGPL-3.0-only 與商業雙重授權案例。
2. **`alchaincyf/huashu-bird-sticker`**：花叔鳥類貼紙庫自 PolyForm Shield 1.0.0 轉換至 CC BY 4.0 之歷史地雷與多重授權拆分。
3. **AI 模型特有地雷專題**：
   - `MiniMaxAI/MiniMax-H3`：社群協議中罕見之「排除美/歐/英/韓」地域限制與 2,000 萬美元營收門檻。
   - `Qwen/Qwen-Image-2.1`：GitHub Apache-2.0 與權重「研究專用商用禁止」授權脫鉤。

### 擴充案例（v1.1 新增）
4. **`meta-llama/Llama-3.1-8B`**：Llama 3.1 Community License 全文、700M MAU 門檻、AUP 禁止領域、"Built with Llama" 標記與下游模型開頭命名義務。
5. **`n8n-io/n8n`**：Sustainable Use License（Fair-code）限縮「內部業務目的」、嚴禁雲端工作流託管與競品嵌入、深度對比 AGPL 與 SSPL 之機制差異。
6. **`vercel/next.js`**：模組 3 遙測排查實案，原始碼層檢驗 Default-On Telemetry 預設開啟真相、回傳端點追蹤、關閉環境變數（`NEXT_TELEMETRY_DISABLED=1`）與本地終端除錯審核。
7. **`black-forest-labs/FLUX.1-dev`**：權重非商用（Non-Commercial License）與輸出圖片（Outputs）放行商用之權限分離、三階授權體系（schnell / dev / pro）對照、競品蒸餾禁令與反向賠償條款。
8. **`hanbyel0105/Diff-HMR`**：跨來源說法不一致實案，README 明文標註 CC BY-NC 但 LICENSE 檔案存放 MIT，直擊傳統自動化掃描工具只看 LICENSE 產生的假綠燈盲區，嚴格客觀並列不選邊。

---

## 法律免責聲明（Legal Disclaimer）

> [!CAUTION]
> **重要聲明**：  
> 本工具及其產出之所有報告、分析與清單僅供工程架構評估、商務決策初篩與技術參考，**不構成任何形式的正式法律意見、法律諮詢或律師-委託人關係（Attorney-Client Relationship）**。
> 智慧財產權法規與法院判例在不同司法管轄區（如台灣、美國、歐盟）各異。若面臨版權爭議、專利紛爭或重大商業併購投資，請務必諮詢合格智慧財產權專業律師。

---

## 授權方式（License）

本專案採用 [MIT License](LICENSE) 開源釋出。任何人皆可免費使用、修改、分發或整合至個人與商業專案中。
