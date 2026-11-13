<!-- 產品／專案：密文智庫（EncryMind）（repo：AEGIS-Vault）｜法人／事業體：AEGIS（`aegis.expert`；正式法人名稱 [待確認]）｜日期：2026-11-13（Asia/Taipei）｜顯示修訂 2026-11-13 -->
# AEGIS-Vault
C2-AE-FAN 密文智庫（EncryMind）節點 for  高科技研發、生技、家族辦公室
# 密文智庫 (with USB)

> \*\*Repository ID\*\*: `C3-AE-FAN-F` (`aegis-fhe-ai-nas-app`)  
> \*\*系統定位\*\*: 地端私有知識庫 AI NAS 節點與去回雙向密文推論回吐系統  
> \*\*所屬事業體\*\*: AEGIS (`aegis.expert`) / 善 AI 聯盟 (Good AI Alliance)  
> \*\*維護角色\*\*: Dev Lead: `dev-aegis-fhe-ai-nas-app@aegis.expert` | QA Lead: `qa-aegis-fhe-ai-nas-app@aegis.expert`

\---

## 1\. 產品概述與 S0 單向輸入加密架構（Legacy）的差異

本專案旨在解決高價值知識產權（專利配方、製程參數、法律與醫療特種專家資產）在 AI 推論過程中的洩漏風險。

### 1.1 與 S0 單向輸入加密架構（Legacy）的本質差別

|比較維度|S0 單向輸入加密架構 (Legacy)|本系統：密文智庫 (with USB)|
|-|-|-|
|**加密覆蓋範疇**|**單向（HE-in only）**：主要保護 Prompt 輸入端，推論完成後以明文流式回傳。|**雙向（Bi-directional FHE Pipeline）**：去程密文、推論受控、回程密文。|
|**金鑰同源性**|無法保證回答能以客戶端相同的對話金鑰（Session Key）封裝回傳。|**嚴格對稱封閉**：模型回答必須依客戶端提供的同一把會話金鑰密文封裝後始得回吐。|
|**運算執行實體**|依賴遠端公有雲或外部閉源 LLM API。|**地端專屬硬體**：AI NAS 本地運算節點（Local SLM / LLM）實體私有化。|
|**資料落盤與讀出**|本地不具備大型儲存與知識庫，僅作轉發代理。|**大容量安全儲存 + 向量儲存**：領域知識常駐本機，且正式讀取路徑完全拒絕明文提取。|
|**增量核心架構**|無回吐層概念，接收明文 Token 即送前端。|**地端模型回吐層 (Model Cipher Egress Layer)**：攔截生成 Token 並進行動態密文封裝。|

註：HEShell 採 L0–L3 分級，不以 S0–S3 評級；S0 指架構類型，不指特定產品。

\---

## 2\. 嚴格技術分級與安全性邊界 (Security Boundaries: S0 - S3)

為了防止開發團隊混淆不同強度的密碼學保證與行銷術語，本專案嚴格定義四級技術邊界。未達標前**嚴禁跨級宣稱**：

```
\[S0: 輸入加密/回答明文] ---> Legacy 架構（非本專案範圍，禁止進入本倉庫代碼庫）
          │
\[S1: 地端受控執行 (v1 GA)] --> 密文輸入 -> 地端隔離邊界解密/推理 -> 回吐層密文封裝 -> 密文輸出
          │
\[S2: 密文域直接推論 (Target)] -> 密文輸入 -> 密文計算核心 (無明文暴露點) -> 密文輸出 (研發中)
          │
\[S3: 任意雲端大模型密文互動] -> 雲端即時雙向 FHE 推論 (超出 2026 年工程現實，明確排除)
```

### S0：單向輸入加密架構（Legacy）

* **特徵**：Prompt 加密送出，模型產生明文回答直接串流回呼叫端。
* **本專案狀態**：**嚴格廢棄**。本倉庫程式碼若出現明文回吐至傳輸層者，QA 閘門直接判定阻斷。

註：HEShell 採 L0–L3 分級，不以 S0–S3 評級；S0 指架構類型，不指特定產品。

### S1：地端受控執行與雙向密文封裝（v1.0 交付基準）

* **特徵**：

  1. 提問端（Client）以 USB 派生之臨時對話金鑰 $K\_{session}$ 加密 Prompt 送入 NAS。
  2. 請求在地端安全邊界（Isolated Local Runtime / Enclave / Container with IPC control）內解封，並調度地端 SLM/LLM 進行本地推論與 RAG 檢索。
  3. **核心增量（回吐層）**：模型生成的每個 Token 或完整 Response，在離開地端隔離邊界前，必須由回吐層以同一把 $K\_{session}$ 重新封裝成密文。
  4. 傳輸層、儲存層、日誌層完全不可見明文答案。
* **邊界宣告**：此階段在推論運算熱路徑中存在受控的短暫記憶體解密點，**不得對外宣稱為純 S2 數學密文運算**。

### S2：密文域同態推論（S2 Target Branch / Research Preview）

* **特徵**：模型權重與計算圖直接在密文上進行同態加乘運算（Homomorphic Evaluation via CKKS/TFHE），推論過程全程不解密，直接輸出同態密文。
* **工程現實說明（2026 現狀）**：端到端 LLM 密文推論延遲仍以數十秒至數分鐘計（約 $10^2$ 至 $10^4$ 倍開銷）。
* **本專案狀態**：僅作為特定固定 Kernel（如機密評分、私密相似度比對、私密檢索 PIR）之實驗模組；**未通過專屬效能與正確性測試前，不得標記為 GA**。

### S3：任意雲端大模型雙向密文互動（Out of Scope）

* **特徵**：在第三方不可信公有雲的大型 LLM（如 70B+）上直接進行即時互動式雙向純密文推理。
* **本專案狀態**：**明確排除於 v1 規格外**，禁止在代碼架構中預留對雲端明文 API 的回退（Fallback）路徑。

\---

## 3\. 系統架構與關鍵組成元件

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Client Tier (Terminal)                          │
│  ┌──────────────────────┐              ┌────────────────────────────┐  │
│  │  USB Hardware Key    │──────(插拔)───►│ Client UI / Zero-RAM Cache │  │
│  │  (Session Key Holder)│              │ (Only decrypts while USB in)│  │
│  └──────────────────────┘              └─────────────┬──────────────┘  │
└──────────────────────────────────────────────────────┼─────────────────┘
                                                       │ Encrypted Query /
                                                       │ Encrypted Response
┌──────────────────────────────────────────────────────┼─────────────────┐
│                      AI NAS Appliance Node (Local)   │                 │
│  ┌───────────────────────────────────────────────────┼──────────────┐  │
│  │ Ingress API Gateway (TLS + Payload Verification)  │              │  │
│  └───────────────────────────┬───────────────────────┴──────────────┘  │
│                              │ Encrypted Payload                       │
│  ┌───────────────────────────▼──────────────────────────────────────┐  │
│  │ S1 Local Controlled Enclave / Execution Pipeline                  │  │
│  │                                                                   │  │
│  │   ┌─────────────────────┐          ┌──────────────────────────┐   │  │
│  │   │  Storage Controller │          │ Local Engine             │   │  │
│  │   │  (AES-XTS at Rest)  │          │ (SLM / LLM + Local RAG)  │   │  │
│  │   │  \*No Plaintext Read │          └────────────┬─────────────┘   │  │
│  │   └──────────┬──────────┘                       │ Raw Tokens      │  │
│  │              │ Protected Vector Read            ▼                 │  │
│  │              └──────────────────────► ┌──────────────────────┐    │  │
│  │                                       │ Local Cipher Egress  │    │  │
│  │                                       │ (回吐層 - $K\_{session}$)│    │  │
│  │                                       └──────────┬───────────┘    │  │
│  └──────────────────────────────────────────────────┼────────────────┘  │
│                                                     ▼ Encrypted Egress  │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Append-Only Audit Log (Opaque Hashes Only, Zero Plaintext)       │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### 元件 1：AI NAS 本地儲存與推論載體

* **儲存子系統**：提供大容量 NVMe/SATA 儲存池。正式讀取通道必須受存取控制（ACL）與信封加密保護。領域知識（製程、配方、研究文獻）以靜態加密存放，禁止任何未授權使用者透過 SMB/NFS/WebDAV 讀出明文。
* **本地運算引擎**：本機搭載 x86/ARM 神經網路加速或 GPU 單元，獨立運行地端 SLM（Small Language Model，如 3B–8B 級別量化模型）或專用推論引擎。

### 元件 2：USB 硬體金鑰載體

* **功能定位**：非一般 Mass Storage 隨身碟，而是對話加密金鑰載體（支援 FIDO2/WebAuthn/Secure Element 或專屬 Token）。
* **生命週期**：每次工作階段由 USB 協同協商生成唯一會話金鑰 $K\_{session}$。
* **實體防護約束**：**USB 在位時前端方可解密顯示；USB 拔除，前端暫存與記憶體明文即刻抹除（Zero-Trace Destruction）**。

### 元件 3：地端模型回吐層 (Model Cipher Egress Layer) —— *核心增量*

* **架構位置**：介於地端模型推論引擎輸出端與 NAS 傳輸層之間。
* **執行邏輯**：

  1. 攔截模型推論所產生的 Output Context 或 Token Stream。
  2. 使用呼叫端本次對話之 $K\_{session}$ 進行流式（Stream）或區塊（Block）密文打包。
  3. 密文組包完成後，才允許跨越邊界進入網路上傳通道回到客戶端。
  4. 任何未經回吐層加密之資料流，網路層核心強制斷流。

\---

## 4\. 領域知識保護與授權架構 (Domain IP Authorization)

```
\[領域專家/資料擁有者]
       │
       ▼ (注入獨家配方/製程機密/專利文獻)
\[NAS 知識資料庫 (靜態密文加密)]
       │
       ├──► \[金鑰授權許可清單 (ACL Matrix)]
       │         │
       │         ├──► \[持有合法 USB 金鑰之授權人員 A] ──► 成功提問 ──► 取得回吐密文 ──► 解密可讀
       │         │
       │         └──► \[未持有合法金鑰之外部人員 B]   ──► 遭拒絕 / 僅能截獲隨機密文 ──► 永遠無法解出
       │
       └──► \[NAS 系統管理員 / 網路側聽者] ──────────► 儲存與傳輸全密文 ──► 無法窺探智財原文
```

1. **核心資產保護目標**：

   * 專家研究成果、獨家技術配方、關鍵製程資料、商業併購條款，均可藉由獨立派發的金鑰予以切割與保護。
2. **零信任防禦 (Zero-Trust Retrieval)**：

   * 僅有具備該領域對應權限之 USB 金鑰持有者，方能向該知識命名空間（Namespace）發起有效檢索。
   * 未獲授權者，即使取得 NAS 作業系統 Root 權限、實體抽出硬碟、或自區域網路側錄網路封包，皆無法解出任何可讀答案。

\---

## 5\. 系統強制行為規範 (Mandatory Behavioral Contracts)

所有模組實作與 PR 必須滿足以下行為準則：

* \[x] **REQ-BEH-01 (密文入鏈)**：所有提問 Prompt 必須在客戶端封裝為密文，以密文形式抵達 NAS 推論鏈，入口網關嚴禁接收明文 Prompt。
* \[x] **REQ-BEH-02 (密文回吐)**：推論答案必須由地端回吐層以同一會話金鑰加密封裝，回傳封包載荷（Payload）為純密文。
* \[x] **REQ-BEH-03 (在位顯示與離線清除)**：客戶端僅在 USB 實體插置狀態下於記憶體內解密呈現；偵測到 USB 拔除訊號，必須在 $\\le 200\\text{ ms}$ 內強制覆寫 Client 顯示記憶體與 Session 變數。
* \[x] **REQ-BEH-04 (NAS 原文封鎖)**：NAS 檔案系統、向量資料庫與快取層，在正式對外服務模式下，不可提供任何明文 Export 或無防護 Dump 介面。
* \[x] **REQ-BEH-05 (無權限金鑰阻斷)**：持未授權金鑰發起查詢時，閘道器必須直接拒絕；即使偽造驗證進入推論，輸出層亦不得以有效金鑰重新加密，呼叫端永遠無法解出明文。
* \[x] **REQ-BEH-06 (零原文日誌 - Zero-Plaintext Logging)**：系統日誌（Syslog、Access Log、Trace Log、Audit Trail）僅允許紀錄會話 UUID、時間戳、Token 計數、耗時與密文雜湊，**嚴禁記錄任何 Prompt、Context 或 Answer 原文**。

\---

## 6\. QA 驗收標準與測試矩陣 (Acceptance Test Matrix)

本章節為 QA 團隊開箱與交付閘門（QA Gates）之標準依據，測試案例未全部通過前不得放行發布。

### 6.1 測試矩陣 (Test Cases)

|測試用例 ID|測試項目|測試步驟|預期結果 (Pass Criteria)|嚴重等級|
|-|-|-|-|-|
|**TC-SEC-001**|去回雙向密文驗證|在 NAS 網路介面進行封包側錄（Packet Sniffing），送出測試查詢並攔截回應。|請求封包與回傳封包載荷熵值（Entropy）接近隨機，搜尋無任何明文字串。|**Blocker**|
|**TC-SEC-002**|會話金鑰同一性檢驗|客戶端以 $K\_{session}$ 提問，驗證回傳密文能否由同一 $K\_{session}$ 於本地成功解密。|密文成功解密出正確回覆，驗證去回金鑰對應無誤。|**Blocker**|
|**TC-SEC-003**|USB 拔除瞬時抹除|於推論答案顯示中途或完成後，拔出 USB 金鑰。|前端畫面立即空白或鎖定，檢查前端行程記憶體，無可讀答案字串殘留。|**Critical**|
|**TC-SEC-004**|未授權金鑰提問攔截|使用未經授權之 USB 金鑰發起特定領域知識查詢。|回傳 `403 Forbidden` 或不可解之亂碼密文，日誌標註授權失敗事件。|**Blocker**|
|**TC-SEC-005**|本地儲存路徑防禦|檢查 NAS 本地 RAG 索引目錄與磁碟空間。|無明文檔案索引，未授權路徑直接讀取失敗。|**Critical**|
|**TC-SEC-006**|稽核日誌原文防護|搜尋 `/var/log` 及所有 Container stdout/stderr。|正則表達式掃描確定零 Prompt、零 Context、零 Answer 洩漏。|**Blocker**|
|**TC-PERF-001**|S1 推論時延驗收|測量客戶端發出密文到接收完整密文的時間（TTFT 與 Total Latency）。|地端 SLM 本地推論在可接受時間窗口內完成，回吐層開銷（Overhead）$\\le 8%$。|**Major**|

> \*\*驗收準則重要宣告\*\*：  
> 2026 年純密文計算驗收標準為「\*\*去回皆密文、USB 在位解密、授權才可存取\*\*」，而非追求百毫秒級的雲端公共聊天體驗。系統穩定性與資訊邊界完整性為第一優先級。

\---

## 7\. 開發者快速上手與驗證 (Developer Quickstart)

### 7.1 本地模擬環境需求

* **OS**: Linux (Ubuntu 24.04 LTS / Debian 12) 或專屬 NAS OS (QTS/DSM/TrueNAS SCALE 容器環境)
* **硬體**: x86\_64 CPU (支援 AVX-512 / AMX) 或專屬邊緣 NPU/GPU
* **依賴**: Docker 26+, OpenFHE 1.5+ (C++17 runtime), Python 3.11+ / Rust 1.78+
* **USB 測試載體**: 支援 WebAuthn/CTAP2 協議之標準金鑰裝置或模擬軟體 Token

### 7.2 構建與啟動

```bash
# 1. 複製倉庫
git clone https://github.com/aegis-expert/aegis-fhe-ai-nas-app.git
cd aegis-fhe-ai-nas-app

# 2. 檢驗環境安全設定
make security-lint

# 3. 啟動 S1 地端受控執行環境 (含地端 SLM 與模型回吐層)
docker compose -f deployments/docker-compose.s1.yml up -d --build

# 4. 執行去回密文端到端自我檢驗 (E2E Sanity Suite)
pytest tests/e2e/test\_bidirectional\_cipher\_pipeline.py -v
```

### 7.3 開發驗證指令 (Verification Commands)

```bash
# 驗證日誌是否存在明文外洩 (違規即退件)
./scripts/audit-logs-leak-check.sh

# 執行 USB 拔除模擬訊號測試
python scripts/test\_usb\_hotplug\_eviction.py --device-id SIM\_TOKEN\_01
```

\---

## 8\. 代碼擁有者與變更規範 (CODEOWNERS)

依據 PMO 規範，本倉庫所有關於密鑰管理、回吐層與安全邊界之 PR 必須具備雙簽審核：

* `src/egress\_cipher/` (地端回吐層): `@dev-aegis-fhe-ai-nas-app` `@qa-aegis-fhe-ai-nas-app`
* `src/usb\_enclave/` (USB 金鑰生命週期): `@dev-aegis-fhe-ai-nas-app` `@security-admin`
* `tests/acceptance/` (QA 驗收測試): `@qa-aegis-fhe-ai-nas-app`

---

## 變更註記（2026-11-13，Asia/Taipei）

依使用者 2026-11-13 22:20 指示：將標題中的「前代」改為「S0 單向輸入加密架構（Legacy）」；一併移除「Generational Paradigm Shift」，以避免世代含義。此兩處「前代」均指 S0 架構，而非 HEShell（神盾殼）。本 README 先前未標草稿版本號，本次僅作標題用語修正，不新編版本號。仍待人工審核。
