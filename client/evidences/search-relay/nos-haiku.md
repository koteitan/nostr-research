← [README](../README.md)

# Nos Haiku 検索リレー

## 結論
- **検索リレー**: search.nos.today (NIP-50)。query 指定時に searchRelays に kinds:[40,41] の search フィルタを送信

## ソースコード

**ファイル**: `src/lib/config.ts` (行 26)

```typescript
export const searchRelays = ['wss://search.nos.today/'];
```

## 説明
- 検索リレーとして `wss://search.nos.today/` を定数 `searchRelays` に定義している。
- `resource.ts:1274-1279` で query 指定時に `searchRelays` を relaySet へ追加し、`{ kinds: [40, 41], search: query }` を送信している。
- チャンネル(kind:40/41)の全文検索のみ対応している。

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/config.ts

---
← [README](../README.md)
