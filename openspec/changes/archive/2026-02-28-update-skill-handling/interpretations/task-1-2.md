# Spec Interpretation: Task 1-2 /skill-format コマンドの作成

## 対象要件
- Requirement: REQ-003 /skill-format コマンドの作成
  - **仕様の記述**: 4モード（単一分割、一括 --all、状況確認 --check、同期 --sync）のコマンドを作成。skill-phase-formatter との連携を明記
  - **私の解釈**: 既存コマンド（brainstorm.md, review.md 等）と同じ構造で作成。frontmatter に description と argument-hint を含める。各モードのワークフローを明記
  - **前提**: コマンドはマークダウン定義であり、Claude が対話的に実行する。プログラムコードではない
  - **仕様でカバーされないエッジケース**: --all 実行時にどの Skill をドメインと判定するかの具体的なリストは design.md の「分割対象の判定基準」セクションに記載がある

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| コマンドファイルの配置 | A: ~/.claude/commands/skill-format.md | A | 仕様が明示指定 | 代替案なし -- 仕様が一意に指定 |
| argument-hint の形式 | A: `<skill-name\|--all\|--check\|--sync skill-name>` | A | 仕様が明示指定 | 代替案なし -- 仕様が一意に指定 |
| エラーハンドリング | A: SKILL.md 不在エラー + 上書き確認 | A | 仕様 REQ-003 Error Scenarios が明記 | 代替案なし -- 仕様が一意に指定 |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → 有効: skill-name, --all, --check, --sync skill-name。無効: 存在しない skill-name（エラー出力）
- [x] エラー時の振る舞いは？ → SKILL.md が存在しない場合エラーメッセージ。既存 design.md/constraints.md がある場合は上書き確認
- [x] 外部依存が失敗した場合は？ → 該当なし
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- なし

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- `~/.claude/commands/skill-format.md`: /skill-format コマンド定義ファイル（103行）

#### 修正したファイル
- なし

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: --all モードのスキップ対象リストを design.md の分割対象判定基準に基づいて具体化した
- 却下した代替案: なし
- 想定外の発見: なし
