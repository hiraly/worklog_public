# Worklog 設計詳細

## 1. 設計原則

### 1.1 ログの層別管理
- "全部を最初からストーリーで書く"は重いので、ログを層に分ける
  - 層1：raw（事実・観測・アイデア・意思決定・学習記録）…軽く、捏造なし
  - 層2：ストーリー（ブログ下書き）…後でLLMが編集して良い

### 1.2 自然言語ベース
- 形式的なパラメータ（`[key:value]`）は廃止
- ユーザーは自然言語で指示し、Claude が意図を推定
- タグ（`[why]`, `[stuck]` 等）も自動推定

### 1.3 学習記録の統合
- **その場で記録**：作業中に疑問や躓きをリアルタイムで記録
- **学習の可視化**：何がわかっていないかを捕まえる
- **復習サイクル**：日次・週次・月次で振り返り、繰り返しパターンを検出

### 1.4 エビデンス管理
- 一次情報を README に貼り付けず、Source Card に封じ込める
- README には `src:<id>` の参照だけを残す
- 事実の新規生成は禁止（ログがないものは「ない」）

---

## 2. タグ一覧（自動推定）

### 作業・観測
| タグ | 用途 |
|------|------|
| `[fact]` | 実施したこと（コマンド/変更/対応/作業） |
| `[obs]` | 観測結果（エラー、数値、反応、現象） |
| `[idea]` | 次に試すこと、仮説、改善案 |

### 学習記録
| タグ | 用途 |
|------|------|
| `[why]` | わからなかったこと、理由が不明なこと、疑問 |
| `[tried]` | 試したこと、探索、実験 |
| `[stuck]` | ハマったこと、躓き、ブロッカー |
| `[learned]` | 理解できたこと、学び、気づき |

### メトリクス
| タグ | 用途 |
|------|------|
| `[metric]` | 定量的な材料、効果測定の数値（Before/After） |

### 意思決定
| タグ | 用途 |
|------|------|
| `[decision]` | 意思決定（priority なし） |
| `[decision:final]` | 確定（覆らない前提） |
| `[decision:tentative]` | 暫定（検証中） |

---

## 3. Source Card

### 3.1 概要
一次情報（チャット/ドキュメント/課題管理/Git）を `sources/<type>/<id>.md` に保存し、README からは `src:<id>` で参照する。

### 3.2 テンプレート

```markdown
---
id: chat-01
type: chat             # chat | wiki | issue | git
visibility: internal   # private | internal | public_candidate
url: <permalink>
retrieved_at: 2025-12-21T10:15+09:00
version: ""            # wiki: version / issue: updatedAt or key / git: commit hash
---

## 何のために参照する？
- 決定事項の根拠 / 仕様の前提 / 未収束論点 など

## 抜粋（必要最小限）
- （貼りすぎない。決定や論点に必要な一部だけ）

## 要点（事実ベース）
- 決定:
- 論点:
- 未確定:
```

### 3.3 抜粋ルール
- chat: 関連する発言のみ（全スレッドをコピペしない）
- wiki: 該当セクションの要点のみ
- issue: Description 要点 + 重要なコメント
- git: PR description + 関連コミットメッセージ（diff は貼らない）

長すぎる場合（目安：100行超）：
1. カードには要点のみ記載
2. 全文が必要なら `artifacts/raw_dump_<id>.md` に退避

---

## 4. 運用フロー

### 4.1 日常の記録
1. **課題起票**：`/log/start <自然言語>` で住所を作る
2. **作業中の記録**：`/log/record <自然言語>` でその場で記録
3. **ソース記録**：`/log/source <自然言語>` で一次情報を保存

### 4.2 振り返り（復習サイクル）
- **翌朝（5分）**：`/log/review 昨日の振り返り`
- **週末（30分）**：`/log/review 今週2回以上出た疑問`
- **月末（1時間）**：`/log/review 今月の学習を振り返って`

### 4.3 定期メンテナンス
- **週末 or 区切り**：`/log/index` で全課題横断索引を更新
- **検索**：`/log/find <自然言語>` で過去ログを検索

---

## 5. 全課題横断索引（index.yml）

### 5.1 方針
- `index.yml` は `/log/index` で生成する（手編集しない）

### 5.2 構造例

```yaml
issues:
  - slug: api-timeout-fix
    path: issues/2025/2025-12-21_api-timeout-fix
    title: "API タイムアウト対策"
    status: done
    tags: [performance, api, timeout]
    visibility: public_candidate
    sources_summary:
      - "chat-01: API制約の議論"
    key_decisions:
      - "A案（キャッシュ追加）で進める"
    result_summary: "p95を150ms→80msに改善"
```

### 5.3 生成ルール
- 各課題の README frontmatter + sources.yml を読み取り
- `key_decisions` は `[decision:final]` から抽出
- `result_summary` は Result セクションから1行要約
- 課題は新しい順に並べる

---

## 6. メタ課題の扱い方

技術課題に直接紐づかない活動（目標設定、キャリア相談など）も記録可能。

```bash
/log/start 2025年Q1の目標設定 visibility:private tags:meta,career
```

- `visibility: private` でテックブログ化の対象外に
- 既存の `/log/*` コマンドがそのまま使える
