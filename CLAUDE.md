# CLAUDE.md -- Forge 統合開発ワークフローシステム

## Core Philosophy

1. **Explore-First**: 変更前に必ず既存コードを読んで理解する。推測でコードを書かない
2. **Plan Before Execute**: 3ステップ以上の作業はタスクリストを作成してから実行する
3. **Minimal Change**: 依頼された変更のみ実施。過剰な改善・リファクタ・コメント追加をしない
4. **Action-Aware**: 現在のフェーズに合った作業を行う（実装中に仕様変更しない等）
5. **Skill-First**: 作業開始前に `forge-skill-orchestrator` で適用スキルを判定し、呼び出す

---

## Forge ワークフロー

### コマンドパイプライン

```
/brainstorm → /spec → /implement → /review → /test → /compound
     │            │         │           │         │         │
  proposal.md  delta-spec  TDD実装   7並列    全テスト   学び記録+
              design.md   RED→GREEN  レビュー  実行証明  スペックマージ
              tasks.md    →REFACTOR
```

- `/ship` は上記を連鎖実行する完全自律パイプライン
- `/brainstorm` と `/spec` の後はユーザー承認必須
- `/implement` 以降は自律実行（テスト失敗時は最大3回リトライ）

### OpenSpec 構造

```
openspec/
├── project.md              # プロジェクトコンテキスト
├── specs/                  # 累積スペック（マージ済みの正式仕様）
└── changes/                # 変更単位の作業ディレクトリ
    ├── <change-name>/      # アクティブな変更
    │   ├── proposal.md     # /brainstorm で生成
    │   ├── design.md       # /spec で生成
    │   ├── tasks.md        # /spec で生成
    │   └── specs/          # デルタスペック（/spec で生成）
    └── archive/            # /compound で完了分をアーカイブ
```

---

## Modular Rules

詳細なガイドラインは `~/.claude/rules/` を参照:

### 共通ルール（`rules/common/`）

| Rule File       | Contents                                                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| coding-style.md | ファイルサイズ制限（200-400行推奨/800行上限）、命名規約（PascalCase/camelCase/UPPER_SNAKE_CASE）、インポート順序、TODO形式、コメント規約                            |
| git-workflow.md | Conventional Commits形式（`<type>(<scope>): <説明>`）、ブランチ戦略（main/develop/feature/fix/chore）、コミット粒度、PR規約                                         |
| testing.md      | TDDワークフロー（RED→GREEN→REFACTOR）、Vitest/Playwright/Testing Library使い分け、AAA(Arrange/Act/Assert)パターン、カバレッジ目標80%+、テストファイル配置規約       |
| security.md     | シークレット管理（.env.local/Secret Manager）、Zodバリデーション必須、XSS/CSRF対策、Prismaパラメータ化クエリ、`dangerouslySetInnerHTML`禁止、`pnpm audit`ゼロ脆弱性 |
| performance.md  | Server Components優先、`next/image`必須、動的インポート、N+1防止（select+明示的リレーション）、キャッシュ戦略（revalidate/unstable_cache）、Web Vitals目標値        |
| escalation.md   | エスカレーション3段階（必須/状況依存/自律判断OK）、フェーズ別トリガー条件、エスカレーション形式テンプレート                                                         |

### 技術スタック固有ルール

| Rule File                | Contents                                                                                                                                                                               |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nextjs/conventions.md    | App Routerファイル構成（page/layout/loading/error/not-found/route）、Server/Client Components使い分け、Route Handlers/Server Actions設計、API統一レスポンス形式、generateMetadata設定  |
| prisma/conventions.md    | スキーマ命名規約（PascalCase/camelCase/@@map）、マイグレーション安全戦略（段階的破壊変更）、クエリ最適化（select/take必須）、インデックス設計、トランザクション境界                    |
| terraform/conventions.md | ファイル構成（main/variables/outputs/providers/backend）、モジュール化（GCPサービス単位）、リモートステート（GCS+ロック）、IAM最小権限、Cloud KMS暗号化、plan→review→applyワークフロー |

---

## Available Agents

エージェント定義は `~/.claude/agents/` を参照:

### リサーチエージェント（`agents/research/`）

| Agent                         | Purpose                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| stack-docs-researcher         | Context7 MCP経由で公式ドキュメントのベストプラクティスを取得 |
| web-researcher                | Web Search MCPで最新記事・落とし穴・参考実装を調査           |
| codebase-analyzer             | 既存コードのパターン・影響範囲・OpenSpecスペックを分析       |
| compound-learnings-researcher | `docs/compound/` から過去の学び・教訓を抽出                  |

### 実装エージェント（`agents/implementation/`）

| Agent                    | Purpose                                         |
| ------------------------ | ----------------------------------------------- |
| implementer              | タスク単位のTDD駆動実装（RED→GREEN→REFACTOR）   |
| spec-compliance-reviewer | デルタスペックとの照合・逸脱検出（model: opus） |
| build-error-resolver     | TypeScriptビルドエラーの最小差分修正            |

### レビューエージェント（`agents/review/`）-- 全て model: opus

| Agent                   | Purpose                                                           |
| ----------------------- | ----------------------------------------------------------------- |
| security-sentinel       | OWASP Top 10、シークレット検出、認証・認可、Terraformセキュリティ |
| performance-oracle      | N+1クエリ、バンドルサイズ、再レンダリング、キャッシュ戦略         |
| architecture-strategist | コンポーネント境界、責務分離、App Router規約準拠                  |
| prisma-guardian         | マイグレーション安全性、クエリ最適化、インデックスカバレッジ      |
| terraform-reviewer      | IaCベストプラクティス、GCPリソース設定、ステート管理              |
| type-safety-reviewer    | strict mode準拠、any排除、Zodスキーマ検証                         |
| api-contract-reviewer   | Route Handlers/Server Actions入出力型整合性、エラーレスポンス統一 |

---

## Available Skills

スキル定義は `~/.claude/skills/` を参照:

| Skill                          | Purpose                                                     |
| ------------------------------ | ----------------------------------------------------------- |
| forge-skill-orchestrator       | 作業開始時のスキル判定・ルーティング（1%ルール適用）        |
| test-driven-development        | TDD絶対ルール（RED→GREEN→REFACTOR、違反は削除やり直し）     |
| systematic-debugging           | 4フェーズデバッグ（再現→原因特定→修正→防御）                |
| verification-before-completion | 完了の証明（実行結果貼付必須、「通るはず」禁止）            |
| iterative-retrieval            | 段階的コンテキスト取得（Glob→Grep→Read）                    |
| strategic-compact              | コンテキストウィンドウ管理（80%超過時の手動コンパクション） |

---

## Hook 自動ガードレール

以下のフックが自動的に品質を守る。フック定義は `~/.claude/hooks/` を参照:

| Hook                     | Trigger | Action                                                                                |
| ------------------------ | ------- | ------------------------------------------------------------------------------------- |
| block-unnecessary-files  | Write前 | プロジェクトルートへの `.md`/`.txt` 作成をブロック（`docs/`, `openspec/` 配下は許可） |
| detect-console-log       | Write後 | `.ts`/`.tsx` 内の `console.log` を警告（`console.error`/`console.warn` は許可）       |
| require-tmux-for-servers | Bash前  | `pnpm dev` 等の長時間プロセスを tmux 外で実行するのをブロック                         |
| gate-git-push            | Bash前  | `git push --force` をブロック、通常 push 時にチェックリスト表示                       |

---

## Skill Orchestration（1% ルール）

**1% でも適用される可能性があれば、そのスキルを呼び出せ。**

作業開始前に必ず以下を実行:

1. フェーズ判定（コマンド名 or 作業内容 → フェーズ）
2. ドメイン判定（対象ファイルパス → ドメイン）
3. レジストリ照合 → 適用スキルを特定
4. 1% ルール適用 → 除外してよいか再確認

### サブエージェントへのスキル注入

サブエージェントは Skill ツールを使えない。親コマンドが:

1. エージェント定義の `skills` frontmatter を確認
2. 該当 SKILL.md を読み込み
3. タスクプロンプトにインラインで含める

---

## Escalation Rules

詳細は `~/.claude/rules/common/escalation.md` を参照。

### 判断フロー

1. CLAUDE.md / `rules/` のルールで解決できるか？ → Yes: 自律判断
2. 影響範囲は自モジュール内か？ → Yes: 自律判断
3. 上記すべて No → ユーザーに確認（`AskUserQuestion`）

### 常にエスカレーション（必須）

- **セキュリティ**: 認証・認可・暗号化・PII処理の設計判断
- **データ**: スキーマ変更・マイグレーション・データ整合性に影響する判断
- **アーキテクチャ**: 新サービス追加・破壊的API変更・レイヤー構造変更
- **本番環境**: デプロイ・設定変更・ロールバック

### 自律判断OK

- コードフォーマット、lint修正、ローカル変数リネーム
- 明らかなバグ修正（null例外、off-by-one）
- 自モジュール内のリファクタリング
- P3レビュー指摘の修正

---

## Personal Preferences

### Code Style

- TypeScript strict mode 準拠
- 既存のコード規約・パターンを踏襲
- コードやコメントにエモジを入れない

### Git

- コミット形式: `<type>(<scope>): <日本語の説明>`
- PR説明は日本語
- 小さく焦点を絞ったコミット
- コミット前に `git diff` でレビュー

### Quality

- テスト前にコードを書かない（TDD: RED → GREEN → REFACTOR）
- テストをスキップ・無効化して通過させない
- TODO/モック/スタブを本実装に残さない
- `pnpm tsc --noEmit` をコミット前に実行

---

## Compound Learning（100ドルルール）

防げたはずの失敗が起きたら:

1. `docs/compound/YYYY-MM-DD-<topic>.md` に学びを記録
2. コストが100ドル超（推定）なら、`rules/`・`skills/`・`hooks/` の更新を提案
3. ユーザー承認後に適用

---

## Learned: Tools & Runtime

（セッション中に学んだツール・ランタイム関連の知見をここに追記）

## Learned: Patterns

（セッション中に学んだコードパターン・設計知見をここに追記）

## Learned: Pitfalls

（セッション中に学んだ落とし穴・回避策をここに追記）
