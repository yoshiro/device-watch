# device-watch

家の端末が止まっていないかを、GitHub Actions で外から見張る。

1. 各端末は10分ごとに、このリポジトリの変数 `HEARTBEAT_<端末>` を「`<UNIX秒> <表示名>`」で上書きする（例: `HEARTBEAT_SURFACE` = `1793270000 Surface`）
2. [`watch.yml`](.github/workflows/watch.yml) が30分ごとに全端末の合図を見て、30分以上途切れていれば LINE に「止まっているようです」、戻れば「復旧しました」を1回ずつ送る

端末側の仕組みはこのリポジトリには置かない。

## 端末を増やすとき

その端末に fine-grained PAT（このリポジトリだけ、Variables: Read and write・Actions: Read and write）を置き、10分ごとに次を実行させる。

```bash
gh variable set HEARTBEAT_<端末> -R yoshiro/device-watch --body "$(date +%s) <表示名>"
```

公開リポジトリの定期実行は、60日間動きが無いと GitHub に止められる。どれか1台が週に1回、次で有効に戻す。

```bash
gh api -X PUT repos/yoshiro/device-watch/actions/workflows/watch.yml/enable
```

## Secrets

| 名前 | 中身 |
|---|---|
| `WATCH_TOKEN` | fine-grained PAT（このリポジトリだけ、Variables: Read and write）。期限の2週間前から LINE で予告する |
| `LINE_CHANNEL_ACCESS_TOKEN` | LINE Messaging API のチャネルアクセストークン |
| `LINE_TO` | 送り先のグループ ID |

## 使用量

公開リポジトリの GitHub Actions は無料で、時間の上限が無い。
