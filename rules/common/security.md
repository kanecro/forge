# セキュリティ規約

- ハードコードされたシークレット・APIキー禁止
- 環境変数は `.env.local`（開発）、Secret Manager（本番）
- ユーザー入力は必ずZodでバリデーション
- SQLインジェクション防止: Prismaのパラメータ化クエリのみ使用
- XSS防止: `dangerouslySetInnerHTML` 禁止（例外は明示的なレビュー後のみ）
- CSRF: Server Actionsは自動保護、Route Handlersは明示的に対策
- 認証: middleware.tsでルートレベルの保護
- 依存関係: `pnpm audit` でゼロ脆弱性を維持
