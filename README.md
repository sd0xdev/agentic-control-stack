# Agentic Control Stack

> 用控制理論的視角，拆解 agentic coding system 為什麼會壞、怎麼讓它穩定收斂。

## 目錄

- [這是什麼？](#這是什麼)
- [五層架構](#五層架構)
- [文件目錄](#文件目錄)
- [核心概念速覽](#核心概念速覽)
- [適用場景](#適用場景)
- [Claude Code Skill](#claude-code-skill)
- [授權](#授權)

---

## 這是什麼？

當我們把 AI agent 放進軟體開發流程，最常遇到的問題不是「它不會寫 code」，而是「它寫出來的東西不可控」——修了 A 壞了 B、刪掉測試讓 CI 變綠、越改越歪直到整個 PR 不能看。

**Agentic Control Stack** 是一套教學框架，借用控制理論（control theory）的語言——sensor、effector、gain、damping、oscillation——來解釋這些問題的根因，並提供可操作的設計方法。

核心觀點：**設計 agentic system 的功夫不是把 loop 跑起來，而是把 loop「跑歪」的可能性壓到最低。**

---

## 五層架構

```
Layer 5  Human Governance      — 定義目標・策展知識・斷路器
Layer 4  Sensors & Effectors   — 看得到什麼・能動什麼・動到哪
Layer 3  Hierarchical Loops    — tactical → operational → strategic
Layer 2  Feedback Loop (MAPE)  — tests・runtime・logs・CI
Layer 1  Feedforward Gate      — static analysis・type check・lint
```

治理由上而下（L5 → L1），執行由下而上（L1 → L5）。

---

## 文件目錄

| 文件 | 內容 | 適合誰 |
|------|------|--------|
| [agentic-control-stack-lecture.md](agentic-control-stack-lecture.md) | **原理篇**：從 MAPE-K 迴路、前饋 vs 回饋、分層迴路、手眼系統、人的治理層到迴路病理學，附完整端到端案例與課後作業 | 想理解「為什麼」的人 |
| [agentic-control-stack-in-practice.md](agentic-control-stack-in-practice.md) | **實作篇**：以 [sd0x-dev-flow](https://github.com/sd0xdev/sd0x-dev-flow) plugin 為範例，逐層展示控制理論如何落地成可運行的系統 | 想知道「怎麼做」的人 |

建議閱讀順序：先讀原理篇建立框架，再讀實作篇看具體對照。

---

## 核心概念速覽

### MAPE-K 控制迴路

每個 agentic coding system 的核心都是一個 MAPE-K 迴路：

| 步驟 | 做什麼 |
|------|--------|
| **M**onitor | 跑測試、看 log、讀 error output |
| **A**nalyze | 推斷 root cause，做假設管理 |
| **P**lan | 產生修補策略 + 驗證計畫 |
| **E**xecute | 落地變更，回到 Monitor |
| **K**nowledge | 沉澱經驗，讓迴路不重蹈覆轍 |

### 三種經典失敗模式

| 病態 | 症狀 | 關鍵旋鈕 |
|------|------|----------|
| **Oscillation（振盪）** | 修好 A 壞 B，revert 來 revert 去 | diff budget + 記憶（K） |
| **Local Minimum（局部最小值）** | 靠 hack 讓測試變綠（刪 assertion、跳過測試） | 強化 oracle + invariant 檢查 |
| **Divergence（發散）** | 越改 diff 越大，偏離原始目標 | file allowlist + 斷路器 |

### 前饋 vs 回饋

- **前饋（Feedforward）**：在 Execute 之前用靜態分析擋掉確定性錯誤（型別、格式、規範）
- **回饋（Feedback）**：Execute 之後用 runtime 驗證行為性錯誤（邏輯、整合、效能）

設計原則：**能前移的錯誤全部前移**，讓 feedback loop 只處理「跑起來才知道」的問題。

---

## 適用場景

這套框架適用於任何需要設計或評估 agentic coding system 的場景：

- **課程教學**：原理篇附有課後作業、討論題、課堂活動設計
- **系統設計**：用五層架構作為 checklist，檢查你的 agentic system 是否每層都有對應的控制
- **故障診斷**：用病理學框架診斷 agent「跑歪」的根因，找到該調的旋鈕
- **團隊溝通**：用控制理論的詞彙精確描述問題（「它在 oscillation，因為 gain 太大」比「agent 不太穩」有用一百倍）

---

## Claude Code Skill

本專案附帶一個 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill，clone 下來後在專案目錄裡用 `/agentic-control-stack` 就能互動式地使用這套教學框架。

### 前置條件

- 已安裝 [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code/getting-started)（`npm install -g @anthropic-ai/claude-code`）
- 已完成 Claude Code 登入驗證

### 使用方式

```bash
git clone https://github.com/sd0xdev/agentic-control-stack.git
cd agentic-control-stack
claude  # 在專案根目錄啟動 Claude Code
```

在 Claude Code 中輸入 `/agentic-control-stack` 即可觸發 skill（需在專案根目錄啟動，Claude Code 會自動偵測 `.claude/skills/` 下的 skill）。你也可以直接用自然語言提問，例如：

- 「幫我講解 MAPE-K 迴路」
- 「什麼是迴路振盪？怎麼治？」
- 「生成一份完整的 agentic control stack 講義」
- 「用 sd0x-dev-flow 當範例，講解五層架構怎麼落地」

### 三種使用模式

| 模式 | 適用情境 | 說明 |
|------|---------|------|
| **講義生成** | 完整上課、生成教學素材 | 自動組合所有 reference，輸出連貫的純理論講義 |
| **互動問答** | 單一概念提問 | 只載入相關 reference，針對特定概念深入講解 |
| **原理 + 實作** | 進階學習、實際導入參考 | 理論搭配 [sd0x-dev-flow](https://github.com/sd0xdev/sd0x-dev-flow) 實作對照 |

完整的模式說明與主題對照表請參考 [SKILL.md](.claude/skills/agentic-control-stack/SKILL.md)。

---

## 授權

MIT