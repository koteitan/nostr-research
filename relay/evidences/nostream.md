[<< back](../README-ja.md) | [Japanese] | [English](nostream-en.md)

# nostream

## 概要
- **言語**: TypeScript
- **設定ファイル**: `resources/default-settings.yaml`
- **リポジトリ**: https://github.com/cameri/nostream
- **確認バージョン**: `33f0ba98530d87a1e54ea1bd64481a425294021d` (2026-06-25)

## limit パラメータ

**デフォルト最大limit**: [5000](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L221)

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

イベント送信レートはイベントkindごとに細かく設定されている。今回のバージョンではMarmotグループイベント (kind 445) 用に60 events/minの行が追加された。レート制限戦略は `ewma` (指数加重移動平均)。

### イベントkindごとのレート制限

| Kind | イベント送信レート | 説明 |
|------|-------------------|------|
| [0](https://github.com/nostr-protocol/nips/blob/master/01.md), [3](https://github.com/nostr-protocol/nips/blob/master/02.md), [40](https://github.com/nostr-protocol/nips/blob/master/28.md), [41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [6 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L162-L169) | メタデータ、コンタクト、チャンネル作成/更新イベント |
| [1](https://github.com/nostr-protocol/nips/blob/master/01.md), [2](https://github.com/nostr-protocol/nips/blob/master/01.md), [4](https://github.com/nostr-protocol/nips/blob/master/04.md), [42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [12 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L170-L177) | ノート、DM、チャンネルメッセージ |
| [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md), [43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [30 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L178-L185) | 削除、リアクション、チャンネルイベント |
| [10000-19999, 30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [24 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L186-L194) | 置換可能イベント・パラメータ化置換可能イベント |
| [445](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L195-L199) | Marmotグループイベント (新規追加) |
| [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L200-L205) | 一時イベント |
| 全イベント | [720 events/hour](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L206-L208) | 全体制限 |

### その他の制限

| 項目 | 値 |
|------|-----|
| 最大サブスクリプション数 | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) |
| 生メッセージ(REQ含む)レート | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) |
| 接続レート | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) および [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) |

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+900秒 (15分)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L144) |
| 最大過去オフセット | [制限なし (0)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L145) |

**備考**: `createdAt.maxPositiveDelta: 900` で未来方向に最大15分まで許容。`maxNegativeDelta: 0` は過去方向の制限を無効化 (制限なし) を意味する。一時イベント(kind 20000-29999)はDBに保存されず配送のみ。通常イベントの保存期間は `event.retention.maxDays: -1` で無期限。

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | [2500](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L219) (合計) | `limits.client.subscription.maxFilterValues` |
| REQあたり最大フィルター数 | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L218) | `limits.client.subscription.maxFilters` |
| 最大authors数 | 2,500 (フィルター値制限による) | - |
| 最小プレフィックス長 | [4](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L222) | `limits.client.subscription.minPrefixLength` |
| サブスクリプションID長 | [256](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L220) | `limits.client.subscription.maxSubscriptionIdLength` |

**備考**: `maxFilterValues` は authors, ids, kinds, #tags など全フィルター値の合計上限

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| ネットワークペイロード | [524,288バイト (512 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L77) | `network.maxPayloadSize` |
| イベントコンテンツ | [102,400バイト (100 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L147) | `limits.event.content[].maxLength` |

**備考**: コンテンツサイズはkind範囲ごとに設定可能 (0-10, 40-49, 11-39, 50-max)

## サポートNIP

1, 2, 3, 4, 9, 11, 12, 14, 15, 16, 17, 20, 22, 25, 28, 33, 40, 44, 45, 65

## 追加機能

- 決済プロセッサ統合 (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL, NWC)
- Web of Trustフィルタリング
- NIP-05検証
- イベントkindごとの詳細なレート制限

---
[<< back](../README-ja.md)
