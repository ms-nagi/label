# label
issue用のラベル

ここからクローンして使う。
あえてPublic

Cloneコマンド：

```
gh label clone ms-nagi/label --repo OWNER/新リポジトリ
```

既存のLabelをすべて上書き（+削除）
```
gh label-sync sync --file labels.yml --repo [OWNER]/[repo] --force --delete-unmanaged
```
