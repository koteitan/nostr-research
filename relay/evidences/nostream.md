[<< back](../README-ja.md) | [Japanese] | [English](nostream-en.md)

# nostream

## 概要
- **言語**: TypeScript
- **設定ファイル**: `resources/default-settings.yaml`
- **リポジトリ**: https://github.com/cameri/nostream
- **確認バージョン**: `6a8ccb491973b2470e3b332bef61ed7ea1143091` (2024-10-23)

## limit パラメータ

**デフォルト最大limit**: [5000](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L162)

**設定パラメータ**: `limits.client.subscription.maxLimit`

```yaml
limits:
  client:
    subscription:
      maxSubscriptions: 10
      maxFilters: 10
      maxFilterValues: 2500
      maxSubscriptionIdLength: 256
      maxLimit: 5000
      minPrefixLength: 4
```

**動作**:
- クライアントが `limit: N` でイベントをリクエストした場合、nostreamは **min(N, 5000)** 件のイベントを返す
- `maxLimit: 5000` 設定が許可される最大limit値を制御
- 以下も強制:
  - クライアントごとの最大同時サブスクリプション数: 10
  - サブスクリプションごとの最大フィルター数: 10
  - 合計最大フィルター値: 2500

## レート制限

### イベントkindごとのレート制限

| Kind | イベント送信レート | 説明 |
|------|-------------------|------|
| [0](https://github.com/nostr-protocol/nips/blob/master/01.md), [3](https://github.com/nostr-protocol/nips/blob/master/02.md), [40](https://github.com/nostr-protocol/nips/blob/master/28.md), [41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [6 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L108-L115) | メタデータ、コンタクト、チャンネルイベント |
| [1](https://github.com/nostr-protocol/nips/blob/master/01.md), [2](https://github.com/nostr-protocol/nips/blob/master/01.md), [4](https://github.com/nostr-protocol/nips/blob/master/04.md), [42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [12 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L116-L123) | ノート、DM、チャンネルメッセージ |
| [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md), [43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [30 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L158-L131) | 削除、リアクション、チャンネルイベント |
| [10000-19999, 30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [24 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L132-L141) | 置換可能イベント |
| [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L142-L147) | 一時イベント |
| 全イベント | [720 events/hour](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L148-L150) | 全体制限 |

### その他の制限

| 項目 | 値 |
|------|-----|
| 最大サブスクリプション数 | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L158) |
| フィルター/REQレート | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) |
| 接続レート | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) |

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+900秒 (15分)](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L90) |
| 最大過去オフセット | 制限なし |

## サイズ制限

| 項目 | 値 |
|------|-----|
| ネットワークペイロード | 524,288バイト (512 KB) |
| イベントコンテンツ | kind範囲0-10、40-49、11-39、50-maxで102,400バイト (100 KB) |

## サポートNIP

01, 02, 04, 09, 11, 11a, 12, 13, 15, 16, 20, 22, 28, 33, 40

## 追加機能

- 決済プロセッサ統合 (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL)
- イベントkindごとの詳細なレート制限

---
[<< back](../README-ja.md)
