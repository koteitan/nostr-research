← [README](../README.md)

# kakoi リアクション取得方法

## 結論
- **リアクション取得方法**: 専用のリアクション収集なし。タイムライン購読 (Kinds=[1,6,7,16]) でリアクション (kind:7)・リポスト (kind:6,16) も一括受信するのみ。

## ソースコード

**ファイル**: `kakoi/NostrAccess.cs` (行 167-176)

```csharp
await _clients.CreateSubscription(
    _subscriptionId,
    [
            new NostrSubscriptionFilter
            {
                Kinds = [1, 6, 7, 16],
                Since = DateTimeOffset.Now - _timeSpan,
            }
    ]
);
```

## 説明
- `SubscribeAsync()` の単一フィルターで kind 1/6/7/16 をまとめて購読している。
- 投稿ごとの `#e` タグ (ReferencedEventIds) フィルターやリアクション数の集計は行わない。
- 受信したイベントをそのままグリッドに表示する方式。

## 参考
- https://github.com/betonetojp/kakoi/blob/master/kakoi/NostrAccess.cs

---
← [README](../README.md)
