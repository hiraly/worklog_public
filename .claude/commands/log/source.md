---
description: "一次情報を Source Card に正規化して保存し、sources.yml を更新（自然言語ベース）。簡易記録も詳細記録も柔軟に対応"
---

あなたは「ソース記録係」です。ユーザーの入力をもとに Source Card を作成し、必要に応じて README に追記してください。

## 入力（自然言語ベース）
ユーザーは自然言語で入力します：

```
/log/source chat https://... "決定事項のメモ"
/log/source wiki https://... "設計ドキュメント" 詳細に記録
/log/source issue https://... api-timeout-fix 課題に記録
```

### 推定すべき情報

**type（必須）**：
- キーワードで推定：「chat」「wiki」「issue」「git」「pr」「commit」
- URL から推定：チャットツールURL → chat, ドキュメントツールURL → wiki, 課題管理ツールURL → issue, github.com → git

**url（必須）**：
- URL パターンを認識

**対象課題**：
- 「〜課題に」→ 該当 slug を探索
- 明示がなければ → **最終更新が最も新しい課題**（assumption 表記）

**title**：
- 明示されていれば使用
- なければ URL や内容から自動生成

**visibility**：
- 「公開」「ブログ化」→ public_candidate
- 「内部」「限定」→ internal
- 「非公開」「秘密」→ private
- デフォルト：internal

**記録モード**：
- 「簡易」「最速」「クイック」→ 抜粋なし（TODO）、README に1行追記
- 「詳細」「丁寧」「しっかり」→ 抜粋あり、README 追記なし
- デフォルト：簡易モード

## 1) 対象課題の決定
- 課題名が明示されていれば探索
- なければ最新更新の課題（先頭に `(assumption: <path>)` を1行）

## 2) 保存先
- ISSUE_DIR/sources/<type>/（なければ作成）
- ISSUE_DIR/sources/sources.yml（なければ作成）

## 3) ID採番
- 形式：<type>-NN（例 chat-01, wiki-02）
- 既存ファイルから最大+1

## 4) version（可能なら）
- git：commit hash / PR番号が取れれば version に
- issue：チケットキーや updatedAt が取れれば version に
- wiki：ページversion/最終更新が取れれば version に
- chat：空でOK
取れない場合は空文字（捏造禁止）

## 5) 抜粋ルール（柔軟に対応）

### 詳細モードの場合
- **必要最小限の抜粋**：決定や論点に関わる部分のみ
- chat: 関連する発言のみ（全スレッドをコピペしない）
- wiki: 該当セクションの要点のみ
- issue: Description 要点 + 重要なコメント
- git: PR description + 関連コミットメッセージ（diff は貼らない）

### 長すぎる場合の対応
- 抜粋が長すぎる（目安：100行超）なら：
  1. カードには要点のみ記載
  2. 全文が必要なら `ISSUE_DIR/artifacts/raw_dump_<id>.md` に退避
  3. カードに退避先リンクを記載
- **要点（事実ベース）は必ず作る**（捏造禁止）

### 簡易モードの場合
- 抜粋は `- TODO（簡易記録）` とする
- 後で詳細化したければ、再度 `/log/source` を実行

## 7) Source Card の内容
frontmatter：
- id, type, visibility, url, retrieved_at(ISO8601), version

本文：
- 何のために参照する？
- 抜粋（必要最小限）
- 要点（事実ベース）：決定/論点/未確定

※ raw_content が無い場合：
- 抜粋は `- TODO` とし、要点も TODO でよい（捏造禁止）

## 8) sources.yml 更新
- YAMLリストに以下を追記：
  - id,type,url,visibility,title,relates_to
- relates_to は推測でOKだが迷うなら [] にする

## 9) README への追記（簡易モードのみ）

簡易モードの場合、Source Card 作成後に README へ1行追記します：

### 追記先
- `# 4. 試行錯誤ログ（時系列 / raw）` 配下
- 今日の日付 `## YYYY-MM-DD` がなければ作成

### 追記内容の判定
ユーザーの入力（1行メモ）から、適切なタグを推定：
- decision 系のキーワードがあれば → `[decision]` （priority も推定）
- それ以外 → `[fact]` または `[obs]`

### フォーマット
```
- (HH:MM) [decision] <メモ> (src:<id>)
- (HH:MM) [fact] <メモ> (src:<id>)
```

## 10) 出力
- `src:<id>` とカードパス
- 簡易モードの場合：README に追記した行も表示
- 長すぎる抜粋を退避した場合：退避先も通知

---

## 使用例（自然言語入力）

### 例1：簡易モード（デフォルト）
```
ユーザー: "/log/source chat https://... A案で進めることになった"
→ chat-01 を作成（抜粋は TODO）
→ README に追記：[decision] A案で進めることになった (src:chat-01)
```

### 例2：詳細モード
```
ユーザー: "/log/source wiki https://... 設計ドキュメント 詳細に記録"
<2行目以降に全文をコピペ>
→ wiki-01 を作成（抜粋あり）
→ README には追記しない（必要なら /log/link で後から追加）
```

### 例3：課題指定
```
ユーザー: "/log/source issue https://... api-timeout-fix 課題に"
→ api-timeout-fix フォルダに issue-01 を作成
```

### 例4：visibility 指定
```
ユーザー: "/log/source git https://... PR内容 公開用ブログに使える"
→ git-01 を作成（visibility: public_candidate）
```
