# Spec Interpretation: Task 4-1 core-essentials.md + CLAUDE.md の更新

## 対象要件
- Requirement: REQ-017 core-essentials.md + CLAUDE.md の更新
  - **仕様の記述**:
    - GIVEN 全 Phase が完了する WHEN core-essentials.md を検証する THEN 「Skill Orchestration」セクションが Phase-Aware 版に更新されている: `/brainstorm` → `<skill>/constraints`、`/spec` → `<skill>/design`、`/implement`, `/review` → `<skill>`（現行通り）
    - GIVEN 全 Phase が完了する WHEN CLAUDE.md を検証する THEN Available Skills に `skill-phase-formatter` が追加されている
    - GIVEN グローバル CLAUDE.md を更新する WHEN プロジェクト CLAUDE.md を検証する THEN 両方のファイルが同期されている
    - GIVEN グローバル CLAUDE.md を更新する WHEN プロジェクト CLAUDE.md の同期を忘れる THEN 同期漏れが発生する（過去の学び: 同期漏れ事例あり）。最終検証で差分を確認する
  - **私の解釈**:
    - core-essentials.md の「Skill Orchestration（1% ルール）」セクションにフェーズ別サフィックスマッピングを追加する
    - CLAUDE.md の Available Skills テーブルの「方法論」行に `skill-phase-formatter` を追加する
    - グローバル（~/.claude/CLAUDE.md）とプロジェクト（/Users/kosuke/forge/CLAUDE.md）の両方を同期する
  - **前提（仕様に未記載だが私が仮定すること）**:
    - core-essentials.md は既存の説明文を拡張する形で Phase-Aware 情報を追加する（全面書き換えではない）
    - CLAUDE.md の他のセクション（Agents, Hooks 等）は変更しない
  - **仕様でカバーされないエッジケース**:
    - プロジェクト CLAUDE.md にグローバル CLAUDE.md と異なるセクション（Forge ワークフロー図等）が存在する場合: プロジェクト固有セクションはそのまま保持し、共通セクション（Available Skills）のみ同期する

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| core-essentials.md の更新範囲 | A: Skill Orchestration セクションのみ, B: 全セクション見直し | A | 仕様に「Skill Orchestration セクションが Phase-Aware 版に更新」と明記 | B: スコープ外 |
| CLAUDE.md の更新範囲 | A: Available Skills のみ, B: Available Skills + Available Commands | A | tasks.md に「Available Commands に /skill-format を追加（暗黙的記載）」とあるが、現行 CLAUDE.md に Available Commands セクションが存在しないため、コマンドは既存 Agents/Skills テーブルの枠組みで対応 | B: 存在しないセクションの新規追加はMinimal Change原則に反する |
| CLAUDE.md 同期方式 | A: グローバル先行→プロジェクトコピー, B: 個別更新 | A | 同期漏れ防止のため、グローバルを先に更新しプロジェクトに反映 | B: 同期漏れリスクが高い |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → 全 Phase 完了が前提。Task #6, #10 の完了を待つ
- [x] エラー時の振る舞いは？ → 該当なし（マークダウン定義のため）
- [x] 外部依存が失敗した場合は？ → 該当なし
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- tasks.md に「Available Commands に /skill-format を追加（暗黙的記載）」とあるが、現行 CLAUDE.md に Available Commands セクションが存在しない。新規セクション追加ではなく、既存テーブルの方法論行に skill-phase-formatter Skill を追加することで /skill-format コマンドの存在も暗黙的に示す方針とする

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- なし

#### 修正したファイル
- `~/.claude/rules/core-essentials.md`: 「Skill Orchestration」セクションに Phase-Aware ファイルサフィックスのサブセクションを追加。フェーズ別サフィックスマッピングテーブル（/brainstorm→/constraints, /spec→/design, /implement,/review→なし）とフォールバック動作を定義
- `~/.claude/CLAUDE.md`: Available Skills テーブルの「方法論」行に `skill-phase-formatter` を追加
- `/Users/kosuke/forge/rules/core-essentials.md`: グローバル版と同一の Phase-Aware ファイルサフィックスセクションを追加（同期）

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: プロジェクト CLAUDE.md はグローバル CLAUDE.md と構造が異なる（詳細ワークフロー図あり、スキルはディレクトリ参照方式）ため、構造の同一化ではなく内容の整合性を確保する方針とした
- 却下した代替案: プロジェクト CLAUDE.md にもグローバルと同じ詳細 Skills テーブルを追加する案 → プロジェクト CLAUDE.md は意図的にコンパクト化されており、スキルの自動検出参照方式を維持した方が一貫性がある
- 想定外の発見: core-essentials.md がグローバル（~/.claude/rules/）とプロジェクト（/Users/kosuke/forge/rules/）の両方に存在し、両方とも同期が必要だった
