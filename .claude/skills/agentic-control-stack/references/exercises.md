# Exercises & Checklist

## Design Checklist

If designing an agentic coding loop from scratch, answer at least these questions:

1. **Oracle**: 我的 oracle 是什麼？除了 tests pass 還有哪些 invariants？
2. **Feedforward**: 我的 feedforward gates 有哪些？能不能在 Execute 前擋掉 60% 的錯？
3. **Pathology detection**: 我怎麼偵測振盪 / 局部最小值 / 發散？各自的 circuit breaker 條件是什麼？
4. **Sensors**: 我有哪些 sensor 能提供結構化訊號？哪些只是 raw text？
5. **Effectors**: 我的 effector 動作空間有多大？有沒有 diff/file budget？
6. **Hierarchy**: 迴路分幾層？每層的 K 是什麼？誰負責維護？
7. **Human role**: 人在什麼地方介入？介入的門檻與成本是什麼？

---

## Discussion Questions

### Q1: Feedforward vs Feedback boundary
你的團隊目前有哪些檢查是在 feedback 層做的，但其實可以前移到 feedforward？
舉出至少兩個可以前移的例子，並估算前移後能省掉多少 CI 時間。

### Q2: Diagnose a pathology
場景：一個 coding agent 在修一個 API endpoint 的 bug，commit history 如下：
1. Fix null check in handler
2. Revert: null check breaks other test
3. Add null check with different approach
4. Revert: still breaks
5. Disable flaky test
6. Fix passes

這裡出現了哪些病理？分別是什麼根因？你會怎麼調系統旋鈕來防止？

### Q3: Sensor quality
比較以下兩種 test failure output 給 agent 的品質：
- (A) `FAIL: 3 tests failed`
- (B) `FAIL: test_user_login (test_auth.py:42) — AssertionError: expected status 200, got 401. Fixture: mock_db with user role=admin`

哪個讓 Analyze 步驟更容易？為什麼？如果你只能改一件事來提升 sensor 品質，你會改什麼？

### Q4: Hierarchical loop design
你的團隊想讓 AI agent 協助日常 bug fix。請為以下三層各設計一個具體的「升級條件」：
- Tactical → Operational: 什麼時候小修補應該升級為 PR 級 review？
- Operational → Strategic: 什麼時候 PR 級問題應該觸發架構層級的討論？

### Q5: Human governance boundary
「人應該寫所有測試」vs「讓 agent 也寫測試」— 用 oracle 的角度分析這兩種策略的 trade-off。什麼情況下讓 agent 寫測試是安全的？什麼情況下不是？

### Q6: Effector budget
如果你要為一個 coding agent 設定 diff budget（單次改動行數上限），你會設多少？為什麼？
提示：考慮以下因素 — review 成本、回滾風險、測試覆蓋率的信心。

---

## Classroom Activity: Build Your Own Control Stack

### 目標
3-4 人一組，為一個真實的 coding task 設計完整的 control stack。

### 步驟
1. 選一個真實場景（例：修復一個 flaky test、實作一個新 API endpoint、遷移一個依賴）
2. 畫出你的 5 層 stack，每層至少寫出具體工具/檢查
3. 預測：這個 stack 最可能遇到哪種病理？你的防護機制是什麼？
4. 各組互相 review：找出對方 stack 的弱點

### 評分維度
- 各層是否具體可操作（不是空泛的「跑測試」）
- 病理預測是否合理
- 人的角色是否明確定義
