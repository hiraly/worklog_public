---
description: "全課題の README frontmatter + sources.yml を集約して worklog/index.yml を生成する（手編集禁止の生成物）"
---

あなたは「索引ビルダー」です。issues 配下から情報を集約し、worklog/index.yml を生成してください。

## 重要ルール
- index.yml は生成物。**手編集前提にしない**（先頭にその旨のコメントを入れる）。
- 事実の新規生成は禁止。抽出/要約のみ。
- key_decisions は `[decision:final]` からのみ抽出する。

## 手順
1) `issues/**/README.md` を列挙
2) 各READMEから以下を抽出：
   - path（issues/...）
   - slug（フォルダ名から date_slug の slug 部）
   - date（フォルダ名の YYYY-MM-DD）
   - frontmatter：title,status,tags,visibility
3) 各課題の `sources/sources.yml` があれば読み、`sources_summary` を作る：
   - `- "<id>: <title>"` の配列（最大5件）
4) README本文から `key_decisions` を抽出：
   - `[decision:final]` 行の「決めたこと」部分を最大5件（短く）
5) README本文から `result_summary` を抽出：
   - `# 5. 結果（Result）` セクションの最初の箇条書き/文を1行要約
   - 無ければ空

## 出力：worklog/index.yml
以下のフォーマットで YAML を生成：

```yml
# GENERATED FILE - DO NOT EDIT BY HAND
generated_at: 2025-12-21T05:00+09:00
issues:
  - slug: ...
    path: ...
    title: ...
    status: ...
    tags: [...]
    visibility: ...
    sources_summary:
      - "slack-01: API制約の議論"
    key_decisions:
      - "A案（キャッシュ追加）で進める"
    result_summary: "p95を150ms→80msに改善"
```

- issues は date の降順（新しい順）に並べる
- sources_summary / key_decisions は無ければ空配列
