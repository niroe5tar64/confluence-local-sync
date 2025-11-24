# プロジェクトコンテキスト

このファイルは、開発セッション間でコンテキストを共有するためのドキュメントです。

## 現在の状態（2025-11-24）

### ✅ 完了したタスク

1. **プロジェクトドキュメント作成**
   - `docs/index.md`: プロジェクト概要
   - `docs/01_idea.md`: アイデア・背景
   - `docs/02_requirements.md`: 要件定義
   - `docs/03_specifications.md`: 技術仕様
   - `docs/code-reuse-analysis.md`: 既存コード再利用分析

2. **リポジトリセットアップ**
   - モノレポ構成のディレクトリ作成（`packages/{gas-backend,cli,shared}`）
   - `CLAUDE.md`: Claude Code 用ガイダンス
   - `README.md`: プロジェクト概要

3. **既存コード調査**
   - `confluence-gas-toolkit`: Confluence API クライアント、HTTP クライアント、型定義
   - `nira-jiro`: Markdown パーサー（参考用）

### 📂 リポジトリ構成

```
confluence-local-sync/
├── packages/
│   ├── gas-backend/     # GAS バックエンド（未実装）
│   ├── cli/             # CLI ツール（未実装）
│   └── shared/          # 共通ライブラリ（未実装）
├── docs/                # プロジェクトドキュメント ✅
│   ├── index.md
│   ├── 01_idea.md
│   ├── 02_requirements.md
│   ├── 03_specifications.md
│   └── code-reuse-analysis.md
├── .claude/
│   └── context.md       # このファイル
├── CLAUDE.md            # Claude Code 用ガイダンス ✅
└── README.md            # プロジェクト概要 ✅
```

## 次のセッションでやること

### Phase 1: 共通ライブラリのセットアップ（`packages/shared`）

**目的**: `confluence-gas-toolkit` から再利用可能なコードを移行

**タスク**:
1. `packages/shared` のセットアップ
   ```bash
   cd packages/shared
   bun init
   # package.json, tsconfig.json を設定
   ```

2. 以下のファイルをコピー・調整:
   - `src/http-client.ts` ← `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit/src/clients/http-client.ts`
   - `src/confluence-client.ts` ← `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit/src/clients/confluence-client.ts`
   - `src/types/confluence.ts` ← `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit/src/types/confluence.ts`
   - `src/utils/` ← `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit/src/utils/`

3. 必要な調整:
   - 環境変数取得方法の統一
   - ページ履歴取得メソッドの追加
   - ページ本文取得メソッドの追加

**参考ドキュメント**: `docs/code-reuse-analysis.md` の「4. 推奨ディレクトリ構成」セクション

### Phase 2: GAS Backend の実装（`packages/gas-backend`）

**目的**: Confluence ページの取得・Markdown 変換・Google Drive 保存

**タスク**:
1. プロジェクトセットアップ
2. HTML → Markdown コンバーターの実装
   - Must 要素: 見出し、リスト、テーブル、コードブロック、リンク、画像
   - Want 要素: 装飾、Confluence 独自記法
3. Google Drive 管理機能の実装
   - カテゴリフォルダ管理
   - Markdown ファイル保存（frontmatter 付き）
   - `_manifest.json` 生成・更新
4. 定期ポーリングハンドラーの実装
5. HTTP エンドポイントの実装（CLI からのリクエスト処理）

**参考**:
- `nira-jiro` の AST 構造、Visitor パターン
- `docs/02_requirements.md` の「1.2 HTML → Markdown 変換」
- `docs/03_specifications.md` の「データモデル」

### Phase 3: CLI Tool の実装（`packages/cli`）

**目的**: ローカル同期、マニフェスト管理、検索機能

**タスク**:
1. プロジェクトセットアップ（Bun + TypeScript + Biome）
2. GAS API クライアントの実装
3. マニフェスト管理機能の実装
4. 同期コマンドの実装（単一ページ、バッチ処理）
5. 差分同期機能の実装
6. 検索・フィルタリング機能の実装

## 重要な設計決定

### 1. セキュリティ

- **システム管理者 PAT**: GAS のスクリプトプロパティで管理（定期ポーリング用）
- **ユーザー PAT**: ローカル環境変数/設定ファイルで管理（CLI 同期用）
- **アクセス制御**: 両方の PAT でアクセス可能なページのみ配信

### 2. データ保存形式

- **Markdown ファイル**: frontmatter にメタデータを保存
  ```markdown
  ---
  pageId: "123456789"
  title: "ページタイトル"
  version: 42
  lastModified: "2025-11-23T10:30:00Z"
  ---

  # ページ内容
  ```

- **カテゴリ管理ファイル**: 各カテゴリフォルダに `_manifest.json` を配置
  ```json
  {
    "category": "画面仕様書",
    "lastUpdated": "2025-11-23T12:00:00Z",
    "pages": {
      "123456789": {
        "pageId": "123456789",
        "title": "ページタイトル",
        "version": 42,
        "lastModified": "2025-11-23T10:30:00Z",
        "fileName": "page-name.md"
      }
    }
  }
  ```

### 3. 変換優先度

**Must（MVP で実装）**:
- 見出し（h1-h6）
- リスト（ul, ol）
- テーブル
- コードブロック
- リンク
- 画像（URL のみ）

**Want（余裕があれば）**:
- 基本的な装飾（太字、斜体）
- Confluence 独自記法（マクロ、ステータスラベル、パネル）

### 4. CLI 仕様

- ページ指定: ページ ID、カテゴリ名のみ（URL 指定は実装しない）
- バッチ処理: 初期段階から実装
- 検索・フィルタリング: 初期段階から実装

## 既存リポジトリへの参照

### confluence-gas-toolkit
- **パス**: `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit`
- **再利用**: ConfluenceClient, HttpClient, 型定義, ユーティリティ

### nira-jiro
- **パス**: `/Users/eitarofutakuchi/source_code/ops-tools/nira-jiro`
- **参考**: Markdown パーサーの設計、AST 構造、Visitor パターン
- **注意**: Markdown → HTML の変換だが、今回は逆方向（HTML → Markdown）

## 開発時のコマンド

```bash
# 新しいセッションを開始
cd /Users/eitarofutakuchi/source_code/ops-tools/confluence-local-sync

# ドキュメントを確認
cat CLAUDE.md
cat docs/code-reuse-analysis.md

# 共通ライブラリのセットアップ（Phase 1）
cd packages/shared
bun init
```

## トラブルシューティング

### モノレポ管理

このプロジェクトはモノレポ構成ですが、ワークスペース管理ツール（Turborepo, pnpm workspace など）は未導入です。必要に応じて、ルートの `package.json` でワークスペース設定を追加してください。

### GAS のデプロイ

GAS Backend は clasp を使ってデプロイします。`.clasp.json` の設定は `confluence-gas-toolkit` を参考にしてください。

## 関連 Issues / PRs

現時点ではなし。

## 更新履歴

- 2025-11-24: 初版作成（プロジェクトセットアップ完了時点）
