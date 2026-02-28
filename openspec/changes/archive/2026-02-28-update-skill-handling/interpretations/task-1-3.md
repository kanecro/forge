# Spec Interpretation: Task 1-3 主要ドメイン Skill 9個のファイル分割

## 対象要件
- Requirement: REQ-004 主要ドメイン Skill 9個のファイル分割
  - **仕様の記述**: 9個のドメイン Skill を Phase-Aware File Structure に分割。各 design.md ≤ 120行、constraints.md ≤ 30行、design.md にコードブロックなし
  - **私の解釈**: 各 SKILL.md を読み込み、skill-phase-formatter の分割基準に従って技術的制約 → constraints.md、設計指針 → design.md に抽出する。全内容は SKILL.md の部分集合であることが必須
  - **前提**: 新しい知識は追加しない。SKILL.md に記述されている内容のみを抽出・要約する
  - **仕様でカバーされないエッジケース**: SKILL.md に明確な「制約」セクションがない Skill もある。その場合、アンチパターンやルールテーブルから制約を抽出する

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| ヘッダー形式 | A: `# <skill-name>: 設計ガイダンス` / `# <skill-name>: 技術的制約` | A | REQ-004 Non-Functional Requirements で CONSISTENCY として明記 | 代替案なし -- 仕様が一意に指定 |
| 抽出方法 | A: SKILL.md の記述を要約・再構成して抽出 | A | design.md は行数制限があるため、SKILL.md の全文転記は不可。要約だが内容は部分集合 | 代替案なし |
| コードブロック除外 | A: design.md からコードブロックを完全除外 | A | REQ-004 が明示的に禁止 | 代替案なし -- 仕様が一意に指定 |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → 9つの SKILL.md が入力。全て存在確認済み
- [x] エラー時の振る舞いは？ → 該当なし（ファイル作成作業）
- [x] 外部依存が失敗した場合は？ → 該当なし
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- なし

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- `~/.claude/skills/prisma-expert/constraints.md`: 技術的制約（17行）
- `~/.claude/skills/prisma-expert/design.md`: 設計ガイダンス（78行）
- `~/.claude/skills/database-migrations/constraints.md`: 技術的制約（15行）
- `~/.claude/skills/database-migrations/design.md`: 設計ガイダンス（92行）
- `~/.claude/skills/next-best-practices/constraints.md`: 技術的制約（14行）
- `~/.claude/skills/next-best-practices/design.md`: 設計ガイダンス（92行）
- `~/.claude/skills/nextjs-api-patterns/constraints.md`: 技術的制約（14行）
- `~/.claude/skills/nextjs-api-patterns/design.md`: 設計ガイダンス（101行）
- `~/.claude/skills/security-patterns/constraints.md`: 技術的制約（16行）
- `~/.claude/skills/security-patterns/design.md`: 設計ガイダンス（111行）
- `~/.claude/skills/architecture-patterns/constraints.md`: 技術的制約（13行）
- `~/.claude/skills/architecture-patterns/design.md`: 設計ガイダンス（120行）
- `~/.claude/skills/terraform-gcp-expert/constraints.md`: 技術的制約（16行）
- `~/.claude/skills/terraform-gcp-expert/design.md`: 設計ガイダンス（108行）
- `~/.claude/skills/vercel-react-best-practices/constraints.md`: 技術的制約（14行）
- `~/.claude/skills/vercel-react-best-practices/design.md`: 設計ガイダンス（112行）
- `~/.claude/skills/vercel-composition-patterns/constraints.md`: 技術的制約（13行）
- `~/.claude/skills/vercel-composition-patterns/design.md`: 設計ガイダンス（82行）

#### 修正したファイル
- なし

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: SKILL.md に明確な「制約」セクションがない Skill では、アンチパターンテーブル、ルールテーブル、「禁止」「必須」等のキーワードから制約を抽出した
- 却下した代替案: SKILL.md の文章をそのまま転記する方法は行数制限に収まらないため却下。要約・再構成して内容の本質を保ちつつ行数を削減した
- 想定外の発見: prisma-expert の design.md が78行と80行の目安を若干下回ったが、境界シナリオ「80行未満で生成する。空にはしない」に該当し問題なし
