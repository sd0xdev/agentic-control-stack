# Agentic Control Stack: Five Layers

## Table of Contents
- [Overview](#overview)
- [Layer 1: Feedforward Gate](#layer-1-feedforward-gate)
- [Layer 2: Feedback Loop (MAPE)](#layer-2-feedback-loop-mape)
- [Layer 3: Hierarchical Loops](#layer-3-hierarchical-loops)
- [Layer 4: Sensors & Effectors](#layer-4-sensors--effectors)
- [Layer 5: Human Governance](#layer-5-human-governance)

## Overview

MAPE-K 只是迴路骨架；真正的 agentic coding system 要再補上「穩定性、前饋、分層、手眼、以及人的位置」。

---

## Layer 1: Feedforward Gate

### 概念
Feedforward（前饋）= 在還沒「跑壞」之前就把明顯錯誤擋掉，減少迴路迭代次數。

### 工具對應
- static analysis / type checking / lint / formatting / schema validation / pre-commit hooks
- 特色：不用等跑測試、甚至不用 runtime，就能判斷「一定錯」或「不符合規範」

### 與 Feedback 的關係

| | Feedforward | Feedback |
|---|---|---|
| 判斷時機 | Plan→Execute 之間 | Execute 之後 |
| 成本 | 低（靜態、快速） | 高（需要 runtime） |
| 能抓的錯 | 確定性的（型別、格式、規範） | 行為性的（邏輯、整合、效能） |

### 設計原則

> 把能前移的錯誤全部前移；讓 feedback loop 只處理「跑起來才知道」的問題。

落到 coding agent 的管線：
- Plan 之後先過 feedforward gates（type/lint/static rules）→ 不過就不要 Execute
- Execute 之後才進 feedback（tests/CI）→ 成本高，但處理真正行為層問題

**結論**：feedforward 做得越好，agent 看起來越像「天生就很會修」，因為它根本不會進到那些低級錯誤的迴圈。

---

## Layer 2: Feedback Loop (MAPE)

### 概念
Feedback（回饋）= 執行之後觀測結果，比對期望狀態，再決定下一步。跟 feedforward 互補：feedforward 擋得住確定性錯誤，feedback 處理「跑起來才知道」的行為問題。

### MAPE 在 Coding Agent 的對應

| MAPE 步驟 | Coding Agent 的行為 | 輸出 |
|---|---|---|
| **Monitor** | 跑測試、收 CI log、讀 runtime error | failing test 清單、stack trace、coverage diff |
| **Analyze** | 從 test failure 推 root cause 候選集；排列假設（H1/H2/H3） | 最可能的根因 + 最便宜的驗證實驗 |
| **Plan** | 決定修法：最小變更 vs 系統性重構、是否補測試、是否調設定 | patch 策略 + 「怎麼證明改對」 |
| **Execute** | apply patch → 立刻回到 Monitor，看是否更接近目標 | 變更 + 驗證結果 |

### 行為性錯誤的範例

Feedforward 抓不到、只有 feedback 能發現的問題：
- 邏輯錯誤：if 條件反了，型別正確但語義錯誤
- 整合問題：A 模組改了 API，B 模組還在用舊介面
- 效能退化：新 code 讓 response time 從 200ms 變 2s
- 環境依賴：本機跑得過、CI 裡 fail（環境差異）

### Knowledge (K) 的角色

K 是迴路的記憶體。沒有 K，agent 每一輪都從零開始，可能重踩已知的死路。

K 的具體內容：
- 錯誤簽名 → 常見修法的對照表
- 「這個方向試過了，失敗原因是 X」的 case log
- 團隊的 coding convention 和架構約束

詳細 MAPE-K 理論基礎見 [mape-k-foundation.md](mape-k-foundation.md)。

### Punchline

> Feedback loop 的品質不取決於「能不能修好」，而是「每一輪能把不確定性降低多少」。

---

## Layer 3: Hierarchical Loops

學生很容易把 MAPE-K 想成「一個大 loop 處理所有事」，但真正好用的 agentic coding system 是分層的：每層週期、粒度、oracle、K 都不同。

### 內層（Tactical，30 秒級）
- **粒度**：改一個函式/一個 if/一個參數 → 跑一個小驗證（單測子集合、lint、型別）
- **目標**：快速收斂、快速排除假設
- **K**：錯誤簽名→常見修法、剛剛試過的 patch、當前上下文（局部）

### 中層（Operational，10 分鐘級）
- **粒度**：一個 changeset / 一個 PR → 全套 verify（測試、靜態規則、風格、review gate）
- **目標**：完整性、可維護性、回歸風險控制
- **K**：PR 設計決策、驗證口徑、review checklist、風險點與回歸案例

### 外層（Strategic，跨天/跨專案）
- **粒度**：跨任務學習與治理（團隊規範、架構決策、長期 playbook）
- **目標**：一致性、可治理、長期收斂（避免每次都從零開始）
- **K**：架構 invariant、安全規則、最佳實務、事故後的規則沉澱

### Punchline

> 內層追速度，中層追完整性，外層追一致性與治理。

也很吻合「用管 junior 的方式管 AI」：你設的是不同層級的 checkpoint，不是盯每一步。

---

## Layer 4: Sensors & Effectors

如果要點破「為什麼同樣用差不多的 LLM，有的團隊 agent 很穩、有的團隊很鬧」，答案通常不在模型，而在手眼。

### Sensor（能看到什麼）

強的 sensor 不是「把一坨 raw text 丟給模型看」，而是能提供結構化訊號，直接降低 Analyze 的不確定性：

- 結構化 test output（哪個 case、哪個 assertion、哪個 fixture）
- 可解析 stack trace → blame 到檔案/函式/行號
- env fingerprint（版本、lockfile hash、feature flags）
- coverage / runtime trace / dependency graph
- 將 review feedback 或規範轉成可機器檢查的 signals

### Effector（能動什麼、被限制成什麼）

好的 effector 不是「能做一切」，而是「能做得精準、受控」：

- file allowlist / denylist（哪些檔可改）
- diff budget（單次改動上限）
- tool allowlist（format/codemod/migration 能不能用）
- 權限分級（內層只能改 app code，中層才可改依賴/設定，外層才可動 infra）

### Punchline

> 模型是大腦，但 sensor 和 effector 是眼和手；沒有好的手眼，大腦再強也是在黑箱裡亂摸。

### 產業觀察

大模型能力在收斂，差距在縮小；真正的系統差異在「看得見什麼、動得了什麼、動得多精準」。

---

## Layer 5: Human Governance

MAPE-K 的理想是 self-management、減少人介入，但在 coding agent 的現階段，人其實是「定義目標、治理迴路」的核心角色。

### 角色 1: Desired State 的定義者
- 寫 spec、寫測試、定義 acceptance criteria
- 定義什麼叫「對」、什麼叫「可接受」
- **最不可取代**：oracle 不夠強，迴路必定收斂到錯的地方

### 角色 2: Knowledge 的策展人
- 決定哪些規則進 K、哪些 pattern 值得沉澱
- 把一次性經驗變成可重用政策（policy）或案例（case）
- 本質：把個人經驗制度化，讓 agent 不是每次從零開始

### 角色 3: Loop 的斷路器（Circuit Breaker）
- 當振盪、發散、或陷入 hack 的局部最小值，人要介入：停、重設目標、收緊邊界、補強 oracle
- 也包含升級迴路：把問題從內層升到中層或外層處理

### Punchline

> 開發者的角色正在從「寫 code 的人」變成「設計與調校控制迴路的人」。
> 你不是在手動修程式，你是在調 sensor、effector、oracle 和 knowledge，讓 loop 自己收斂。
