# label
issue用のラベル

ここからクローンして使う。
あえてPublic

## label-sync を使うときは GH_TOKEN が必須

`gh label-sync` は `gh` 本体とは別の実行ファイルで、**gh が keyring（macOS キーチェーン）に
保存したトークンを読めない**。`gh auth status` が `Token: gho_... (keyring)` と表示する環境では
拡張は未認証のままリクエストを投げるため、次の症状になる。

- ラベルの**読み取りは成功する**（このリポジトリは Public なので匿名でも GET が通る）
- `--dry-run` も成功し、「7 label(s) to create」などと正常に見える
- しかし**書き込みだけが `HTTP 404: Not Found` で失敗する**（未認証の書き込みに GitHub は 404 を返す）

そのため label-sync 系のコマンドには必ず `GH_TOKEN` を明示的に渡すこと。

```
GH_TOKEN="$(gh auth token)" gh label-sync ...
```

`gh label clone` は `gh` 本体のサブコマンドなので keyring から読める。こちらは不要。

## 前提：配布元のラベルを最新にしておく

`gh label clone` がコピーするのは **このリポジトリの GitHub 上に登録されている実ラベル** であって、
`labels.yml` の中身ではない。`labels.yml` を更新したら、まずこのリポジトリ自身へ sync して
両者を一致させること。これを忘れると、追加したばかりのラベルが clone 先に入らない。

```
GH_TOKEN="$(gh auth token)" gh label-sync sync --file labels.yml --repo ms-nagi/label --force --delete-unmanaged --yes
```

## Cloneコマンド

```
gh label clone ms-nagi/label --repo OWNER/新リポジトリ --force
```

`--force` が無いと、clone 先に同名ラベルが既にある場合はスキップされ、色や説明が更新されない。
また clone 先にしか無いラベルは削除されない。全消しして揃えたい場合は下の label-sync を使う。

## labels.yml から同期する（既存のものはすべて消す、初回用）

以下のコマンド使用には label-sync が必要
```
gh extension install scttfrdmn/gh-label-sync
```

```
curl -sL https://raw.githubusercontent.com/ms-nagi/label/refs/heads/main/labels.yml -o /tmp/labels.yml
GH_TOKEN="$(gh auth token)" gh label-sync sync --file /tmp/labels.yml --repo [OWNER]/[REPO] --force --delete-unmanaged --yes
```

`--yes` は確認プロンプト（`? Apply changes? (y/N)`）をスキップするフラグ。
これを省くと、TTY の無い環境（スクリプト・CI・エージェント経由の実行）ではプロンプトに
応答できず `Cancelled.` となり、1件も反映されないまま exit 0 で終了する。
手元のターミナルで対話的に実行する場合のみ省略してよい。

反映前に差分だけ見たいときは `--dry-run` を付ける。ただし前述のとおり dry-run は未認証でも
通ってしまうため、dry-run が成功しても書き込めるとは限らない点に注意。
