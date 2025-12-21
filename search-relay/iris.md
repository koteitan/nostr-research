# iris 検索リレー

## 結論
- **検索リレー**: なし (専用検索リレーは使用しない)
- プロフィール検索: ローカル検索 (Fuse.js)
- コンテンツ検索: 接続中のリレーに search フィルタを送信

## ソースコード

### コンテンツ検索 (投稿検索)

**ファイル**: `src/shared/hooks/useFeedEvents.ts` (行 314-417)

```typescript
if (filters.search) {
  const searchTerms = filters.search.toLowerCase().split(/\s+/)
  const hashtags: string[] = []
  const regularWords: string[] = []

  searchTerms.forEach((term) => {
    if (term.startsWith("#") && term.length > 1) {
      hashtags.push(term.substring(1))
    } else if (term.length > 0) {
      regularWords.push(term)
    }
  })

  // For searches with regular words
  if (regularWords.length > 0) {
    // Also include search filter for relays that support full-text search
    filterArray.push({
      ...baseFilter,
      search: regularWords.join(" "),
    })
  }
}

const sub = ndk().subscribe(subscriptionFilters, relayUrls ? {relayUrls} : undefined)
```

### プロフィール検索 (ローカル)

**ファイル**: `src/workers/profile-search.ts`

- Fuse.jsを使用したメモリ内検索インデックス
- Dexieデータベース（IndexedDB）にキャッシュされたプロフィール情報を検索

### デフォルトリレー

**ファイル**: `src/shared/constants/relays.ts`

```typescript
const PRODUCTION_RELAYS = [
  "wss://temp.iris.to/",
  "wss://vault.iris.to/",
  "wss://relay.damus.io/",
  "wss://relay.snort.social/",
  "wss://nos.lol/",
]
```

## 説明

- 専用の検索リレーはハードコードされていない
- コンテンツ検索: 接続中のリレーに `search` フィルタを送信
- プロフィール検索: ローカルのFuse.js検索を使用
- ハッシュタグは `#t` タグフィルタに変換
- クライアント側でも検索結果をフィルタリング

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/hooks/useFeedEvents.ts
- https://github.com/irislib/iris-client/blob/main/src/workers/profile-search.ts
