# Worklog

エンジニアリング活動を「テックブログに書ける前提」で記録するローカルログ基盤。
Claude Code の Slash Commands で運用する。

## コマンド一覧

| コマンド | 用途 |
|----------|------|
| `/log/start <課題名>` | 課題フォルダを作成 |
| `/log/record <メモ>` | 作業ログ・学習記録を追記 |
| `/log/source <URL> <メモ>` | 一次情報を Source Card 化 |
| `/log/find <キーワード>` | 過去ログを検索 |
| `/log/review <期間>` | 振り返り・パターン検出 |
| `/log/draft <課題名>` | ブログドラフト生成 |
| `/log/index` | 全課題索引を更新 |

## ディレクトリ構成

```
issues/
  2025/
    2025-12-21_api-timeout-fix/
      README.md        # 課題ログ本体
      sources/         # 一次情報（Source Card）
      drafts/          # ブログ下書き
      artifacts/       # 成果物・データ
```

- 1課題 = 1フォルダ（`issues/YYYY/YYYY-MM-DD_<slug>/`）
- 空ディレクトリは作成しない（使用時に作成）

## クイックスタート

```bash
# 課題を起票
/log/start api-timeout-fix

# 作業中にメモ
/log/record タイムアウトの原因はコネクションプールの枯渇だった

# 振り返り
/log/review 今週の振り返り
```

## 詳細

- [設計詳細](docs/design.md) - 設計原則、タグ一覧、Source Card 仕様
- [課題テンプレート](templates/issue.md) - 課題ログの雛形
