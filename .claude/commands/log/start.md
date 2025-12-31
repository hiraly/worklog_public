---
description: "課題ログを新規作成する（自然言語ベース）。issues/YYYY/YYYY-MM-DD_<slug>/ を作って README.md をテンプレから生成する"
---

あなたは「課題ログ起票係」です。ユーザーの入力から課題フォルダを作成し、README.md をテンプレから生成してください。

## 0) 入力（自然言語ベース）
ユーザーは自然言語で入力します：

```
/log/start api-timeout-fix
/log/start CI hardening を段階導入
/log/start 認証処理のリファクタリング タイトル: ユーザー認証処理の改善
```

### 推定すべき情報

**slug**：
- 最初のキーワードまたは短いフレーズから生成
- 例：「api-timeout-fix」→ そのまま
- 例：「CI hardening を段階導入」→ `ci-hardening`
- slug が明示されていない場合：`log-<YYYYMMDD-HHMM>` を使用

**title**：
- 「タイトル:」「題:」の後の文字列
- 明示がなければ slug から生成、または入力全体を使用

## 1) slug 正規化
- 小文字化
- 空白は `-` に
- 許可: `a-z 0-9 - _`
- それ以外は削除
- 連続ハイフンは1つに畳む

## 2) ルート判定
- `worklog/` ディレクトリが存在すれば、それをルートとする
- なければ、リポジトリ直下をルートとみなす（`issues/` を直下に作る）

以後、`ROOT` は上記ルート。

## 3) 作成パス
- ローカル日付（YYYY-MM-DD）を取得する
- `YEAR = YYYY`
- `ISSUE_DIR = ROOT/issues/YEAR/YYYY-MM-DD_<slug>/`

中に `README.md` のみ作成（空ディレクトリは作成しない。他のディレクトリは使用時に作成する）

※ すでに同名フォルダが存在する場合は **上書きしない**。代わりに末尾に `-2` `-3` を付けて空き番を探す。

## 4) README.md 生成（テンプレ優先）
テンプレを次の優先順位で探す：

1. `ROOT/templates/issue.md`
2. なければ、このコマンド内の「内蔵テンプレ」を使う

### frontmatter ルール
- title: `title:"..."` があればそれ、なければ `<slug>`
- status: `in_progress`
- visibility: `private`
- created: 今日の日付（YYYY-MM-DD）
- tags: `[]`
- links: ticket/repo/pr は空文字でOK

## 5) index.yml（横断索引）は更新しない
- index.yml は `/log/index` が生成する（startでは触らない）

## 6) 出力（チャット）
- 作成した `ISSUE_DIR` と README.md パスを1〜3行で報告
- 次に埋めるべき最小項目（課題/成功条件/H1）を1行だけ促す

---

## 内蔵テンプレ（ROOT/templates/issue.md が無い場合に使用）
以下の内容で `README.md` を生成すること：

```md
---
title: "{{TITLE}}"
status: "in_progress" # in_progress | done | paused | published
visibility: "private" # private | internal | public_candidate
created: {{DATE}}
tags: []
links:
  ticket: ""
  repo: ""
  pr: ""
---

# 0. 要約（あとでAI更新してOK）
- 課題：
- ねらい（何が良くなる？誰が得する？）：
- 現状の痛み：
- 結果（現時点）：

# 1. 課題（Problem）
## 背景 / 文脈
## 制約（期限・コスト・権限・技術制約）
## ゴール / 非ゴール
## 成功条件（定量/定性）

**定量（Before → After で記録）：**
- 例：API応答時間 p95: 150ms → 80ms 以下
- 例：デプロイ頻度：週1回 → 週3回以上
- 例：エラー率：5% → 1% 以下

**定性（定性的な変化）：**
- 例：自分の作業効率が向上する
- 例：レビュー時の議論が活発になる

# 2. 仮説（Hypotheses）
- H1:
  - 期待する変化：
  - 反証条件：
  - 検証方法：

# 3. 検証（Experiments）
- E1:
  - 手順：
  - 取得したいログ/メトリクス：
  - 結果：
  - 次の一手：

# 4. 試行錯誤ログ（時系列 / raw）
## {{DATE}}
- [fact]
- [obs]
- [idea]
- [metric]（定量的な材料は[metric]タグで記録）
- [decision] (evidence: TODO)

# 5. 結果（Result）

## Before / After（数値）
| 指標 | Before | After | 達成率 |
|------|--------|-------|--------|
| （例）API応答時間 p95 | 150ms | 80ms | 188% |
| （例）デプロイ頻度 | 週1回 | 週3回 | 300% |

## 何が解決した / 何が残った
## 副作用・トレードオフ

# 6. 学び（Learnings）
- 再現可能な学び
- やらない方がよかったこと

# 7. 根拠・参照（Evidence）
- 計測ログ：
- PR/コミット：
- 画像：
- 外部リンク：

# 8. 公開用にするなら（Blog draft seed）
- 想定読者：
- 何を持ち帰ってほしいか：
- 構成案（見出しだけでも）：
```

README.md 生成時は {{TITLE}} を title に、{{DATE}} を YYYY-MM-DD に置換する。
