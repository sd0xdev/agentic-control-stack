---
name: agentic-control-stack
description: >
  Agentic Control Stack 教學框架 — 用控制理論視角解釋 agentic coding system 的設計原理。
  涵蓋五層架構（Feedforward Gate / Feedback Loop / Hierarchical Loops / Sensors & Effectors / Human Governance）、
  迴路病理學（振盪 / 局部最小值 / 發散）、MAPE-K 基礎，並附 sd0x-dev-flow 完整實作對照。
  觸發時機：(1) 使用者要求講解 agentic system 設計原理 (2) 討論 AI coding agent 的控制迴路
  (3) 教學或上課需要展示 agentic control stack (4) 討論 MAPE-K、feedback loop、feedforward、
  sensor/effector、human-in-the-loop 等概念 (5) 需要生成教學講義或圖表
  (6) 詢問控制理論如何落地實作 (7) 想看 sd0x-dev-flow 的實作對照或設計模式
  (8) 討論 sentinel 通訊、雙層防禦、auto-loop、hook/rule 設計
---

# Agentic Control Stack

以控制理論視角解釋 agentic coding system 的教學框架。

## 模式選擇

| 情境 | 選擇 |
|---|---|
| 完整上課 / 生成講義 | 模式一（純理論） |
| 單一概念提問 | 模式二（互動問答） |
| 進階學習 / 導入參考 / 理論+實作完整講義 | 模式三（原理+實作） |

## 三種模式

### 模式一：講義生成 — 生成完整教學素材（純理論）

依序讀取以下 references 並組合成連貫講義：
1. [references/mape-k-foundation.md](references/mape-k-foundation.md) — MAPE-K 基礎（含 motivation bridge）
2. [references/five-layers.md](references/five-layers.md) — 五層架構
3. [references/loop-pathology.md](references/loop-pathology.md) — 迴路病理學
4. [references/walkthrough.md](references/walkthrough.md) — 端到端貫穿案例（修復 API 500 Error）
5. [references/diagrams.md](references/diagrams.md) — Mermaid 圖表（嵌入對應章節）
6. [references/exercises.md](references/exercises.md) — 附加為課後作業

講義組裝規則：
- 建議章節順序：MAPE-K → 前饋 vs 回饋 → 分層迴路 → 手眼系統 → 人的治理 → 病理學（收尾整合）
- 每節必須附上 **punchline**（一句話總結）
- Mermaid 圖表嵌入對應章節位置
- 使用台灣繁體中文、台灣慣用語，技術名詞保留英文
- 若使用者指定子集主題，只包含該部分

### 模式二：互動問答 — 針對特定概念深入講解

使用者問到特定層級或概念時：
1. 判斷哪個 reference 檔案包含答案（見下方主題對照表）
2. 只讀取該檔案
3. 搭配範例說明，再主動提供進階探索或圖表選項

### 模式三：原理 + 實作 — 理論搭配 sd0x-dev-flow 實作對照

在理論基礎上疊加真實實作範例，適合進階學習或實際導入參考。

額外讀取：[references/practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) — sd0x-dev-flow 五層映射、sentinel 通訊、雙層防禦、設計模式

組裝策略：
- 每個理論章節後穿插對應的實作段落（例：講完 Feedforward 原理 → 接 `pre-edit-guard.sh` 實作）
- 實作段落引用 practice reference 中的程式碼片段與對照表
- 收尾加上「設計模式摘要」與「病態防禦對照表」作為實戰 checklist
- 可單獨使用（只輸出實作對照）或合併模式一（理論 + 實作完整講義）

## 主題對照表

| 主題 | 理論 Reference | 實作 Reference |
|---|---|---|
| MAPE-K 基礎 | [mape-k-foundation.md](references/mape-k-foundation.md) | — |
| 前饋層 (Feedforward) | [five-layers.md](references/five-layers.md) §Layer 1 | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Layer 1 |
| 回饋層 (Feedback / MAPE) | [five-layers.md](references/five-layers.md) §Layer 2 | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Layer 2 |
| 分層迴路 (Hierarchical) | [five-layers.md](references/five-layers.md) §Layer 3 | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Layer 3 |
| 手眼系統 (Sensor/Effector) | [five-layers.md](references/five-layers.md) §Layer 4 | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Layer 4 |
| 人的治理層 (Human) | [five-layers.md](references/five-layers.md) §Layer 5 | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Layer 5 |
| 振盪 / 局部最小值 / 發散 | [loop-pathology.md](references/loop-pathology.md) | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §病態防禦對照表 |
| sentinel 通訊協議 | — | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §Sentinel 通訊協議 |
| 雙層防禦架構 | — | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §雙層防禦架構 |
| 設計模式摘要 | — | [practice-sd0x-dev-flow.md](references/practice-sd0x-dev-flow.md) §設計模式摘要 |
| 圖表 | [diagrams.md](references/diagrams.md) | — |
| 端到端案例 | [walkthrough.md](references/walkthrough.md) | — |
| 練習題 / 設計 Checklist | [exercises.md](references/exercises.md) | — |

## 速查：五層一句話

1. **前饋層** — 把能前移的錯誤全部前移；讓 feedback loop 只處理「跑起來才知道」的問題
2. **回饋層 (MAPE)** — 觀測→分析→計畫→執行→再觀測；把實際狀態拉回期望狀態
3. **分層迴路** — 內層追速度，中層追完整性，外層追一致性與治理
4. **手眼系統** — 模型是大腦；sensor 和 effector 是眼和手；沒有好的手眼，大腦再強也是在黑箱裡亂摸
5. **人的治理層** — 開發者從「寫 code 的人」變成「設計與調校控制迴路的人」

## 核心金句

> 設計 agentic system 的功夫不是把 loop 跑起來，而是把 loop「跑歪」的可能性壓到最低。
