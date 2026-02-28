# Spec Interpretation: Task 2-3 spec-validator に STRIDE + Google 4観点を追加

## 対象要件
- Requirement: REQ-008 spec-validator の STRIDE + Google 4観点
  - **仕様の記述**:
    - GIVEN 保護リソースへのアクセスを含む仕様を検証する WHEN STRIDE 簡易チェックを実行する THEN Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege の6観点で評価する
    - GIVEN 仕様の品質を検証する WHEN Google 4観点で評価する THEN Correctness, Completeness, Consistency, Clarity で評価する
    - GIVEN セキュリティに無関係な仕様 WHEN STRIDE チェックを実行する THEN スキップ可能
    - GIVEN 曖昧表現がある WHEN Clarity チェックを実行する THEN フラグして具体的な基準の記述を推奨する
  - **私の解釈**:
    1. frontmatter の skills に `architecture-patterns` を追加する
    2. 既存の「7 つの検証項目」セクションに「8. STRIDE 簡易チェック」を追加する
    3. 既存の「EARS ベースの 4 品質基準」セクションを「EARS + Google Design Review 品質基準」に拡張し、Google 4観点を統合する
  - **前提（仕様に未記載だが私が仮定すること）**:
    - STRIDE チェックはセキュリティ無関係な仕様（UI スタイル変更等）ではスキップ可能。判断基準は「保護リソース（認証、個人情報、権限制御等）へのアクセスを含むか」
    - Google 4観点は既存の EARS 4基準を置き換えるのではなく、統合・拡張する。EARS の「テスト可能性」「振る舞い中心」「一意解釈性」「十分な完全性」は維持しつつ、Google の Correctness/Completeness/Consistency/Clarity を追加の視点として組み込む
  - **仕様でカバーされないエッジケース**:
    - STRIDE の各観点で「問題なし」の場合の出力形式: 「N/A」または「該当なし」と記載して検証済みであることを明示する
    - STRIDE と Google 4観点の重複（例: Completeness と EARS の「十分な完全性」は類似）: 統合して1つの観点にまとめ、重複を避ける

- Requirement: REQ-M02 spec-validator の品質基準拡張
  - **仕様の記述**:
    - GIVEN 品質基準セクションを検証する WHEN 内容を確認する THEN EARS 4基準と Google 4観点の両方が含まれている
  - **私の解釈**: 品質基準テーブルを拡張し、EARS 基準と Google 観点を統合した形にする
  - **前提**: EARS 基準を維持しつつ Google 観点を追加する（置換ではない）
  - **仕様でカバーされないエッジケース**: なし

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| Google 4観点の統合方式 | A: EARS テーブルに行追加, B: 別テーブル, C: EARS テーブルを再構成して統合 | C | EARS の完全性と Google の Completeness、EARS の一意解釈性と Google の Clarity は重複する。重複を排除しつつ両方の観点を含む統合テーブルが最適 | A: 重複が残る; B: 検証者が2つのテーブルを別々に確認する必要があり非効率 |
| STRIDE の配置 | A: 検証項目8 として追加, B: 品質基準セクションに統合 | A | delta-spec が「検証項目8」と明記。品質基準とは性質が異なる（品質基準は全要件に適用、STRIDE はセキュリティ関連のみ） | B: 仕様が検証項目としての追加を指定 |
| セキュリティ無関係の判断基準 | A: 保護リソースへのアクセス有無, B: ドメイン判定ベース | A | delta-spec の Boundary Scenarios が「セキュリティに無関係な仕様（UI スタイル変更等）」と例示。保護リソース基準が最も直感的 | B: ドメインだけでは判断不十分（フロントエンドでも認証 UI はセキュリティ関連） |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → 検証対象は delta-spec の要件。有効: ADDED/MODIFIED 要件。無効値は該当なし
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
- `~/.claude/agents/spec/spec-validator.md`: frontmatter の skills に `architecture-patterns` を追加。品質基準セクションを「EARS + Google Design Review 品質基準」に拡張（EARS 4基準 + Google 4観点を統合した6項目テーブル）。検証項目8「STRIDE 簡易チェック」を追加。ワークフローの Step 2, Step 3 の説明を更新

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: EARS 4基準と Google 4観点の統合方式について、重複を排除して6項目に統合した。EARS の「一意解釈性」と Google の「Clarity」、EARS の「十分な完全性」と Google の「Completeness」は実質的に同じ観点なので統合した
- 却下した代替案: EARS テーブルに Google 観点を行追加する案（重複が残る）、別テーブルにする案（検証者が2つを参照する非効率）を却下
- 想定外の発見: 品質基準が EARS 4 + Google 4 = 8 ではなく、重複を統合して6項目になった。これは仕様が「両方が含まれている」ことを要求しており、統合形式は禁止していないため適切と判断
