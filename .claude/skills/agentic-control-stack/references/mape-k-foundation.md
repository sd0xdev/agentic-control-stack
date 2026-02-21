# MAPE-K Foundation

## Table of Contents
- [Overview](#overview)
- [Two Boxes](#two-boxes)
- [The Four Steps + K](#the-four-steps--k)
- [Punchline](#punchline)

## Why Control Theory?

為什麼要用控制理論來理解 agentic system？因為控制理論給我們一組**精確的詞彙**來描述 agent 的行為：sensor、effector、gain、damping、oscillation——這些詞讓我們能**診斷問題、調校系統**，而不是靠感覺說「agent 不太穩」。

如果你只有「它有時候會出錯」這種描述，你什麼都調不了。但如果你能說「它在 oscillation，因為 gain 太大且 K 太弱」，你就知道要加 diff budget 和記憶。

## Overview

MAPE-K 是自我管理控制迴路的標準描述：系統不是「會修 bug」，而是在做「把實際狀態拉回期望狀態」的調節。

## Two Boxes

- **Managed element**（被管理的對象）：程式碼、依賴、設定、環境、部署管線、測試、文件
- **Autonomic manager**（管理者）：跑 M-A-P-E 的那個 agent（或 agent + tooling）

## The Four Steps + K

MAPE 的四步很像工程師日常，但關鍵是：每一步都有明確的輸入/輸出，而且目標狀態通常由「可判定的訊號」定義（tests pass / build 成功 / lint clean / acceptance criteria 達成）。

### Monitor
跑測試、看 log、讀 error output；把雜訊整理成可用訊號。
- 輸出：failing tests 清單、stack trace、環境指紋、重現條件

### Analyze
從觀測推斷 root cause 的候選集；做假設管理（H1/H2/H3），找最便宜的排除實驗，把不確定性縮小。

### Plan
產生修補策略 — 不只是「改哪裡」，而是「怎麼證明改對」。
- 決定：最小變更 vs 系統性修補、是否補測試、是否調設定

### Execute
落地變更（apply patch / 改設定 / 補測試 / 重跑驗證），然後立刻回到 Monitor，看是否更接近目標狀態。

### Knowledge (K)
共享底座。把這次的失敗型態、有效/無效修法、規則與限制沉澱下來，避免下一輪重踩；也讓迴路能收斂而不是亂試。

## Punchline

> Agent 的「自癒」不是魔法，而是有 sensor、有 effector、有 oracle 的控制迴路。
