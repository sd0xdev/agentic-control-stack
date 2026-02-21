# Practice: sd0x-dev-flow 實作對照

## Table of Contents
- [五層映射總表](#五層映射總表)
- [Layer 1: Feedforward Gate 實作](#layer-1-feedforward-gate-實作)
- [Layer 2: Feedback Loop 實作](#layer-2-feedback-loop-實作)
- [Layer 3: Hierarchical Loops 實作](#layer-3-hierarchical-loops-實作)
- [Layer 4: Sensors & Effectors 實作](#layer-4-sensors--effectors-實作)
- [Layer 5: Human Governance 實作](#layer-5-human-governance-實作)
- [病態防禦對照表](#病態防禦對照表)
- [Sentinel 通訊協議](#sentinel-通訊協議)
- [雙層防禦架構](#雙層防禦架構)
- [設計模式摘要](#設計模式摘要)
- [Punchline](#punchline)

## 五層映射總表

| 控制層 | sd0x-dev-flow 元件 | 關鍵檔案 | 觸發條件 |
|--------|-------------------|---------|---------|
| **Feedforward** | `pre-edit-guard.sh`、`post-edit-format.sh`（prettier）、lint:fix | hooks/ | Edit/Write 工具執行前/後 |
| **Feedback (MAPE)** | auto-loop 強制序列：Edit → `/codex-review-fast` → fix → re-review → `/precommit` | rules/auto-loop.md | code 變更後自動 |
| **Hierarchical** | Inner(hooks 30s) → Mid(commands 10min) → Outer(rules 跨 PR) | hooks/ → commands/ → rules/ | 各層各自觸發 |
| **Sensors** | `analyze.js`(JSON)、`risk-analyze.js`(JSON)、Codex review(Markdown+sentinel) | skills/*/scripts/ | 各 skill 調用時 |
| **Effectors** | `allowed-tools` whitelist、file denylist、Codex `sandbox: read-only` | SKILL.md frontmatter、hooks/ | 工具呼叫時 |
| **Human Governance** | `rules/`(K 策展)、`codex-invocation.md`(獨立驗證)、`stop-guard.sh`(斷路器) | rules/、hooks/ | session 載入/停止時 |

## Layer 1: Feedforward Gate 實作

### pre-edit-guard.sh — Edit/Write 的守門員

每次 Claude 使用 Edit/Write 工具前：

```bash
# 阻擋敏感路徑（預設：.env、.git/）
if echo "$file_path" | grep -Eq '(\.env|\.git/)'; then
  exit 2  # 拒絕工具呼叫
fi
# 阻擋 shell metacharacter 注入
if [[ "$file_path" =~ [\;\&\|\`] ]]; then
  exit 2
fi
```

### post-edit-format.sh — 寫入即校正

Edit/Write 完成後立即執行 prettier：格式問題在發生的瞬間被修好，永遠不會進入 feedback loop。

### 對應原理

feedforward 做得越好，agent 越少進入低級錯誤的迴圈。表面上「天生就不犯錯」，實際上是很多錯根本沒機會發生。

## Layer 2: Feedback Loop 實作

### auto-loop 強制序列

```
Edit code → post-edit hook（標記 has_code_change、失效 review 狀態）
         → /codex-review-fast（Codex 獨立研究 git diff）
         → 有問題 → Fix → --continue threadId（re-review）
         → ✅ Ready → /precommit（lint+build+test）
         → ✅ All Pass → 可以停止
```

auto-loop 只強制 `/codex-review-fast` + `/precommit`（code）或 `/codex-review-doc`（doc）。`/verify`、`/next-step` 是可選的輔助工具。

### MAPE 對應

| MAPE | 實作 |
|------|------|
| Monitor | `/codex-review-fast`（Codex 獨立研究 git diff） |
| Analyze | Codex 獨立推斷 root cause（禁止餵結論） |
| Plan | Claude 根據 findings 決定修法 |
| Execute | Edit/Write 落地變更 → 立刻回 Monitor |
| K | `rules/`、`--continue threadId`、`.claude_review_state.json` |

### 禁止行為（防止「假收斂」）

| 禁止 | 理由 |
|------|------|
| 「聲明 ≠ 執行」 | 說「需要跑 review」但沒呼叫工具 → Monitor 被跳過 |
| 「摘要 ≠ 完成」 | 輸出漂亮摘要就停 → 沒有 oracle 驗證 |
| 修完問「要不要 re-review？」 | 迴路暫停等人 → 應自動繼續 |

### 退出條件

- `✅ All Pass` → 收斂
- `⛔ Need Human` → 斷路器
- `🔄 3 rounds on same issue` → 升級

## Layer 3: Hierarchical Loops 實作

| 層級 | 週期 | 元件 | 觸發 | K |
|------|------|------|------|---|
| Inner (Tactical) | 30s | hooks | 每次 Edit/Write/Bash | `.claude_review_state.json` |
| Mid (Operational) | 10min | `/codex-review-fast`、`/precommit`（強制）；`/verify`（可選） | auto-loop | review findings、threadId |
| Outer (Strategic) | 跨 PR | `rules/`、`/project-audit`、`/pr-review` | session 載入 / 手動 | 團隊規範、`.claude/CLAUDE.md` |

### 升級機制

| 路徑 | 觸發 | 實作 |
|------|------|------|
| Inner → Mid | 檔案修改 → review 失效 | `post-edit-format.sh` 設 `code_review.passed = false` |
| Mid → Outer | 3 rounds 未收斂 | auto-loop：報告 blocker → 交人 |
| Mid → Outer | 風險 High/Critical | `/risk-assess`：score ≥ 50 → REVIEW；≥ 75 → BLOCK |

## Layer 4: Sensors & Effectors 實作

### Sensors

| Sensor | 輸出格式 | 降低什麼不確定性 |
|--------|---------|----------------|
| `analyze.js`（16 啟發式） | 結構化 JSON：phase、findings、next_actions（含 confidence） | 「現在該做什麼」 |
| `risk-analyze.js`（3 維評分） | JSON：breaking_surface 45% + blast_radius 35% + change_scope 20% | 「改動有多危險」 |
| `post-tool-review-state.sh` | `.claude_review_state.json` | 「哪些步驟還沒做」 |
| Codex MCP（獨立研究） | Markdown + sentinel 字串（hook regex 解析） | 「程式碼品質是否達標」 |

sensor 分兩類：確定性腳本輸出 JSON，review 工具輸出 Markdown+sentinel 再由 hook regex 解析。兩者都比 raw text 好，但結構化程度不同。

### Effectors

| Effector | 控制什麼 |
|----------|---------|
| `allowed-tools` whitelist（SKILL.md frontmatter） | skill 執行時可用的工具 |
| file denylist（`pre-edit-guard.sh` + `GUARD_EXTRA_PATTERNS`） | 不可修改的檔案 |
| Codex `sandbox: read-only` + `approval-policy: never` | Codex 只能讀不能寫 |
| diff budget | 單次 patch 行數上限 |

### 安全控制

| 威脅 | Sensor | Effector | 斷路條件 |
|------|--------|----------|---------|
| Secret 洩漏 | `pre-edit-guard.sh`：`.env` + `.git/` 預設 | file denylist → exit 2 | 匹配 → 拒絕 |
| Shell 注入 | metacharacter regex | path 驗證 → exit 2 | 可疑路徑 → 拒絕 |
| Unsafe tool 呼叫 | `allowed-tools` 比對 | 未授權工具無法使用 | 嘗試 → 阻擋 |

未實作（未來方向）：lockfile hash gate、path-scope enforcement。

## Layer 5: Human Governance 實作

### Role 1: Goal Definer

| 實作 | 做什麼 |
|------|--------|
| `.claude/CLAUDE.md` | 定義 framework、test/lint/build command |
| `/project-setup` | 自動偵測 + 填寫 placeholder |
| tech-spec / issue | 定義 acceptance criteria（oracle 基礎） |

### Role 2: Knowledge Curator

| 實作 | 做什麼 |
|------|--------|
| `rules/`（10 條） | auto-loop、security、testing 等團隊規範 |
| `codex-invocation.md` | Codex 呼叫標準：禁止餵結論、強制獨立研究 |
| `/install-rules` | 把 K 持久化到專案 `.claude/rules/` |

### Role 3: Circuit Breaker

| 實作 | 觸發 | 效果 |
|------|------|------|
| `stop-guard.sh` STRICT | review/precommit 未通過 | exit 2 硬阻擋（預設 warn，需 opt-in） |
| `⚠️ Need Human` sentinel | 架構級變更、3 rounds 未收斂 | agent 停下交人 |
| `HOOK_BYPASS=1` | 人手動設定 | 緊急跳過所有檢查 |

### 獨立驗證（防 Confirmation Bias）

`codex-invocation.md` 禁止：

| 禁止 | 危險 |
|------|------|
| 餵 code：「這修法對嗎？」 | Codex 只看你給的，看不到你漏的 |
| 餵結論：「bug 在 X，確認？」 | 預設答案，Codex 不會挑戰 |
| 限制範圍：「只看 src/service/」 | 漏掉 middleware 問題 |

正確做法：給 Codex 完整專案存取，讓它自己 `git diff`、`grep`、`cat` 獨立發現問題。

## 病態防禦對照表

| 病態 | sd0x-dev-flow 防禦 | 實作元件 | 關鍵參數 |
|------|-------------------|---------|---------|
| Oscillation | 3 rounds 上限 + phase detection | `auto-loop.md`、`analyze.js` | MAX_ROUNDS=3 |
| Local Minimum | Codex 獨立研究（不接受餵結論） | `codex-invocation.md`、`fix-all-issues.md` | sandbox: read-only |
| Divergence | `allowed-tools` whitelist + git rules 禁直推 main | SKILL.md frontmatter、git-workflow.md | file allowlist |
| 過早停止 | stop-guard + 「聲明 ≠ 執行」原則 | `stop-guard.sh`、`auto-loop.md` | STOP_GUARD_MODE=strict |
| Confirmation Bias | 禁止餵結論、強制獨立研究 | `codex-invocation.md` | 執行 checklist |
| 狀態漂移 | post-edit hook 立即失效 review 狀態 | `post-edit-format.sh` | invalidate_review() |

## Sentinel 通訊協議

迴路元件之間用 sentinel 字串通訊：

| Sentinel | 意義 | 解析方式 |
|----------|------|---------|
| `✅ Ready` | Code review 通過 | hook regex（priority 2） |
| `✅ Mergeable` | Doc review 通過 | hook regex（需 `## Document Review` header，priority 1） |
| `## Overall: ✅ PASS` | Precommit 通過 | hook regex（priority 3） |
| `⚠️ Need Human` | 需人介入 | 僅行為層，hook 不解析 |

設計：anchored regex 為主判斷，unanchored + negative lookahead 為 MCP 輸出 fallback；priority routing 解決多義性。

## 雙層防禦架構

| 防禦層 | 機制 | 強制程度 |
|--------|------|---------|
| 行為層 | `auto-loop.md`：Edit 後同一回覆觸發 review | Soft（依賴 LLM 遵守） |
| 系統層 | `stop-guard.sh`：檢查 state file | 預設 warn；`STOP_GUARD_MODE=strict` 才硬阻擋 |

疊加效果：行為層被繞過時，strict mode 系統層仍然攔住。

### Stale-State 對賬

單向校正（只做 `true → false`）：
- 狀態檔 `has_code_change = true` 但 git status 無對應檔案 → 覆寫 false
- 反方向不做（可能是 pre-existing untracked files）

## 設計模式摘要

| 模式 | sd0x-dev-flow 做法 | 可遷移到任何 agentic system |
|------|-------------------|---------------------------|
| State file as truth | `.claude_review_state.json` | 結構化狀態檔 > 解析對話記錄 |
| Dual-layer defense | rules + hooks | 同約束兩種機制，一層被繞另一層仍有效 |
| Sentinel communication | `✅ Ready` / `⛔ Blocked` anchored regex | 可機器解析的標記 > 自然語言 |
| Independent second sensor | Codex 獨立研究 | 第二 reviewer 獨立工作 ≠ rubber stamp |
| One-way reconciliation | stop-guard `true → false` only | 狀態校正寧保守不誤報 |
| Confidence-based ranking | `analyze.js` next_actions sorted | sensor 帶信心分數，優先處理最確定的 |
| Escape hatch | `HOOK_BYPASS=1` | 永遠留緊急出口 |

## Punchline

> sd0x-dev-flow 是控制理論五層架構的完整實作範例——每個 hook、rule、skill、script 都能在理論框架中找到精確位置。理論告訴你「為什麼需要這一層」，實作告訴你「這一層長什麼樣子」。
