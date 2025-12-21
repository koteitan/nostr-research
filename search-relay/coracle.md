# Coracle 検索機能のリレー調査結果

## 概要
Coracle (https://github.com/coracle-social/coracle) の検索機能で使用されているリレーについて調査しました。

## 検索用リレーURL

Coracleの検索機能では、以下の3つのリレーを使用しています：

1. **wss://relay.nostr.band**
2. **wss://nostr.wine**
3. **wss://search.nos.today**

## 設定ファイル

これらのリレーは `.env.template` ファイルで以下のように定義されています：

```
VITE_SEARCH_RELAYS=wss://relay.nostr.band,wss://nostr.wine,wss://search.nos.today
```

## 実装の仕組み

### 1. 環境変数の読み込み
`src/engine/state.ts` で環境変数を読み込んでいます：

```typescript
export const env = {
  // ...
  SEARCH_RELAYS: fromCsv(import.meta.env.VITE_SEARCH_RELAYS).map(normalizeRelayUrl) as string[],
  // ...
}
```

### 2. Routerの設定
初期化時にRouterコンテキストに検索リレーを設定：

```typescript
routerContext.getSearchRelays = always(env.SEARCH_RELAYS)
```

### 3. 検索リクエストの実行
`src/engine/requests.ts` の `createPeopleLoader` 関数で検索リレーに接続：

```typescript
myRequest({
  autoClose: true,
  skipCache: true,
  relays: Router.get().Search().getUrls(),
  filters: [{kinds: [0], search: term, limit: 100}],
  onEvent,
  onClose: async () => {
    await sleep(Math.min(1000, Date.now() - now))
    loading.set(false)
  },
})
```

### 4. NIP-50 検索の利用
検索クエリでは NIP-50（Search Capability）を使用しており、`search` パラメータを filter に含めています。

## 補足情報

### その他のリレー設定
Coracleでは検索リレー以外にも、目的別にリレーを分けています：

- **DVM_RELAYS**: 5つのリレー（分散型価値調整用）
  - wss://offchain.pub
  - wss://nos.lol
  - wss://relay.nostr.net
  - wss://relay.nostr.band
  - wss://relay.primal.net

- **DEFAULT_RELAYS**: 2つのリレー（デフォルト/フォールバック用）
  - wss://relay.damus.io
  - wss://nos.lol

- **INDEXER_RELAYS**: 3つのリレー（インデックス用）
  - wss://relay.nostr.band
  - wss://purplepag.es
  - wss://indexer.coracle.social

- **SIGNER_RELAYS**: 3つのリレー（署名操作用）
  - wss://relay.nsec.app
  - wss://bucket.coracle.social
  - wss://offchain.pub

### 初期化時のリレー
アプリケーション起動時には、これらすべてのリレーが初期リレーリストに含まれます：

```typescript
const initialRelays = [
  ...env.DEFAULT_RELAYS,
  ...env.DVM_RELAYS,
  ...env.INDEXER_RELAYS,
  ...env.SEARCH_RELAYS,
]
```

## 調査日
2025-12-21

## 参考リンク
- リポジトリ: https://github.com/coracle-social/coracle
- 設定ファイル: https://raw.githubusercontent.com/coracle-social/coracle/master/.env.template
- 状態管理: https://raw.githubusercontent.com/coracle-social/coracle/master/src/engine/state.ts
- リクエスト処理: https://raw.githubusercontent.com/coracle-social/coracle/master/src/engine/requests.ts
