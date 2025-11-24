# 既存コードの再利用分析

このドキュメントは、既存リポジトリから再利用可能なコードを分析した結果をまとめています。

## 1. confluence-gas-toolkit からの再利用

**リポジトリパス**: `/Users/eitarofutakuchi/source_code/ops-tools/confluence-gas-toolkit`

### 再利用可能なコンポーネント

#### 1.1 ConfluenceClient (`src/clients/confluence-client.ts`)

**機能**:
- Confluence REST API v2 への HTTP リクエスト送信
- GAS 環境と Node.js/Bun 環境の両方に対応
- シングルトンパターンによるインスタンス管理

**主要メソッド**:
- `getPage(pageId)`: ページ情報を取得
- `getSearchPage(extraCql, option)`: CQL クエリによるページ検索
- `callApi<T>(method, endpoint, body)`: 汎用 API 呼び出し

**再利用の方針**:
- ✅ **そのまま使える**: 基本的な API クライアント機能は完全に再利用可能
- ⚠️ **調整が必要**: 環境変数の取得方法を `packages/shared` に統一
- 📝 **拡張が必要**: ページ更新履歴取得メソッドの追加

```typescript
// 必要な追加メソッド（例）
async getPageHistory(pageId: string): Promise<Confluence.Version[]>
async getPageWithBody(pageId: string): Promise<Confluence.Content>
```

#### 1.2 HttpClient (`src/clients/http-client.ts`)

**機能**:
- GAS 環境 (`UrlFetchApp.fetch`) と ローカル環境 (`fetch`) の抽象化
- レスポンスの JSON パース
- ISO 8601 日時文字列の自動変換

**再利用の方針**:
- ✅ **そのまま使える**: 完全に再利用可能
- 📦 **配置先**: `packages/shared/src/http-client.ts`

#### 1.3 型定義 (`src/types/confluence.ts`)

**機能**:
- Confluence API のリクエスト・レスポンス型定義
- `fetch-confluence` パッケージの型を拡張

**再利用の方針**:
- ✅ **そのまま使える**: 型定義は完全に再利用可能
- 📦 **配置先**: `packages/shared/src/types/confluence.ts`
- 📝 **拡張が必要**: 新しいプロジェクトに必要な型を追加

#### 1.4 ユーティリティ関数 (`src/utils/`)

**再利用候補**:
- `toQueryString()`: オブジェクトをクエリ文字列に変換
- `getEnvVariable()`: 環境変数の取得
- `datetime.ts`: 日時処理ユーティリティ

**再利用の方針**:
- ✅ **部分的に再利用**: 必要なものを選択して `packages/shared/src/utils/` に配置

---

## 2. nira-jiro からの再利用

**リポジトリパス**: `/Users/eitarofutakuchi/source_code/ops-tools/nira-jiro`

### 再利用可能なコンポーネント

#### 2.1 Markdown パーサー (`src/parsers/markdown/`)

**機能**:
- Markdown → AST (Abstract Syntax Tree) への変換
- 標準 Markdown と Jira Markdown の両方をサポート
- ブロックレベル・インラインレベルの構文解析

**主要コンポーネント**:
```typescript
parseStandardMarkdown(source: string): MarkdownBlockNode[]
parseJiraMarkdown(source: string): MarkdownBlockNode[]
```

**再利用の方針**:
- ⚠️ **逆方向の変換が必要**: このコードは Markdown → DOM だが、今回は HTML → Markdown が必要
- 💡 **参考にする**: AST の構造設計やパーサーの実装パターンを参考にする
- ❌ **直接利用は困難**: 目的が異なるため、そのまま使うのは難しい

#### 2.2 AST ノード定義 (`src/parsers/markdown/common/ast/`)

**主要ノード**:
- **Block**: `HeadingNode`, `ParagraphNode`, `ListNode`, `CodeBlockNode`, `BlockquoteNode`
- **Inline**: `TextNode`, `StrongNode`, `EmphasisNode`

**再利用の方針**:
- ✅ **参考にする**: HTML → Markdown 変換時の出力構造設計に活用
- 💡 **アイデア**: Visitor パターンを使った変換ロジックの設計

#### 2.3 DomRendererVisitor (`src/parsers/markdown/common/ast/visitors/DomRendererVisitor.ts`)

**機能**:
- AST → DOM への変換
- Visitor パターンによる再帰的なレンダリング

**再利用の方針**:
- ⚠️ **逆方向の変換が必要**: Markdown → HTML だが、今回は HTML → Markdown
- 💡 **Visitor パターンを参考**: HTML パーサーで同様のパターンを採用

---

## 3. 新規実装が必要な機能

### 3.1 HTML → Markdown コンバーター

**必要な機能**:
- Confluence API から取得した HTML を Markdown に変換
- 優先度に応じた変換対応（Must / Want）

**実装方針**:
- GAS のデプロイサイズ制約のため、軽量な自作実装
- nira-jiro の AST 構造を参考にした設計
- 段階的な実装（Must 要素から開始）

**Must 要素**:
```typescript
// 変換が必須の HTML 要素
- <h1>~<h6> → # ~ ######
- <ul><li> / <ol><li> → - / 1.
- <table> → Markdown table
- <code> / <pre><code> → `code` / ```code```
- <a href=""> → [text](url)
- <img src=""> → ![alt](url)
```

**Want 要素**:
```typescript
// 余裕があれば対応
- <strong> / <b> → **text**
- <em> / <i> → *text*
- Confluence 独自マクロ → カスタム変換
```

### 3.2 Google Drive 管理

**必要な機能**:
- カテゴリフォルダの作成・管理
- Markdown ファイルの保存（frontmatter 付き）
- `_manifest.json` の生成・更新

**実装方針**:
- Google Drive API の直接利用
- GAS の DriveApp サービスを活用

### 3.3 CLI ツール

**必要な機能**:
- GAS API クライアント（HTTP 通信）
- ローカルマニフェスト管理
- バッチ処理（複数ページ同期）
- 差分同期
- 全文検索・フィルタリング

**実装方針**:
- Bun + TypeScript + Biome
- 新規実装（既存コードの再利用は限定的）

---

## 4. 推奨ディレクトリ構成

### packages/shared

```
packages/shared/
├── src/
│   ├── http-client.ts          # HttpClient (confluence-gas-toolkit)
│   ├── confluence-client.ts    # ConfluenceClient (confluence-gas-toolkit)
│   ├── types/
│   │   ├── confluence.ts       # Confluence 型定義 (confluence-gas-toolkit)
│   │   └── index.ts
│   └── utils/
│       ├── query-string.ts     # toQueryString (confluence-gas-toolkit)
│       ├── env.ts              # getEnvVariable (confluence-gas-toolkit)
│       └── index.ts
```

### packages/gas-backend

```
packages/gas-backend/
├── src/
│   ├── converters/
│   │   └── html-to-markdown.ts    # 新規実装（nira-jiro 参考）
│   ├── services/
│   │   ├── drive-manager.ts       # 新規実装
│   │   └── polling-handler.ts     # 新規実装
│   └── index.ts
```

### packages/cli

```
packages/cli/
├── src/
│   ├── commands/
│   │   ├── sync.ts              # 新規実装
│   │   ├── list.ts              # 新規実装
│   │   └── search.ts            # 新規実装
│   ├── lib/
│   │   ├── gas-client.ts        # 新規実装
│   │   ├── manifest.ts          # 新規実装
│   │   └── file-manager.ts      # 新規実装
│   └── index.ts
```

---

## 5. 次のステップ

1. ✅ `packages/shared` に共通コードをコピー・調整
2. ⏸️ `packages/gas-backend` で HTML → Markdown 変換を実装
3. ⏸️ `packages/cli` のセットアップ

---

## 参考リンク

- [confluence-gas-toolkit](https://github.com/niroe5tar64/confluence-gas-toolkit)
- [nira-jiro](https://github.com/niroe5tar64/nira-jiro)
- [Confluence REST API Documentation](https://developer.atlassian.com/cloud/confluence/rest/v2/intro/)
