[<< back](../README-ja.md) | [Japanese] | [English](strfry-en.md)

# strfry

## 概要
- **言語**: C++
- **設定ファイル**: `strfry.conf`
- **リポジトリ**: https://github.com/hoytech/strfry
- **確認バージョン**: `b80cda3a812af1b662223edad47eb70b053508b6` (2026-06-22)

## limit パラメータ

**デフォルト最大limit**: [500](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L111)

**設定パラメータ**: `relay.maxFilterLimit`

```conf
relay {
    # フィルターごとに返せる最大レコード数
    maxFilterLimit = 500
}
```

**動作**:
- クライアントが `limit: N` でイベントをリクエストした場合、strfryはフィルターごとに **min(N, 500)** 件のイベントを返す
- クライアントがlimitを指定しないか、500より大きいlimitを指定した場合、strfryは500に制限
- これはサブスクリプションごとではなく、フィルターごとの制限

## レート制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大サブスクリプション数 | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L120) | `relay.maxSubsPerConnection` |
| イベント送信レート | デフォルトでは未設定 | 設定可能 |
| フィルター/REQレート | 未設定 | 設定可能 |
| 接続レート | 未設定 | 設定可能 |

**備考**: `relay.maxSubsPerConnection` で接続あたりの同時REQ数を制限。イベント送信/フィルター/接続レートはコア設定としては未設定（外部のリバースプロキシ等で対応）

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+900秒 (15分)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L24) |
| 最大過去オフセット | [-94,608,000秒 (約3年)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L27) |

### イベント保存/削除ポリシー

| 項目 | 値 |
|------|-----|
| 一時イベント経過時間 | [60秒](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L30)より古い場合拒否 |
| 一時イベント生存期間 | [300秒](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L33)後に自動削除 |
| 通常イベント最大経過時間 | - |

**備考**: 一時イベント関連の制限はKind 20000-29999のみに適用。`rejectEventsNewerThanSeconds`/`rejectEventsOlderThanSeconds` で `created_at` の未来・過去オフセットを検証し範囲外を拒否

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | 制限なし | - |
| REQあたり最大フィルター数 | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L99) | `relay.maxReqFilterSize` |
| 最大authors数 (概算) | ~1,900 (WebSocket制限による) | - |

**備考**: 明示的なフィルター値制限はなく、WebSocketペイロード（128 KB）等のメッセージサイズ制限が実質的な上限となる。`relay.maxTagsPerFilter`（デフォルト3）によりフィルターあたりのタグフィルター数も制限される

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| イベントサイズ | [65,536バイト (64 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L21) | `events.maxEventSize` |
| WebSocketペイロード | [131,072バイト (128 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L96) | `relay.maxWebsocketPayloadSize` |
| タグ値サイズ | [1,024バイト](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L39) | `events.maxTagValSize` |
| 最大タグ数 | [2,000](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L36) | `events.maxNumTags` |

## サポートNIP

1, 2, 4, 9, 11, 28, 40, 45, 70, 77

---
[<< back](../README-ja.md)
