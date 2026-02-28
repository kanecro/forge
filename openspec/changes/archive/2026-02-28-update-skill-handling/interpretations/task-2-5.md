# Spec Interpretation: Task 2-5 Review Agent 6個にドメイン Skill 宣言を追加

## 対象要件
- Requirement: REQ-010
  - **仕様の記述**: 6個の Review Agent の frontmatter skills にドメイン Skill を追加し、SSOT ルールを追加しなければならない。各 Agent の frontmatter skills に指定されたドメイン Skill が設定されていること。サフィックスなし（SKILL.md 全体を参照する）。SSOT ルールが本文末尾に追加されていること。
  - **私の解釈**: 各 Agent ファイルの frontmatter `skills` 配列に既存の `iterative-retrieval` に加えてドメイン Skill を追加する。各 Agent の本文末尾に SSOT ルール（ドメイン Skill 優先）を追加する。type-safety-reviewer にはドメイン Skill を追加しない（横断的関心事のため）。
  - **前提（仕様に未記載だが私が仮定すること）**: 既存の `iterative-retrieval` は維持したまま、ドメイン Skill を追加する。SSOT ルールのテキストは各 Agent で同一。
  - **仕様でカバーされないエッジケース**: review-aggregator への影響。対応: review-aggregator は指摘の統合が役割であり、ドメイン Skill の宣言は不要。

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| SSOT ルールの配置 | A: 本文末尾に独立セクション, B: Required Skills セクション内 | A | 仕様に「本文末尾に追加」と明記 | B: 仕様に反する |
| スキル名のサフィックス | A: なし（例: prisma-expert）, B: あり（例: prisma-expert/design） | A | 仕様の Boundary Scenario に「サフィックスなし（SKILL.md 全体を参照する）」と明記 | B: 仕様に反する。Review Agent は実装フェーズ相当で SKILL.md 全体を参照 |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → 6個の Agent ファイルが対象。各ファイルに追加するスキルは仕様で一意に指定
- [x] エラー時の振る舞いは？ → 該当なし（マークダウンファイルの編集のみ）
- [x] 外部依存が失敗した場合は？ → 該当なし
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- なし

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- なし

#### 修正したファイル
- `~/.claude/agents/review/security-sentinel.md`: frontmatter skills に `security-patterns` 追加、本文末尾に SSOT ルール追加
- `~/.claude/agents/review/performance-oracle.md`: frontmatter skills に `next-best-practices`, `prisma-expert` 追加、本文末尾に SSOT ルール追加
- `~/.claude/agents/review/architecture-strategist.md`: frontmatter skills に `architecture-patterns`, `next-best-practices` 追加、本文末尾に SSOT ルール追加
- `~/.claude/agents/review/prisma-guardian.md`: frontmatter skills に `prisma-expert`, `database-migrations` 追加、本文末尾に SSOT ルール追加
- `~/.claude/agents/review/terraform-reviewer.md`: frontmatter skills に `terraform-gcp-expert` 追加、本文末尾に SSOT ルール追加
- `~/.claude/agents/review/api-contract-reviewer.md`: frontmatter skills に `nextjs-api-patterns`, `security-patterns` 追加、本文末尾に SSOT ルール追加

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: 仕様は各 Agent に追加するスキルを一意に指定しており、曖昧性はなかった。
- 却下した代替案: SSOT ルールを Required Skills セクション内に記載する案を却下。仕様に「本文末尾」と指定されているため、独立セクションとして追加した。
- 想定外の発見: type-safety-reviewer と review-aggregator の2ファイルはドメイン Skill 追加対象外であることを確認。仕様通りに6ファイルのみ変更。
