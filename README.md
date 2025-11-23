# Confluence Local Sync

Confluence ドキュメントをローカル環境に同期し、AI エージェント（Cursor など）が MCP サーバー経由でアクセスできるようにする業務効率化ツール

## 概要

社内 Confluence で管理されているドキュメントを、Google Drive をキャッシュ層として活用しながらローカル環境に同期します。AI エージェントが社内情報を参照しながら作業できるようにすることで、開発効率の向上を目指します。

## システム構成

```
Confluence → GAS Backend → Google Drive Cache → CLI Tool → Local Files → MCP Server → AI Agent
```

- **GAS Backend**: 定期的に Confluence ページを取得し、Markdown 形式に変換して Google Drive に保存
- **CLI Tool**: ユーザーが必要なページをローカルに同期（バッチ処理、差分同期対応）
- **MCP Server** (Phase 3): ローカル Markdown ファイルを AI エージェントに提供

## 主な機能

- ✅ Confluence ページの自動取得・Markdown 変換
- ✅ Google Drive によるキャッシュ管理
- ✅ バッチ処理による複数ページの一括同期
- ✅ 差分同期（更新されたページのみ取得）
- ✅ Personal Access Token による2段階アクセス制御
- ✅ 全文検索・カテゴリフィルタリング

## 技術スタック

### GAS Backend
- TypeScript → JavaScript (ビルド後)
- Google Apps Script V8 Runtime
- Confluence REST API v2
- Google Drive API

### CLI Tool
- TypeScript
- Bun (ランタイム)
- Biome (リント/フォーマット)

## プロジェクト構成

このプロジェクトはモノレポ構成を採用しています：

```
confluence-local-sync/
├── packages/
│   ├── gas-backend/     # GAS バックエンド
│   ├── cli/             # CLI ツール
│   └── shared/          # 共通ライブラリ
├── docs/                # プロジェクトドキュメント
├── CLAUDE.md            # Claude Code 用ガイダンス
└── README.md            # このファイル
```

## ドキュメント

詳細な設計・仕様については、以下のドキュメントを参照してください：

- [プロジェクト概要](./docs/index.md)
- [アイデア・背景](./docs/01_idea.md)
- [要件定義](./docs/02_requirements.md)
- [技術仕様](./docs/03_specifications.md)

## 開発フェーズ

- **Phase 1**: GAS Backend + Markdown 変換
- **Phase 2**: CLI ツール（同期機能）
- **Phase 3**: ローカル MCP サーバー（AI エージェント連携）

## ライセンス

Private - 社内利用のみ

## 既存コードの活用

このプロジェクトは以下のリポジトリのコードを部分的に使い回しています：

- `confluence-gas-toolkit`: Confluence API クライアント
- `nira-jiro`: Markdown パーサー・レンダラー

詳細は [CLAUDE.md](./CLAUDE.md) を参照してください。
