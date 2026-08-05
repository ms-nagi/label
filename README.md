# label
issue用のラベル

ここからクローンして使う。
あえてPublic

Cloneコマンド：

```
gh label clone ms-nagi/label --repo OWNER/新リポジトリ
```


このリポジトリから labels.yml を落としてきてそれをリポジトリに突っ込む（既存のものはすべて消す、初回用）
以下のコマンド使用には label-sync が必要
```
gh extension install scttfrdmn/gh-label-sync
```

```
curl -sL https://raw.githubusercontent.com/ms-nagi/label/refs/heads/main/labels.yml -o /tmp/labels.yml
gh label-sync sync --file /tmp/labels.yml --repo [OWNER]/[REPO] --force --delete-unmanaged
```
