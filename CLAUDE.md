# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリについて

GitHub Issue 用ラベル定義を一元管理するためのリポジトリ。アプリケーションコードは無く、
実質的な成果物は `labels.yml` 1ファイルのみ。ビルド・lint・テストの仕組みは存在しない。

配布元として `ms-nagi/label` の名前で **Public** 公開されている（`curl` で raw 取得できるようにするため意図的に Public）。
`main` にマージした内容がそのまま他リポジトリへ配布されるため、変更は即座に外部影響を持つ。

## 主要コマンド

他リポジトリへラベルを反映する（利用側で実行するもので、このリポジトリ内では実行しない）:

```bash
# 既存リポジトリからラベルをコピー
gh label clone ms-nagi/label --repo OWNER/REPO --force

# labels.yml を取得して同期（既存ラベルを全消しする初回用）
# 事前に gh 拡張が必要: gh extension install scttfrdmn/gh-label-sync
curl -sL https://raw.githubusercontent.com/ms-nagi/label/refs/heads/main/labels.yml -o /tmp/labels.yml
GH_TOKEN="$(gh auth token)" gh label-sync sync --file /tmp/labels.yml --repo OWNER/REPO --force --delete-unmanaged --yes
```

`labels.yml` を更新したときは、配布元であるこのリポジトリ自身にも同じ sync を
`--repo ms-nagi/label` で実行する（理由は下記）。

### label-sync に GH_TOKEN が必須な理由

`gh label-sync` は `gh` 本体とは別バイナリで、gh が keyring（macOS キーチェーン）に保存した
トークンを読めない。`gh auth status` が `Token: gho_... (keyring)` を表示する環境では
拡張は未認証でリクエストするため、Public リポジトリ相手だと**読み取りと `--dry-run` は成功するのに
書き込みだけが `HTTP 404: Not Found` で失敗する**。`GH_TOKEN="$(gh auth token)"` を前置して回避する。
`gh label clone` は gh 本体のサブコマンドなので keyring を読めるため不要。

### そのほかの落とし穴

`--delete-unmanaged` は `labels.yml` に無いラベルを削除する。つまり「このファイルに書かれていないラベルは
利用側リポジトリから消える」ことを前提に編集する。

`--yes` を省くと `? Apply changes? (y/N)` の確認プロンプトが出る。TTY の無い環境（スクリプト・CI・
エージェント経由の実行）ではこれに応答できず `Cancelled.` となり、1件も反映されないまま exit 0 で
終わるため、対話実行以外では必ず付ける。

`gh label clone` がコピーするのは `labels.yml` ではなく **このリポジトリの GitHub 上に登録済みの実ラベル**。
そのため `labels.yml` を変更してコミットしただけでは clone 先に反映されない。また `--force` が無いと
clone 先の同名ラベルはスキップされ、色や説明が更新されない。

## labels.yml の構造とルール

トップレベルは `labels:` のみで、各要素は `name` / `color` / `description` の3キーを持つ。

- **カテゴリ分け**: コメント（`# --- 状態(ワークフロー) ---` など）で
  `状態(ワークフロー)` / `優先度` / `種別` / `領域` / `エージェント制御(フラグ)` の5グループに区切られている。
  ラベル追加時は該当グループ内へ入れ、状態系はワークフローの進行順に並べる。
- **name**: 日本語。ただし領域ラベルのみ英大文字（`FRONTEND` / `BACKEND` / `INFRA`）、
  エージェント制御ラベルのみ `agent:` プレフィックス付きの英小文字（`agent:ready` など）。
  コロンを含む名前（全角の `"優先度：高"`、半角の `"agent:ready"` など）は
  YAML パースの都合でクォートする。
- **color**: `#` なしの6桁 hex。既存の色分けの意図は下記のとおりで、新規ラベルは
  同カテゴリ内の既存色と視覚的に紛れない色を選ぶ。
  - 待ち・保留系 … 黄〜橙（`d4a017`, `fb8c00`, `e8734d`, `fbca04`）
  - 異常・緊急 … 赤（`d73a4a`, `b60205`）
  - 領域 … 濃いティール〜紺（`0e8a6d`, `006b75`, `264653`）
  - 終了・無効 … グレー（`aaaaaa`, `cfd3d7`, `6e7781`）
- **エージェント制御ラベル**: 人間の運用ではなく、エージェントが読んで挙動を決めるためのフラグ。
  `agent:ready` → `agent:wip` → （失敗時 `agent:retry-1` → `agent:retry-2`）→ `agent:done` /
  `agent:asked`（ユーザへ確認中・回答後に再開）/ `agent:blocked`（人の判断待ちで停止）
  というライフサイクルを前提とし、`labels.yml` でもその順に並べる。
  リトライ段数を増やす場合は `agent:retry-N` の形式を踏襲する。
- **description**: 「どういう状態か」を1文で書く。GitHub のラベル一覧でそのまま表示される。
