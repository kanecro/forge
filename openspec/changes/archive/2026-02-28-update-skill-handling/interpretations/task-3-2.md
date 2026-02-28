# Spec Interpretation: Task 3-2 /compound に派生ファイル同期ロジックを追加

## 対象要件
- Requirement: REQ-012 `/compound` の派生ファイル同期
  - **仕様の記述**:
    - GIVEN Learning Router でドメイン Skill の SKILL.md が更新される WHEN 同 Skill ディレクトリに design.md が存在する THEN 更新内容が設計知識に影響するか判定し、影響があれば design.md を同期更新する
    - GIVEN SKILL.md が更新される WHEN 同 Skill ディレクトリに constraints.md が存在する THEN 更新内容が制約に影響するか判定し、影響があれば constraints.md を同期更新する
    - GIVEN SKILL.md が更新される WHEN 同 Skill ディレクトリに design.md / constraints.md が存在しない THEN スキップする
    - GIVEN 同期更新後の design.md が120行を超過する WHEN 検証する THEN 超過部分を SKILL.md に残し、design.md は120行以内に収める
  - **私の解釈**:
    - ワークフローのステップ4（Learning Router）の後にステップ4.5を挿入する
    - Learning Router で SKILL.md が更新された場合にのみ実行される同期ステップ
    - skill-phase-formatter の同期手順を参照する旨を明記
    - 派生ファイルが存在しない場合は新規生成せずスキップ
  - **前提（仕様に未記載だが私が仮定すること）**:
    - ステップ4で SKILL.md を更新しなかった場合（更新提案がなかった or ユーザーが却下した場合）、ステップ4.5はスキップ
    - 同期更新は自動ではなく、差分をユーザーに提示して承認を得てから適用する（Learning Router のユーザー承認フローに準拠）
  - **仕様でカバーされないエッジケース**:
    - constraints.md の行数上限（30行）超過時の対応: design.md と同様に超過部分を SKILL.md に残す

## 実装判断

| 判断項目 | 選択肢 | 採用 | 根拠 | 却下理由 |
|---|---|---|---|---|
| ステップの挿入位置 | 代替案なし -- 仕様が一意に指定（ステップ4の後、4.5） | --- | --- | 代替案なし -- 仕様が一意に指定 |
| 同期判定方式 | A: 全派生ファイルを無条件更新, B: 影響判定してから更新 | B | 仕様に「影響するか判定し」と明記 | A: 不要な更新が発生する |
| skill-phase-formatter 連携 | A: 手順を直接記載, B: skill-phase-formatter を参照 | B | タスク説明に「skill-phase-formatter との連携明記」と指定 | A: SSOT 原則に反する |

## 必須チェック項目
- [x] 入力の有効値/無効値は何か？ → SKILL.md が更新されたか否か。更新されなければスキップ
- [x] エラー時の振る舞いは？ → design.md が120行を超過した場合は超過部分を SKILL.md に残す
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
- `~/.claude/commands/compound.md`: ワークフローにステップ4.5（Skill派生ファイルの同期）を追加。SKILL.md更新時の派生ファイル存在チェック、影響判定、同期フロー、行数上限検証、skill-phase-formatter連携を定義

#### 削除したファイル
- なし

### 実装の振り返り
- 仕様の曖昧性への対処: constraints.md の行数上限超過時の対応は仕様に未記載だったが、design.md と同様のアプローチ（超過部分をSKILL.mdに残す）を採用
- 却下した代替案: 自動同期（ユーザー承認なし）→既存のLearning Routerのユーザー承認フローに準拠して承認を要求
- 想定外の発見: 特になし
