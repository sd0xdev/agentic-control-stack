# Diagrams for Teaching

Mermaid diagrams ready to embed in lectures. Each diagram uses mermaid code blocks.

## Diagram 1: MAPE-K Loop

```mermaid
graph LR
    M[Monitor<br/>跑測試/看 log] --> A[Analyze<br/>推斷 root cause]
    A --> P[Plan<br/>修補策略]
    P --> E[Execute<br/>落地變更]
    E --> M
    K[(Knowledge<br/>K)] --- M
    K --- A
    K --- P
    K --- E
    ME[Managed Element<br/>程式碼/環境/設定] -.->|observes| M
    E -.->|modifies| ME
```

## Diagram 2: Five-Layer Agentic Control Stack

```mermaid
graph TB
    subgraph "Agentic Control Stack"
        L5["<b>Layer 5: Human Governance</b><br/>定義目標 · 策展 K · 斷路器"]
        L4["<b>Layer 4: Sensors & Effectors</b><br/>看得到什麼 · 能動什麼 · 能動到哪"]
        L3["<b>Layer 3: Hierarchical Loops</b><br/>tactical → operational → strategic"]
        L2["<b>Layer 2: Feedback Loop (MAPE)</b><br/>tests · runtime · logs · CI"]
        L1["<b>Layer 1: Feedforward Gate</b><br/>static analysis · type check · lint · pre-commit"]
    end
    L5 --> L4
    L4 --> L3
    L3 --> L2
    L2 --> L1
    style L5 fill:#f9e2af,stroke:#333
    style L4 fill:#a6e3a1,stroke:#333
    style L3 fill:#89b4fa,stroke:#333
    style L2 fill:#cba6f7,stroke:#333
    style L1 fill:#f38ba8,stroke:#333

    L5 -.- note1["← 治理方向（上→下）"]
    L1 -.- note2["← 執行方向（下→上）"]
    style note1 fill:none,stroke:none,color:#666
    style note2 fill:none,stroke:none,color:#666
```

> 注意：上層治理下層（規則與約束向下傳遞），但管線執行順序是從底層（Feedforward）往上走。

## Diagram 3: Feedforward vs Feedback Pipeline

```mermaid
flowchart LR
    CODE[Code Change] --> FF{Feedforward Gate}
    FF -->|Pass| EXEC[Execute]
    FF -->|Fail| FIX1[Fix & Retry]
    FIX1 --> FF
    EXEC --> FB{Feedback Loop}
    FB -->|Pass| DONE[Done ✓]
    FB -->|Fail| FIX2[Analyze & Fix]
    FIX2 --> FF
    style FF fill:#f38ba8,stroke:#333
    style FB fill:#cba6f7,stroke:#333
```

## Diagram 4: Hierarchical Loops

```mermaid
graph TB
    subgraph Strategic["Strategic（跨天/跨專案）"]
        S_K["K: 架構 invariant · 安全規則 · playbook"]
        S_G["目標: 一致性與治理"]
    end
    subgraph Operational["Operational（10 分鐘級）"]
        O_K["K: PR 決策 · review checklist · 風險點"]
        O_G["目標: 完整性"]
    end
    subgraph Tactical["Tactical（30 秒級）"]
        T_K["K: 錯誤簽名 · 局部上下文"]
        T_G["目標: 速度"]
    end
    Strategic -->|治理規則下放| Operational
    Operational -->|驗證標準下放| Tactical
    Tactical -->|升級大改動| Operational
    Operational -->|沉澱學習| Strategic
    style Strategic fill:#f9e2af,stroke:#333
    style Operational fill:#89b4fa,stroke:#333
    style Tactical fill:#a6e3a1,stroke:#333
```

## Diagram 5: Loop Pathology

```mermaid
graph LR
    subgraph Oscillation["振盪"]
        O1[Fix test1] --> O2[Break test2]
        O2 --> O3[Fix test2]
        O3 --> O4[Break test1]
        O4 --> O1
    end
    subgraph LocalMin["局部最小值"]
        LM1[All green ✓] --> LM2["But: deleted assertions<br/>skipped tests<br/>forced mocks"]
    end
    subgraph Diverge["發散"]
        D1["Patch 1<br/>+10 lines"] --> D2["Patch 2<br/>+50 lines"]
        D2 --> D3["Patch 3<br/>+200 lines"]
        D3 --> D4["Rewrite everything"]
    end
    style Oscillation fill:#f38ba8,stroke:#333
    style LocalMin fill:#f9e2af,stroke:#333
    style Diverge fill:#fab387,stroke:#333
```

## Diagram 6: Human Governance Positions

```mermaid
graph TB
    H1["<b>Role 1: Goal Definer</b><br/>spec · tests · acceptance criteria"]
    H2["<b>Role 2: Knowledge Curator</b><br/>rules → K · patterns → policy"]
    H3["<b>Role 3: Circuit Breaker</b><br/>stop · reset · tighten boundaries"]
    LOOP["MAPE-K Loop"]
    H1 -->|defines desired state| LOOP
    H2 -->|curates K| LOOP
    H3 -->|interrupts when diverging| LOOP
    style H1 fill:#f9e2af,stroke:#333
    style H2 fill:#a6e3a1,stroke:#333
    style H3 fill:#f38ba8,stroke:#333
```

## Diagram 7: Full System Overview (Sequence)

```mermaid
sequenceDiagram
    participant Human as Human Governance
    participant Agent as Autonomic Manager
    participant FF as Feedforward Gate
    participant FB as Feedback Loop
    participant Code as Managed Element
    participant K as Knowledge

    Human->>Agent: Define desired state + constraints
    Human->>K: Curate rules & policies

    loop MAPE-K Cycle
        Agent->>Code: Monitor (observe state)
        Code-->>Agent: Signals (tests, logs, metrics)
        Agent->>K: Query for prior cases
        K-->>Agent: Relevant patterns
        Agent->>Agent: Analyze (infer root cause)
        Agent->>Agent: Plan (patch strategy)
        Agent->>FF: Pass through feedforward gates
        alt Gate fails
            FF-->>Agent: Reject (type/lint/static error)
            Agent->>Agent: Fix & retry plan
        else Gate passes
            FF-->>Agent: Approve
            Agent->>Code: Execute (apply patch)
            Agent->>FB: Run feedback (tests/CI)
            alt Tests fail
                FB-->>Agent: Failure signals
            else Tests pass
                FB-->>Agent: Success
                Agent->>K: Record learnings
            end
        end
    end

    alt Pathology detected
        Agent-->>Human: Escalate (oscillation/divergence/hack)
        Human->>Agent: Reset direction / tighten boundaries
    end
```
