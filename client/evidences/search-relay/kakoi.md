← [README](../README.md)

# kakoi 検索リレー

## 結論
- **検索リレー**: なし。NIP-50 全文検索リレーは実装されていない。

## ソースコード

**ファイル**: `kakoi/NostrAccess.cs` (行 167-176)

```csharp
Kinds = [1, 6, 7, 16],
Since = DateTimeOffset.Now - _timeSpan,
```

## 説明
- 購読フィルターは Kinds と Since のみを指定しており、NIP-50 の `search` フィールドは含まれていない。
- NIP-50 (search) フィルターは全ソースに存在しない。
- コード中の "Search" は `FormAI.cs` の `UseGoogleSearch`（Gemini の Google 検索）のみで、Nostr 検索とは無関係。

## 参考
- https://github.com/betonetojp/kakoi/blob/master/kakoi/NostrAccess.cs

---
← [README](../README.md)
