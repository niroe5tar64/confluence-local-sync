---
title: "Confluence Local Sync"
description: "Confluence ドキュメントをローカル環境に同期し、AI エージェントが活用できるようにする業務効率化ツール"
---

## プロジェクト概要

社内 Confluence で管理されているドキュメントをローカル環境に同期し、AI エージェント（Cursor など）が MCP サーバー経由でアクセスできるようにする仕組みを構築します。

## 背景・課題

- 社内の主要な情報が Confluence に集約されている
- AI エージェントが社内情報にアクセスできない
- Confluence API から取得したデータには HTML タグなどノイズが多い
- 大量リクエストによるサーバー負荷を避ける必要がある
- 事業部として専用サーバーを建てることはできない制約

## 解決アプローチ

1. GAS（Google Apps Script）をバックエンドとして活用し、定期的に Confluence ページを取得・Markdown 変換します。
2. Markdown形式に変換した Confluence ページを Google Drive にキャッシュします。
3. ユーザーは CLI ツールで必要なページをローカルに同期します。

## システム構成

```mermaid
graph TB
    subgraph "Confluence Server"
        C[Confluence Pages]
    end

    subgraph "GAS Backend (定期実行)"
        G1[Confluence API Polling]
        G2[HTML → Markdown 変換]
        G3[Google Drive 保存]
    end

    subgraph "Google Drive"
        GD[Markdown Cache + Metadata]
    end

    subgraph "Local Environment"
        CLI[CLI Tool<br/>Bun + TypeScript]
        LOCAL[Local Markdown Files]
        MCP[MCP Server<br/>将来実装]
    end

    subgraph "AI Agent"
        AI[Cursor など]
    end

    C -->|定期ポーリング| G1
    G1 --> G2
    G2 --> G3
    G3 --> GD
    GD -->|CLI で同期| CLI
    CLI --> LOCAL
    LOCAL -.->|将来| MCP
    MCP -.->|将来| AI
```

## 技術スタック

- **バックエンド**: Google Apps Script (GAS)
- **CLI ツール**: Bun + TypeScript + Biome
- **ストレージ**: Google Drive
- **形式**: Markdown（Confluence HTML から変換）

## 開発フェーズ

1. **Phase 1**: GAS バックエンド + Markdown 変換
2. **Phase 2**: CLI ツール（同期機能）
3. **Phase 3**: ローカル MCP サーバー（AI エージェント連携）

## ドキュメント構成

- [01. アイデア・背景](./01_idea/) - なぜこのツールが必要か
- [02. 要件定義](./02_requirements/) - 機能要件・非機能要件
- [03. 技術仕様](./03_specifications/) - アーキテクチャ・実装詳細
