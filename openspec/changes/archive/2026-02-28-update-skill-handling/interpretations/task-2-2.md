# Spec Interpretation: Task 2-2 /spec コマンドにドメイン判定ロジックを追加

## 対象要件
- Requirement: REQ-007 `/spec` コマンドのドメイン判定ロジック
  - **仕様の記述**:
    - GIVEN proposal.md に「データベース」「テーブル」が含まれる WHEN ドメイン判定を実行する THEN `prisma-expert/design` と `database-migrations/design` が REQUIRED SKILLS に注入される
    - GIVEN proposal.md に「API」「エンドポイント」が含まれる WHEN ドメイン判定を実行する THEN `nextjs-api-patterns/design` と `security-patterns/design` が注入される
    - GIVEN proposal.md に複数ドメインのキーワードが含まれる WHEN ドメイン判定を実行する THEN 全ての該当ドメイン Skill を Union で含める
    - GIVEN ドメイン判定を実行する WHEN `architecture-patterns/design` の包含を検証する THEN 常に含まれている
    - GIVEN ドメイン判定で6個以上の Skill が該当する WHEN 注入する Skill を決定する THEN 最大5個に制限する
    - GIVEN proposal.md にキーワードが一つも該当しない WHEN ドメイン判定を実行する THEN `architecture-patterns/design` のみを注入する
  - **私の解釈**: Phase 1b と Phase 2 の間に Phase 1.7 を挿入する。proposal.md のキーワードからドメインを推論するテーブルを定義し、該当するドメイン Skill を `/design` サフィックス付きで注入する。`architecture-patterns/design` は常に含める。spec-writer と spec-validator の REQUIRED SKILLS に注入する仕組みを記述する
  - **前提（仕様に未記載だが私が仮定すること）**:
    - Phase 1.7 は Teams モードと Sub Agents モードの両方に適用される（Phase 1b の直後に位置するため、どちらのモードでも Phase 2 の前に実行される）
    - キーワード推論テーブルは design.md セクション6 のテーブルと一致させる（仕様の SSOT）
    - 最大5個制限時の優先順位: `architecture-patterns/design` は必ず含め、残り4枠をキーワード該当順（上から）で埋める
  - **仕様でカバーされないエッジケース**:
    - 日本語と英語のキーワード混在: design.md のテーブルは日本語キーワードを使用。英語キーワードも補完的に検出すべきだが、仕様で明示されていないため日本語キーワードのみとする

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| Phase 1.7 の適用範囲 | A: 両モード共通, B: Sub Agents のみ | A | Phase 1.7 はドメイン判定であり、spec-writer/spec-validator のプロンプトに Skill を注入するためにどちらのモードでも必要 | B: Teams モードでも spec-writer にドメイン Skill を渡す必要がある |
| 5個制限時の優先順位 | A: architecture-patterns 必須 + 残り4枠をテーブル順, B: 全て均等に扱う | A | delta-spec が「architecture-patterns/design は常に含まれている」と明記。必須枠として確保すべき | B: 仕様に反する |
| キーワード推論テーブルのソース | A: design.md のテーブルを転記, B: 独自定義 | A | design.md セクション6 に詳細なキーワード→ドメイン→Skill のマッピングが定義済み。SSOT 原則に従い同一テーブルを使用 | B: 二重管理になり陳腐化リスク |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → proposal.md のテキスト。キーワード未該当時は architecture-patterns/design のみ
- [x] エラー時の振る舞いは？ → proposal.md が存在しない場合は Step 0 でエラー（既存フロー）。ドメイン判定自体はエラーにならない
- [x] 外部依存が失敗した場合は？ → 該当なし（ファイル読み込みのみ）
- [x] 非同期処理の場合、競合状態への対処は？ → 該当なし

## ギャップ検出（エスカレーション候補）
- なし（仕様が十分に明確）

---

## Phase B: 実装完了後の追記

### 変更ファイル一覧

#### 作成したファイル
- なし

#### 修正したファイル
- `~/.claude/commands/spec.md`: Phase 1.7（ドメイン判定）を Phase 1b と Phase 1.5 の間に挿入。キーワード推論テーブル（5ドメイン）、判定ルール（Union、architecture-patterns 常時含む、最大5個制限）、Skill 注入方法を定義。Phase 3 の spec-validator 検証項目の記述を EARS + Google Design Review 品質基準 / 9つの検証項目に更新

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: Phase 1.7 の適用範囲が Teams/Sub Agents 両方か仕様に明示されていなかったが、どちらのモードでも spec-writer/spec-validator にドメイン Skill を渡す必要があるため両方に適用した
- 却下した代替案: Phase 1.7 を Sub Agents モードのみに適用する案を却下。Teams モードでも spec-writer のプロンプトにドメイン Skill を注入する必要がある
- 想定外の発見: spec.md の Phase 3 セクションに spec-validator の品質基準・検証項目の参照があり、Task #7, #10 の変更に合わせて更新が必要だった（EARS 4品質基準 → EARS + Google Design Review、7検証項目 → 9検証項目）
