# Workflow Rules

Lattice ワークフローのルール。セッション管理、タスク管理、コラボレーション。

---

## Session Lifecycle

### Session Start

```
1. 自動実行 (session-start.js):
   - セッション統計を報告
   - 関連コンテキストをロード
   - 保留中のエスカレーションを表示

2. 手動確認:
   /action-status          # フェーズ確認
   /checkpoint-list       # 前回のチェックポイント確認
```

### During Session

```
1. フェーズに沿った作業
   - spec: 仕様書作成
   - design: 設計文書作成
   - implement: コード実装
   - integrate: 統合テスト

2. 定期チェックポイント (30分ごと推奨)
   /checkpoint [name]

3. パターン記録
   - 成功パターン → log_pattern
   - 判断 → log_decision
   - 不明点 → log_escalation
```

### Session End

```
1. 自動実行 (session-end.js):
   - チェックポイント保存
   - パターンを蒸留キューにプッシュ
   - MEMORY.md に同期

2. 推奨:
   /checkpoint session-end --description "今日の作業サマリー"
```

---

## Task Management

### Task Structure

```
Plan (計画)
└── Phase (フェーズ)
    └── Task (タスク)
        └── Todo (TODO)
```

### Task Workflow

```
1. タスク作成
   - 明確な目標を定義
   - 完了条件を明記
   - 依存関係を特定

2. タスク実行
   - フェーズに適した作業のみ
   - 小さな単位で進行
   - 定期的に進捗を記録

3. タスク完了
   - 完了条件を満たしているか確認
   - 検証を通過
   - チェックポイント作成
```

### Task Status

| Status | Description |
|--------|-------------|
| pending | 未着手 |
| in_progress | 作業中 |
| blocked | ブロック中 |
| review | レビュー待ち |
| completed | 完了 |

---

## Phase Transitions

### Transition Requirements

| From → To | Requirements |
|-----------|--------------|
| init → spec | プロジェクト構造完成、policy.md 作成 |
| spec → design | 仕様書完成、ユーザーストーリー定義 |
| design → implement | 設計書完成、インターフェース定義 |
| implement → integrate | 機能実装完了、ユニットテスト通過 |
| integrate → complete | 統合テスト通過、ドキュメント完成 |

### Transition Workflow

```
1. 現在のフェーズの Exit Criteria を確認
   /action-status

2. 検証を実行
   /verify full

3. すべて通過したらフェーズ遷移
   /workflow-guide

4. 自動でチェックポイントが作成される
```

---

## Verification Workflow

### Before Code Changes

```bash
# 変更前の状態を確認
git status
git diff

# 現在のフェーズを確認
/action-status
```

### After Code Changes

```bash
# クイック検証
/verify quick

# 標準検証 (推奨)
/verify standard

# 問題があれば修正
# 再度検証
```

### Before Commit

```bash
# 完全検証
/verify full

# すべて通過したらコミット
git add <files>
git commit -m "type(scope): description"
```

### Before Phase Advance

```bash
# 完全検証 (必須)
/verify full

# Exit Criteria 確認
/action-status

# フェーズ遷移
/workflow-guide
```

---

## Pattern Learning Workflow

### Pattern Discovery

```
1. パターンを発見
   - 成功した実装アプローチ
   - 効果的な問題解決方法
   - 再利用可能な設計パターン

2. パターンを記録
   log_pattern({
     type: 'implementation',
     summary: 'パターンの要約',
     context: '適用コンテキスト',
     confidence: 0.7
   })

3. パターンを適用
   - 類似の状況で再利用
   - 結果を観察
   - 信頼度を更新
```

### Pattern Evolution

```
観察 → 記録 → 検証 → 蒸留 → ポリシー
  ↑                        ↓
  ←←←← フィードバック ←←←←
```

### Distillation Workflow

```bash
# パターン状態を確認
/pattern-status

# 蒸留を実行
/distill

# 結果を確認
/pattern-status
```

---

## Escalation Workflow

### Creating Escalation

```
1. 不明点や判断が必要な場面を特定

2. エスカレーションを記録
   log_escalation({
     question: '何を確認したいか',
     context: '背景情報',
     options: ['選択肢1', '選択肢2'],
     recommendation: '推奨案',
     blocking: true/false
   })

3. ブロッキングの場合は回答を待つ
4. 非ブロッキングの場合は他の作業を継続
```

### Resolving Escalation

```
1. 回答を受け取る

2. 決定を記録
   log_decision({
     question: 'オリジナルの質問',
     decision: '決定内容',
     rationale: '理由',
     confidence: 0.9
   })

3. 作業を継続
```

---

## Checkpoint Workflow

### Manual Checkpoint

```bash
# 名前付きチェックポイント
/checkpoint pre-refactor

# 説明付き
/checkpoint milestone-1 --description "認証機能完成"

# タグ付き
/checkpoint release-candidate --tags release,tested
```

### Automatic Checkpoints

| Trigger | Checkpoint Name |
|---------|-----------------|
| フェーズ遷移 | auto-phase-{phase} |
| セッション終了 | auto-session-end |
| コンテキスト圧縮前 | auto-pre-compact |
| エスカレーション解決 | auto-escalation-{id} |

### Checkpoint Restoration

```bash
# チェックポイント一覧
/checkpoint-list

# 特定のチェックポイントを復元
/checkpoint-restore chk_20240118_143000

# 名前で復元
/checkpoint-restore pre-refactor
```

---

## Knowledge Synchronization

### Local → Global

```
1. プロジェクト固有のパターンを蓄積

2. 高信頼度パターンを蒸留
   /distill

3. グローバルナレッジに昇格 (自動)
   - 信頼度 >= 0.8
   - 証拠数 >= 5
   - 複数プロジェクトで有効
```

### Global → Local

```
1. セッション開始時に自動ロード
   - ファイルパターンに基づく
   - キーワードに基づく
   - フェーズに基づく

2. 必要に応じて手動検索
   search_knowledge({ query: '認証パターン' })
```

---

## Collaboration Workflow

### Handoff Pattern

```
1. チェックポイント作成
   /checkpoint handoff-to-team --description "現状のサマリー"

2. ドキュメント更新
   - README.md に進捗を記録
   - 保留事項を明記
   - 次のステップを記載

3. Git コミット
   git commit -m "chore: checkpoint for handoff"
```

### Resume Pattern

```
1. 最新のチェックポイントを確認
   /checkpoint-list

2. チェックポイントを復元
   /checkpoint-restore <checkpoint-id>

3. コンテキストを確認
   /action-status
   /pattern-status
```

---

## Error Recovery

### Build Failure

```
1. エラーメッセージを確認

2. 最後の成功チェックポイントを特定
   /checkpoint-list

3. 必要に応じて復元
   /checkpoint-restore <checkpoint-id>

4. 問題を修正

5. 検証
   /verify standard
```

### Phase Violation

```
1. 違反内容を確認

2. オプションを評価:
   a. 現在のフェーズを完了 (推奨)
   b. 作業をフェーズに適した形に調整
   c. 強制進行 (非推奨)

3. 選択した対応を実行

4. フェーズ状態を確認
   /action-status
```

---

## Best Practices

### Do

- ✅ フェーズに沿った作業を行う
- ✅ 定期的にチェックポイントを作成
- ✅ パターンを記録・蓄積する
- ✅ 不明点はエスカレーションする
- ✅ 検証を通過してからコミット

### Don't

- ❌ フェーズをスキップする
- ❌ 検証なしでコミットする
- ❌ エスカレーションを無視する
- ❌ パターンを記録せずに進行する
- ❌ チェックポイントなしで大きな変更をする

---

_Workflow Rules: セッション管理 × フェーズ規律 × 継続学習_
