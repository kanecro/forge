---
name: forge-skill-orchestrator
description: "Use at the START of every task, before writing any code, running tests, debugging, reviewing, or designing. This skill determines which methodology and domain skills must be invoked. MUST be invoked before any implementation, debugging, review, testing, or specification work begins."
---

# Forge Skill Orchestrator

## 1% ルール

**1% でも適用される可能性があれば、そのスキルを呼び出せ。**

スキルの呼び出しコストは低い。呼び出さなかったことによる品質低下コストは高い。
迷ったら呼び出す。迷わなくても呼び出す。

## フェーズ検出テーブル

現在のコマンド名または作業内容からフェーズを判定する:

| コマンド / 作業内容 | フェーズ |
|---|---|
| `/brainstorm` | design |
| `/spec` | spec |
| `/implement` | implementation |
| `/review` | review |
| `/test` | test |
| `/compound` | documentation |
| `/ship` | all（フェーズ遷移あり） |
| バグ修正・エラー対応 | debug |
| コード変更・新機能追加 | implementation |
| コードレビュー依頼 | review |
| テスト実行・修正 | test |
| 設計・要件整理 | design / spec |

## ドメイン検出テーブル

対象ファイルのパスパターンからドメインを判定する:

| ファイルパスパターン | ドメイン |
|---|---|
| `src/app/**/*.tsx`, `src/components/**/*.tsx` | nextjs-frontend |
| `src/app/api/**/*.ts`, `src/actions/**/*.ts` | typescript-backend |
| `prisma/**/*`, `*.prisma` | prisma-database |
| `terraform/**/*`, `*.tf` | terraform-infrastructure |
| `src/**/*.ts` (上記以外) | typescript-backend |
| 複数ドメインにまたがる場合 | 該当する全ドメインの Union |

## Skill レジストリ

### Methodology Skills（universal -- 全ドメイン共通）

| Skill 名 | 適用フェーズ | トリガー条件 |
|---|---|---|
| `test-driven-development` | implementation, debug | コードを書く前。新機能実装、バグ修正、リファクタリング時 |
| `systematic-debugging` | debug, implementation, test | バグ、テスト失敗、ビルドエラー、予期しない動作の発生時 |
| `verification-before-completion` | ALL（完了境界） | タスク完了宣言の直前。「完了」と言う前に必ず |
| `iterative-retrieval` | ALL | サブエージェントとして起動された時。コードベース探索の開始時 |
| `strategic-compact` | ALL | コンテキストウィンドウ 80% 超過時。フェーズ切り替え時。大量出力処理後 |

### Domain Skills（将来追加枠）

| Skill 名 | 適用ドメイン | トリガー条件 |
|---|---|---|
| `nextjs-frontend` | nextjs-frontend | `.tsx`/`.jsx` ファイル、`src/app/` 配下の変更時 |
| `typescript-backend` | typescript-backend | Route Handlers、Server Actions の変更時 |
| `prisma-database` | prisma-database | Prisma スキーマ、マイグレーション、クエリの変更時 |
| `terraform-infrastructure` | terraform-infrastructure | `.tf` ファイルの変更時 |

> **注**: Domain Skills は現時点では未作成。将来追加時にこのレジストリに登録することで、自動的にフェーズ+ドメイン判定の対象になる。

## サブエージェント向け指示

サブエージェント（Task ツールで起動されるエージェント）は `Skill` ツールを直接使用できない。そのため:

1. **親コマンド（`/implement`, `/spec` 等）の責務**:
   - サブエージェントを起動する前に、エージェント定義ファイルの `skills` frontmatter を確認する
   - 該当する各 SKILL.md の内容を読み込む
   - サブエージェントのタスクプロンプトに Skill の内容をインラインで含める

2. **プロンプト挿入テンプレート**:
   ```
   ## 適用スキル

   以下のスキルの指示に従って作業すること:

   ### [Skill名]
   [SKILL.md の本文をここに展開]
   ```

3. **サブエージェントの責務**:
   - プロンプトに含まれた Skill の指示に従う
   - Skill の指示に違反する作業を行わない

## 決定フローチャート

```
START
  │
  ├─ 1. フェーズ判定
  │     └─ コマンド名 or 作業内容 → フェーズ検出テーブル → phases[]
  │
  ├─ 2. ドメイン判定
  │     └─ 対象ファイルパス → ドメイン検出テーブル → domains[]
  │
  ├─ 3. レジストリ照合
  │     ├─ Methodology Skills: phases[] でフィルタ → matched_methodology[]
  │     └─ Domain Skills: domains[] でフィルタ → matched_domain[]
  │
  ├─ 4. Union
  │     └─ skills_to_invoke = matched_methodology ∪ matched_domain
  │
  ├─ 5. 1% ルール適用
  │     └─ 「本当に除外してよいか？」を各 Skill について確認
  │     └─ 疑わしければ追加
  │
  └─ 6. 呼び出し
        ├─ メインセッション: `Skill` ツールで各スキルを呼び出す
        └─ サブエージェント: SKILL.md の内容をタスクプロンプトに含める
```

## 使用例

### 例 1: `/implement` 実行時

1. フェーズ: `implementation`
2. ドメイン: 対象ファイルから判定（例: `src/app/dashboard/page.tsx` → `nextjs-frontend`）
3. Methodology Skills: `test-driven-development`, `verification-before-completion`, `iterative-retrieval`
4. Domain Skills: `nextjs-frontend`（存在する場合）
5. → 4つの Skill を呼び出し

### 例 2: テスト失敗のデバッグ時

1. フェーズ: `debug`
2. ドメイン: エラー発生ファイルから判定
3. Methodology Skills: `systematic-debugging`, `test-driven-development`, `iterative-retrieval`
4. Domain Skills: 該当ドメイン（存在する場合）
5. → 3〜4つの Skill を呼び出し
