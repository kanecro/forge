# Gitワークフロー

## ブランチ戦略
- `main`: 本番環境
- `develop`: 開発環境
- `feature/<issue-id>-<short-description>`: 機能開発
- `fix/<issue-id>-<short-description>`: バグ修正
- `chore/<description>`: メンテナンス

## コミットメッセージ（Conventional Commits）
- `feat: 新機能の追加`
- `fix: バグ修正`
- `refactor: リファクタリング`
- `test: テスト追加・修正`
- `docs: ドキュメント更新`
- `chore: ビルド・ツール設定`
- `perf: パフォーマンス改善`

## コミット粒度
- 1コミット = 1つの論理的な変更
- テストと実装は同じコミットに含める
- 動く状態でコミットする（ビルドが壊れた状態でコミットしない）
