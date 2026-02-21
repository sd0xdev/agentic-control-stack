# Loop Pathology: Three Failure Modes

## Table of Contents
- [Overview](#overview)
- [1. Oscillation（振盪）](#1-oscillation)
- [2. Local Minimum（局部最小值）](#2-local-minimum)
- [3. Divergence（發散）](#3-divergence)
- [Diagnostic Summary Table](#diagnostic-summary-table)
- [Punchline](#punchline)

## Overview

學生最常卡的點是：「你講的迴路聽起來很好，那什麼時候它會壞掉？」
三個經典失敗模式，直接把抽象概念變成可診斷的工程現象。

---

## 1. Oscillation

### 症狀
修好 test1 壞 test2；再修 test2 又壞 test1；commit history 出現 revert-revert-revert。

### 根因（對應 MAPE-K）
- **K 太弱**：不記得「這條路剛試過」
- **Plan 的 gain 太大**：每次改動太猛、太廣
- **Monitor 訊號不穩**：flaky tests 或觀測不一致

### 治法（可操作的旋鈕）
| 旋鈕 | 做法 |
|---|---|
| 加阻尼（damping） | 限制單次 patch 範圍（diff budget / file allowlist） |
| 加記憶（K） | 把「嘗試過的 patch → 造成的失敗型態」寫成 case，下一輪直接禁止重踩 |
| 穩定觀測（Monitor） | 先做 flaky 判定、重跑一致性、固定 seed / env fingerprint |

---

## 2. Local Minimum

### 症狀
綠了，但靠 hack 綠（刪 assertion、硬塞 mock、跳過測試）。

### 根因
- Desired state 定義太弱（oracle 太弱），Monitor 回報「達標」其實是錯的達標

### 治法
| 旋鈕 | 做法 |
|---|---|
| 強化 oracle | 除了 tests pass，再加「不得刪 assertion」「coverage 不得下降」「關鍵 invariant 必須被測到」 |
| 把 review 也當 sensor | 將 code review checklist 形式化成可檢查規則（例如禁止跳過錯誤處理） |
| 測試設計當人類責任 | acceptance criteria 要能把 hack 擋掉，不然 loop 必然收斂到 hack |

---

## 3. Divergence

### 症狀
越修 diff 越大，最後和原本系統行為無關。

### 根因
- Plan 階段缺少「變更邊界」，effector 動作空間太大

### 治法
| 旋鈕 | 做法 |
|---|---|
| 硬邊界 | 可改檔案白名單、禁止改 infra/lockfile、禁止跨模組大重構 |
| 節流 | 大改動要升級到更外層迴路，不能在內層快速迴圈放飛 |
| 斷路器 | 當 diff 超過 budget / 連續 N 次未改善，強制停下交給人重新定義方向 |

---

## Diagnostic Summary Table

| 失敗模式 | 症狀特徵 | 核心根因 | 首要旋鈕 |
|---|---|---|---|
| Oscillation | revert-revert-revert | gain 太大 + K 太弱 | diff budget + 記憶 |
| Local minimum | hack 綠（刪 assertion） | oracle 太弱 | 強化 invariant 檢查 |
| Divergence | diff 爆炸 | 邊界太寬 | file allowlist + 斷路器 |

## Punchline

> 有迴路不代表會收斂；收斂要靠阻尼、oracle、以及邊界。

> 設計 agentic system 的功夫不是把 loop 跑起來，而是把 loop「跑歪」的可能性壓到最低。
