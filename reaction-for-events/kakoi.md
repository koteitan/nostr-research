# kakoi リアクション取得方法

## 結論
- **リアクション取得方法**: リアクション収集なし

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

- `Kinds = [1, 6, 7, 16]` でテキストノート、リポスト、リアクションを全て購読
- `#e` タグでの投稿 ID 指定フィルター（ReferencedEventIds）は使用していない
- つまり、「全てのリアクションイベント」を購読し、受信後にコード内でフィルタリング

### 投稿ごとのリアクション収集について

- 個別投稿のリアクション取得フィルター（`#e` タグ）を使用していない
- リアクション数の集計機能なし
- 自分が送信したリアクションと、自分へのリアクションをイベントとしてグリッドに表示するのみ

## 参考
- https://github.com/betonetojp/kakoi
