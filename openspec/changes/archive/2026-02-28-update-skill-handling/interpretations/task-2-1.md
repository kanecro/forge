# Spec Interpretation: Task 2-1 spec-writer にドメイン Skill 注入

## 対象要件
- Requirement: REQ-006 spec-writer へのドメイン Skill 注入
  - **仕様の記述**:
    - GIVEN spec-writer が起動する WHEN frontmatter を検証する THEN `skills: [iterative-retrieval, verification-before-completion, architecture-patterns]` が設定されている
    - GIVEN spec-writer が design.md を生成する WHEN ドメイン Skill 参照ガイダンスを検証する THEN 設計パターン選択、アンチパターン回避、トレードオフ説明にドメイン Skill を活用する指示が含まれている
    - GIVEN ドメイン Skill の指針とビジネス要件が矛盾する WHEN spec-writer が判断する THEN ビジネス要件を最優先する（ドメイン Skill はガイドライン）
  - **私の解釈**: frontmatter の skills 配列に `architecture-patterns` を追加する。また、spec-writer の本文にドメイン Skill 参照ガイダンスセクションを追加し、design.md 生成時にドメイン Skill の設計知識を活用するよう指示する。ビジネス要件とドメイン Skill が矛盾した場合のルールも明記する
  - **前提（仕様に未記載だが私が仮定すること）**:
    - ドメイン Skill 参照ガイダンスの配置場所は、既存の Step 4（design.md 生成ステップ）内に組み込むのが適切。新しいステップを追加するのではなく、既存ワークフローに自然に統合する
    - `architecture-patterns` は常時読み込みだが、他のドメイン Skill はプロンプトで `REQUIRED SKILLS` として渡される想定（Phase 1.7 のドメイン判定で決定）
  - **仕様でカバーされないエッジケース**:
    - ドメイン Skill が存在しない場合の振る舞い: Claude Code の自動解決に委ねる（spec-writer 側で存在チェックは不要）

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| ガイダンスの配置場所 | A: Step 4 内に統合, B: 独立セクション | A | design.md 生成フローの中で参照するのが自然。既存ワークフローの流れを壊さない | B: 独立セクションにすると参照タイミングが曖昧になる |
| ドメイン Skill の優先度ルール | A: ビジネス要件優先, B: ドメイン Skill 優先 | A | delta-spec REQ-006 が「ビジネス要件を最優先する」と明記 | B: 仕様に反する |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → frontmatter skills は配列形式。architecture-patterns は有効なスキル名
- [x] エラー時の振る舞いは？ → 該当なし（マークダウンファイルの編集のみ）
- [x] 外部依存が失敗した場合は？ → 該当なし
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- なし（仕様が十分に明確）

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- なし

#### 修正したファイル
- `~/.claude/agents/spec/spec-writer.md`: frontmatter の skills に `architecture-patterns` を追加。Required Skills セクションに説明を追加。Step 4 にドメイン Skill 参照ガイダンス（設計パターン選択、アンチパターン回避、トレードオフ説明）と優先順位ルール（ビジネス要件優先）を追加

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: ドメイン Skill 参照ガイダンスの配置場所が仕様で明示されていなかったが、Step 4（design.md 生成ステップ）内に統合するのが最も自然と判断した
- 却下した代替案: 独立セクションとして配置する案を却下。参照タイミングが曖昧になり、ワークフローの流れを壊すため
- 想定外の発見: なし
