---
description: "実装済みコードをSpec-Awareレビュー（仕様コンテキスト注入 + 動的レビュアー選択 + カバレッジマトリクス）で多角的にレビューする"
disable-model-invocation: true
argument-hint: "<change-name>"
---

# /review コマンド

## 目的

実装済みコードを仕様準拠の観点を含めて多角的にレビューする。
delta-spec と design.md をレビュアーに必須入力として提供し、設計意図を理解した上でのレビューを実現する。

## 引数の解析

$ARGUMENTS から change-name を決定する:

- 指定あり: `openspec/changes/<change-name>/` を対象とする
- 省略: `openspec/changes/` 内のアクティブ変更（`archive/` 以外）を自動検出
  - 1つ → 自動選択
  - 複数 → AskUserQuestion で選択
  - 0 → エラー（先に `/brainstorm` を実行するよう案内）

## ワークフロー

### Step 1: 仕様コンテキストの準備

1. `openspec/changes/<change-name>/specs/` 配下のデルタスペックファイル一覧を取得
2. `openspec/changes/<change-name>/design.md` のパスを確認
3. `git diff --stat` で変更ファイル一覧を取得

以下の REVIEW CONTEXT を全レビュアーのプロンプトに注入する:

```
REVIEW CONTEXT:
- delta-spec: openspec/changes/<change-name>/specs/[ファイル一覧]
- design.md: openspec/changes/<change-name>/design.md
- 変更ファイル: [git diff --stat の出力]

REVIEW INSTRUCTION:
1. まず delta-spec と design.md を Read し、設計意図を理解すること
2. 設計上の意図的な選択を「問題」として指摘しないこと
3. 各指摘に「関連する仕様項目」を明記すること（仕様外の指摘は明示すること）
4. 各指摘に確信度（HIGH / MEDIUM / LOW）を付与すること
```

### Step 2: 動的レビュアー選択

`git diff --stat` の出力からドメインを検出し、起動するレビュアーを動的に決定する:

| ドメイン検出条件 | 起動するレビュアー |
|---|---|
| 常時起動（全変更） | **security-sentinel** |
| `.ts` / `.tsx` ファイルが含まれる | **type-safety-reviewer** |
| `src/app/` 配下の `.tsx` ファイルが含まれる | **architecture-strategist**, **performance-oracle** |
| `src/app/api/` または `src/actions/` 配下のファイルが含まれる | **api-contract-reviewer** |
| `.prisma` ファイルまたは `prisma/` 配下のファイルが含まれる | **prisma-guardian** |
| `.tf` ファイルまたは `terraform/` 配下のファイルが含まれる | **terraform-reviewer** |

**注意**: security-sentinel は常時起動。その他は条件に該当する場合のみ起動する。
該当しないレビュアーは起動しない（トークン節約とノイズ低減）。

### Step 3: レビュアー並列実行

Step 2 で選択されたレビュアーを**並列で** Task として起動する。
各レビュアーには Step 1 で準備した REVIEW CONTEXT を注入する。

**レビュアーの役割（参考）:**

1. **security-sentinel**: OWASP Top 10、シークレット検出、XSS/CSRF、認証・認可の穴、Terraformセキュリティ（IAM、ファイアウォール）
2. **performance-oracle**: PrismaのN+1クエリ、Next.jsバンドルサイズ、不要な再レンダリング、キャッシュ戦略、Server Components最適化
3. **architecture-strategist**: コンポーネント境界、責務分離、App Router規約準拠、Route Handlers設計、レイヤー構成
4. **prisma-guardian**: マイグレーション安全性、参照整合性、クエリ最適化、インデックスカバレッジ、トランザクション境界
5. **terraform-reviewer**: IaCベストプラクティス、GCPリソース設定、ステート管理、ドリフト検出、セキュリティグループ
6. **type-safety-reviewer**: TypeScript strictモード準拠、Zodスキーマ検証、any型の排除、型の整合性（Server/Client間、API層）
7. **api-contract-reviewer**: Route Handlers / Server Actionsの入出力型整合性、エラーレスポンス統一、バリデーション

### Step 4: review-aggregator による統合

レビュアー並列実行の完了後、**review-aggregator** を Task として起動する。

review-aggregator には以下を入力として提供する:
- 全レビュアーの出力テキスト
- delta-spec ファイルパス一覧
- design.md ファイルパス

review-aggregator は以下を実行する:
1. **重複排除**: 同一箇所への複数指摘を最も具体的なものに統合
2. **矛盾解決**: 相反する指摘をフラグ
3. **優先度調整**: 確信度に基づく横断的再評価。LOW 確信度の P2/P3 はノイズ候補として分離
4. **カバレッジマトリクス生成**: delta-spec の各要件・シナリオに対するレビューカバレッジを可視化

### Step 5: 結果提示と修正ループ

review-aggregator の統合レポートをユーザーに提示し、以下のフローで対応する:

#### P1 あり（修正ループ）

```
P1 指摘あり
  │
  ├── アーキテクチャ変更を伴う（エスカレーションフラグあり）
  │   → AskUserQuestion でユーザーに修正方針を確認
  │
  └── アーキテクチャ変更不要
      → Task(implementer) で自動修正
      → 修正部分のみ、関連レビュアーのみで再レビュー（最大 1 回）
      → 再レビューで新たな P1 が出た場合はユーザーに報告（無限ループ防止）
```

#### P2 のみ

ユーザーに修正するかどうかの判断を委ねる。

#### P3 のみ

レポートのみ出力。

### Step 6: レビュー学習ループ

レビュー結果の提示後、ユーザーの反応を記録する:

1. ユーザーが指摘を**却下**した場合（修正不要と判断）:
   - 却下理由をユーザーに確認
   - `docs/compound/` に以下の形式で記録:
     ```
     category: review-gap
     レビュアー: [レビュアー名]
     指摘ID: [SECURITY-001 等]
     却下理由: [ユーザーの理由]
     ```

2. 同一パターンの却下が **2 回以上**蓄積した場合:
   - Learning Router が該当レビュアーのチェックリスト更新を提案
   - 例: 「architecture-strategist が Server Actions の直接 DB アクセスを常に指摘するが、このプロジェクトでは設計上許容している」
     → architecture-strategist のチェックリストにプロジェクト固有の例外を追加提案

3. ノイズ候補（LOW 確信度 P2/P3）が却下された場合:
   - `docs/compound/` に `category: review-gap` で記録
   - 同一パターンの蓄積でレビュアー改善提案に昇格

## レビュー出力形式

review-aggregator の出力形式に従う。最終的にユーザーに提示する形式:

```markdown
# コードレビュー結果（Spec-Aware Review）

## サマリー
- P1（修正必須）: X件
- P2（修正推奨）: X件
- P3（あると良い）: X件
- ノイズ候補（低確信度）: X件
- 矛盾検出: X件
- 起動レビュアー: [security-sentinel, type-safety-reviewer, ...]

## P1: クリティカル
### [SECURITY-001] SQLインジェクションの可能性
- **重要度**: P1
- **確信度**: HIGH
- **ファイル**: `src/app/api/users/route.ts:42`
- **問題**: ユーザー入力が直接クエリに渡されている
- **修正案**: Prismaのパラメータ化クエリを使用
- **レビュアー**: security-sentinel
- **関連仕様**: REQ-001 (Error Scenario)

## P2: 重要
...

## P3: 軽微
...

## 矛盾検出
...

## ノイズ候補（低確信度 P2/P3）
...

## Review Coverage Matrix

| 仕様項目 | Security | Performance | Architecture | Type Safety | API Contract | カバー状態 |
|---|---|---|---|---|---|---|
| REQ-001: Happy Path | - | PERF-001 | - | TYPE-002 | - | Covered |
| REQ-001: Error Scenario | SEC-003 | - | - | - | API-001 | Covered |
| REQ-002: Boundary | - | - | - | - | - | **UNCOVERED** |

### 未カバー項目
- REQ-002: Boundary Scenario -- どのレビュアーも検証していない。追加レビューを推奨。
```
