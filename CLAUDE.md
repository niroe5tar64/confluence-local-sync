# CLAUDE.md

このファイルは、Claude Code がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

**Confluence Local Sync** は、社内 Confluence で管理されているドキュメントをローカル環境に同期し、AI エージェント（Cursor など）が MCP サーバー経由でアクセスできるようにする業務効率化ツールです。

## アーキテクチャ

このプロジェクトは **モノレポ構成** を採用しています：

```
confluence-local-sync/
├── packages/
│   ├── gas-backend/     # GAS バックエンド（Confluence API 連携、Markdown 変換）
│   ├── cli/             # CLI ツール（ローカル同期、マニフェスト管理）
│   └── shared/          # 共通ライブラリ（型定義、ユーティリティ）
├── docs/                # プロジェクトドキュメント
│   ├── index.md         # プロジェクト概要
│   ├── 01_idea.md       # アイデア・背景
│   ├── 02_requirements.md  # 要件定義
│   └── 03_specifications.md # 技術仕様
└── CLAUDE.md            # このファイル
```

## 技術スタック

### GAS Backend
- **言語**: TypeScript → JavaScript (ビルド後)
- **ランタイム**: Google Apps Script V8 Runtime
- **API**: Confluence REST API v2
- **ストレージ**: Google Drive API

### CLI Tool
- **言語**: TypeScript
- **ランタイム**: Bun
- **リント/フォーマット**: Biome
- **HTTP クライアント**: Bun の標準 fetch API

## 既存コードの活用

このプロジェクトは、以下の既存リポジトリのコードを部分的に使い回します：

### 1. confluence-gas-toolkit
- **パス**: `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit`
- **使い回す機能**:
  - Confluence API クライアント (`src/clients/confluence-client.ts`)
  - HTTP クライアント (`src/clients/http-client.ts`)
  - Confluence 型定義 (`src/types/confluence.ts`)
  - ユーティリティ関数 (`src/utils/`)

### 2. nira-jiro
- **パス**: `/Users/eitarofutakuchi/source_code/ops-tools/nira-jiro`
- **使い回す機能**:
  - Markdown パーサー・レンダラー (`src/parsers/markdown/`)
  - AST ノード定義 (`src/parsers/markdown/common/ast/`)
  - DOM レンダラー (`src/parsers/markdown/common/ast/visitors/DomRendererVisitor.ts`)

## 開発フェーズ

### Phase 1: GAS Backend (2-3週間)
- [ ] Confluence API 連携
- [ ] HTML → Markdown 変換ロジック
- [ ] Google Drive 保存機能
- [ ] HTTP エンドポイント実装

### Phase 2: CLI Tool (2-3週間)
- [ ] プロジェクトセットアップ (Bun + TypeScript + Biome)
- [ ] GAS API クライアント
- [ ] マニフェスト管理
- [ ] 同期コマンド実装
- [ ] エラーハンドリング

### Phase 3: MCP Server (2週間)
- [ ] MCP プロトコル実装
- [ ] ファイル検索機能
- [ ] AI エージェントとの連携テスト

## 重要な設計方針

### セキュリティ
- **Personal Access Token (PAT) の管理**:
  - システム管理者 PAT: GAS のスクリプトプロパティで管理
  - ユーザー PAT: ローカル環境変数や設定ファイルで管理
- **アクセス制御**:
  - システム管理者 PAT でアクセス可能（= Google Drive に保存済み）かつユーザー PAT でアクセス可能なページのみ配信

### パフォーマンス
- **差分同期**: マニフェストで管理し、更新されたページのみ同期
- **バッチ処理**: 複数ページを一度に処理し、HTTP リクエストのオーバーヘッドを削減
- **キャッシュ活用**: Google Drive をキャッシュ層として使用

### データモデル
- **Markdown ファイル**: frontmatter にメタデータを保存
- **カテゴリ管理ファイル**: 各カテゴリフォルダに `_manifest.json` を配置
- **ローカルマニフェスト**: `sync-state.json` でローカルの同期状態を管理

## 開発時の注意事項

### GAS のデプロイサイズ制約
- 外部ライブラリの使用を最小限に
- Markdown 変換ロジックは自作（既存の nira-jiro のコードを参考に GAS 用に最適化）

### Markdown 変換の優先度
- **[Must]**: 見出し、リスト、テーブル、コードブロック、リンク、画像
- **[Want]**: 基本的な装飾、Confluence 独自記法（マクロ、ステータスラベル、パネルなど）

### CLI コマンド設計
- 単一ページまたは複数ページを指定（ページ ID、カテゴリ名など）
- ページ URL での指定は実装しない（煩雑なため）

## 参考ドキュメント

詳細な仕様については、`docs/` ディレクトリ内のドキュメントを参照してください：
- [プロジェクト概要](./docs/index.md)
- [アイデア・背景](./docs/01_idea.md)
- [要件定義](./docs/02_requirements.md)
- [技術仕様](./docs/03_specifications.md)
