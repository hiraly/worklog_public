# Worklog 設計 全体計画（Claude Code中心 / 自然言語ベース）v2.1

このリポジトリは、エンジニアリングの活動を「テックブログに書ける前提」で記録するためのローカルログ基盤です。
日々の作業を"住所付き"で残し、「課題 → 仮説 → 検証 → 試行錯誤 → 結果」というストーリーへ後から編集できる状態を作ります。
また、**リアルタイムの学習記録**と**復習サイクル**を組み込み、自己学習を加速させます。
チャット / ドキュメント / 課題管理 / Git などの一次情報を Evidence（根拠）として追跡可能にします。

---

## 0. 目的

- エンジニアリングの活動を、テックブログに書ける前提で記録する
- 「課題 → 仮説 → 検証 → 試行錯誤 → 結果」へ後から編集できるログを作る
- **リアルタイムで学習の穴を記録し、復習サイクルで理解を深める**（日報システムの統合）
- Claude Code を主戦場にしつつ、チャット / ドキュメント / 課題管理 / Git を Evidence として追跡可能にする

---

## 1. 設計原則（失敗しやすい点への対策）

### 1.1 ログの層別管理
- "全部を最初からストーリーで書く"は重いので、ログを層に分ける
  - 層1：raw（事実・観測・アイデア・意思決定・**学習記録**）…軽く、捏造なし
  - 層2：ストーリー（ブログ下書き）…後でLLMが編集して良い

### 1.2 自然言語ベース
- **形式的なパラメータ（`[key:value]`）は廃止**
- ユーザーは自然言語で指示し、Claude が意図を推定
- タグ（`[why]`, `[stuck]` 等）も自動推定（ユーザーは覚えなくて良い）

### 1.3 学習記録の統合
- **その場で記録**：作業中に疑問や躓きをリアルタイムで記録
- **学習の可視化**：何がわかっていないかを捕まえる
- **復習サイクル**：日次・週次・月次で振り返り、繰り返しパターンを検出

### 1.4 エビデンス管理
- 一次情報（チャット/ドキュメント/課題管理/Git）を README に貼り付けず、Source Card に封じ込める
- README には `src:<id>` の参照だけを残し、根拠追跡と公開可否の切り分けを容易にする
- 事実の新規生成は禁止（ログがないものは「ない」）

---

## 2. ディレクトリ構成

### 2.1 SSoT（Single Source of Truth）

- ローカルの Markdown / YAML が正（SSoT）
- 1課題 = 1フォルダ（住所）

```
worklog/
  README.md
  index.yml              # 全課題横断索引（生成物）
  templates/
    issue.md
  issues/
    2025/
      2025-12-21_<slug>/
        README.md        # 必須（課題起票時に作成）
        artifacts/       # 必要に応じて作成
        data/            # 必要に応じて作成
        llm/             # 必要に応じて作成
        drafts/          # 必要に応じて作成
        sources/         # 必要に応じて作成
          sources.yml
          chat/
          wiki/
          issue/
          git/
```

**注記：** 空ディレクトリは作成しない。README.md 以外は使用時に初めて作成する。

### 2.2 1課題 = 1フォルダ

- `issues/YYYY/YYYY-MM-DD_<slug>/` が「住所」
- 実験ログ・根拠（リンク/ログ/PR/スクショ）はこのフォルダに寄せる
- 迷子（根拠どこ？）を防ぎ、公開用編集のときに分離しやすい

---

## 3. 課題ログ README.md の必須テンプレ（要点）

README は「ストーリーの骨格 + raw（時系列）」を持ちます。  
frontmatter（YAML）には最低限以下を持ちます。

- `status` / `visibility` / `links`（ticket, repo, pr） / `tags`

セクション構成（推奨）：

1. 要約（AI更新OK）
2. Problem（背景/制約/成功条件）
3. Hypotheses（反証条件つき）
4. Experiments（手順・取得ログ・結果）
5. raw（時系列）：作業・観測・学習・意思決定を記録
   - 作業系：`[fact][obs][idea]`
   - 学習系：`[why][tried][stuck][learned]`
   - 意思決定：`[decision]` / `[decision:final]` / `[decision:tentative]`
6. Result / Learnings / Evidence / Blog draft seed

### タグ一覧（自動推定）

**作業・観測**
- `[fact]`：実施したこと（コマンド/変更/対応/作業）
- `[obs]`：観測結果（エラー、数値、反応、現象）
- `[idea]`：次に試すこと、仮説、改善案

**学習記録**
- `[why]`：わからなかったこと、理由が不明なこと、疑問
- `[tried]`：試したこと、探索、実験
- `[stuck]`：ハマったこと、躓き、ブロッカー
- `[learned]`：理解できたこと、学び、気づき

**メトリクス（v2.1で追加）**
- `[metric]`：定量的な材料、効果測定の数値（Before/After）
  - 例：API応答時間 p95: 150ms → 80ms
  - 例：デプロイ頻度：週1回 → 週3回

**意思決定**
- `[decision]`：意思決定（priority なし）
- `[decision:final]`：確定（覆らない前提）
- `[decision:tentative]`：暫定（検証中）

---

## 4. 入力ソース（チャット / ドキュメント / 課題管理 / Git）のログ化ポリシー

### 4.1 Source Card で正規化

- READMEに貼るのではなく `sources/<type>/<id>.md` に保存
- READMEは `src:<id>` 参照のみ（必要最小限の要点だけを raw/decision に反映）

#### Source Card テンプレ（Markdown）

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

### 4.2 Source Card 抜粋ルール（柔軟に対応）

**基本方針：必要最小限の抜粋**
- chat: 関連する発言のみ（全スレッドをコピペしない）
- wiki: 該当セクションの要点のみ
- issue: Description 要点 + 重要なコメント
- git: PR description + 関連コミットメッセージ（diff は貼らない）

**長すぎる場合の対応**
- 抜粋が長すぎる（目安：100行超）なら：
  1. カードには要点のみ記載
  2. 全文が必要なら `artifacts/raw_dump_<id>.md` に退避
  3. カードに退避先リンクを記載

### 4.3 READMEへの反映ルール

- raw：
  - (HH:MM) `[obs] ... (src:chat-01)`
  - (HH:MM) `[idea] ... (src:issue-02)`
- decision：
  - (HH:MM) `[decision] A案で進める / 理由: ... (evidence: src:wiki-01)`
  - (HH:MM) `[decision:final] A案で進める / 理由: ... (evidence: src:wiki-01)`
  - (HH:MM) `[decision:tentative] B案も並行検証 / 理由: ... (evidence: src:issue-02)`

#### decision priority の意味

- `[decision:final]` = 確定（覆らない前提）
- `[decision:tentative]` = 仮決定（検証中）
- `[decision]` = priority なし（中間扱い）

---

## 5. 運用フロー（日報システム統合版）

### 5.1 日常の記録（リアルタイム）
1. **課題起票**：`/log/start <自然言語>` で住所を作る（中身は薄くてOK）
2. **作業中の記録**：`/log/record <自然言語>` でその場で記録
   - 疑問：「なぜ〜」→ `[why]` と自動判定
   - 試行：「〜を試した」→ `[tried]` と自動判定
   - 躓き：「ハマった」→ `[stuck]` と自動判定
   - 理解：「わかった」→ `[learned]` と自動判定
3. **ソース記録**：`/log/source <自然言語>` で一次情報を保存
   - デフォルト：簡易モード（抜粋なし、README に1行追記）
   - 詳細モード：「詳細に記録」と明示

### 5.2 振り返り（復習サイクル）
- **翌朝（5分）**：`/log/review 昨日の振り返り`
- **週末（30分）**：`/log/review 今週2回以上出た疑問`
- **月末（1時間）**：`/log/review 今月の学習を振り返って`

### 5.3 定期メンテナンス
- **週末 or 区切り**：`/log/index` で全課題横断索引を更新
- **検索**：`/log/find <自然言語>` で過去ログを検索

### 5.4 メタ課題の扱い方

技術課題に直接紐づかない活動（1on1、目標設定、キャリア相談など）も、通常の課題と同じように記録できます。

**記録方法：**
- 通常の課題と同じく `/log/start` で課題を作成
- `visibility: private` にしてテックブログ化の対象外とする
- `tags: [meta, career, 1on1]` などでメタ課題であることを明示

**例：**
```bash
/log/start 2025年Q1の目標設定と1on1記録 visibility:private tags:meta,career,1on1
```

**メリット：**
- 既存の `/log/*` コマンドがそのまま使える
- `/log/review` で技術的な取り組みと一緒に振り返れる
- `/log/find` で検索可能
- 意思決定の記録として `[decision]` タグが使える

---

## 6. Slash Commands（.claude/commands/）一覧

**全コマンドが自然言語ベース。形式的なパラメータ（`[key:value]`）は不要。**

### 6.1 課題管理

**`/log/start <自然言語>`**
- 課題フォルダ生成 + README テンプレ投入（住所作成）
- 例：`/log/start api-timeout-fix`
- 例：`/log/start CI hardening を段階導入 タイトル: CI の段階的強化`

**`/log/record <自然言語>`**
- 作業ログ・学習記録を README に追記
- タグ（`[why]`, `[stuck]`, `[learned]` 等）は自動推定
- 例：`/log/record なぜRustで所有権が必要なのかわからない`
- 例：`/log/record A案で確定。理由はAPI制約のため`

### 6.2 ソース記録

**`/log/source <自然言語>`**
- 一次情報を Source Card 化（簡易/詳細モード自動判定）
- 例：`/log/source chat https://... 決定事項のメモ`
- 例：`/log/source wiki https://... 設計ドキュメント 詳細に記録`

### 6.3 検索・振り返り

**`/log/find <自然言語>`**
- 課題ログを検索（index.yml 検索 + 全文検索を自動選択）
- 例：`/log/find タイムアウト`
- 例：`/log/find 12月の決定事項でキャッシュに関するもの`
- 例：`/log/find 繰り返しハマっていること`

**`/log/review <自然言語>`**
- 日次・週次・月次の振り返り
- 繰り返しパターンを検出、未解決の疑問を抽出
- メトリクスと進捗の可視化（v2.1で追加）
- 例：`/log/review 今日の振り返り`
- 例：`/log/review 今週2回以上出た疑問`
- 例：`/log/review 今週のメトリクス変化`
- 例：`/log/review 今週何をして、どんな効果があったか`

**`/log/draft <自然言語>`（v2.1で追加）**
- テックブログドラフトの自動生成
- 材料の充足度を評価、不足項目を警告
- ストーリー構成を AI が提案
- 例：`/log/draft worklog-system-design`
- 例：`/log/draft 最新の課題`

### 6.4 索引生成

**`/log/index`**
- 全 README frontmatter + sources.yml を集約して `worklog/index.yml` を生成
- パラメータなし（生成物・手編集禁止）

---

## 7. 全課題横断索引（worklog/index.yml）

### 7.1 方針

- `worklog/index.yml` は生成物（`/log/index` で更新）
- 手編集しない（差分が壊れるため）

### 7.2 構造（例）

issues:
  - slug: api-timeout-fix
    path: issues/2025/2025-12-21_api-timeout-fix
    title: "API タイムアウト対策"
    status: done
    tags: [performance, api, timeout]
    visibility: public_candidate
    sources_summary:
      - "chat-01: API制約の議論"
      - "issue-02: タイムアウトチケット"
    key_decisions:
      - "A案（キャッシュ追加）で進める"
    result_summary: "p95を150ms→80msに改善"

### 7.3 生成ルール（/log/index）

- 各課題の README frontmatter + sources.yml を読み取り
- `key_decisions` は `[decision:final]` から抽出
- `result_summary` は Result セクションから1行要約
- 課題は新しい順に並べる

---

## 8. 変更履歴

### v2.1（2025-12-22）- メトリクス記録とドラフト生成

**主な変更点：**

1. **メトリクス記録タグ `[metric]` の追加**
   - 定量的な材料を明示的に記録
   - Before/After を可視化
   - 例：`[metric] API応答時間 p95: 150ms → 80ms`

2. **効果測定テンプレートの強化**
   - `/log/start` のテンプレートに Before/After 表を追加
   - 成功条件に定量指標を記載するよう促進

3. **テックブログドラフト自動生成**
   - `/log/draft` コマンドを新設
   - 材料の充足度を評価、不足項目を警告
   - ストーリー構成を AI が提案

4. **進捗追跡とメトリクス振り返りの強化**
   - `/log/review` に進捗追跡機能を追加
   - メトリクス変化の可視化
   - `[fact]` タグで進捗を抽出

**コマンド数：** 5 → 6（`/log/draft` 追加）

### v2.0（2025-12-22）- 大規模リファクタリング

**主な変更点：**

1. **自然言語ベースへの移行**
   - 形式的なパラメータ（`[key:value]`）を全廃
   - ユーザーは自然言語で指示、Claude が意図を推定
   - タグも自動推定（`[why]`, `[stuck]` 等）

2. **学習記録機能の統合**
   - 日報システムの思想を統合
   - 新タグ：`[why]` `[tried]` `[stuck]` `[learned]`
   - リアルタイム記録を重視

3. **復習サイクルの追加**
   - `/log/review` コマンド新設
   - 日次・週次・月次の振り返り
   - 繰り返しパターンの自動検出

4. **コマンド削減**（8 → 5）
   - 削除：`/log/quick`, `/log/link`, `/log/search`, `/log/grep`
   - 統合：`/log/source`（quick機能を含む）、`/log/find`（search/grep統合）
   - 新設：`/log/review`

5. **抜粋ルールの柔軟化**
   - 「3～5発言」などの数値指定を削除
   - 「必要最小限」という方針のみ

**移行ガイド：**
- 旧：`/log/record issue:xxx priority:final` → 新：`/log/record xxx課題に確定事項として記録`
- 旧：`/log/grep keyword --type=decision` → 新：`/log/find 決定事項でkeywordに関するもの`
- 旧：`/log/quick slack url "memo" issue:xxx` → 新：`/log/source slack url memo xxx課題に`

### v1.1（初期バージョン）
- 基本的な課題ログ機能
- Source Card
- decision priority（final/tentative）
- 8つのコマンド

---

## 9. 実装コマンド一覧（.claude/commands/log/）

現在実装されているコマンド：

- `start.md` - 課題起票（v2.1で効果測定テンプレート追加）
- `record.md` - 作業ログ・学習記録（v2.1で `[metric]` タグ追加）
- `source.md` - ソース記録（簡易/詳細）
- `find.md` - 検索
- `review.md` - 振り返り（v2.1で進捗・メトリクス追跡追加）
- `draft.md` - テックブログドラフト生成（v2.1で新設）
- `index.md` - 索引生成

---

## 10. 今後の拡張案

### 短期
- ~~テックブログドラフト自動生成~~ ✅ v2.1で実装
- MCP サーバーとの連携（チャット/ドキュメント/課題管理ツール自動取得）
- テンプレートの充実（`templates/issue.md` の実ファイル作成）

### 中期
- visibility の機械的除外（ブログ化時）
- ドラフトから公開記事への自動変換
- 会議メモモードの追加（口頭意思決定の記録促進）

### 長期
- 学習段階の可視化（遊→型→観→心→空）
- AI なしの時間の提案（認知的負荷の維持）
- メトリクスの自動グラフ化
