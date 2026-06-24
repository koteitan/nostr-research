← [README](../README.md)

# Coracle リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル（kind:10002 NIP-65）。`@welshman/router` がユーザーの RELAYS(10002) からリレーを選択。INDEXER_RELAYS でそのリレーリストを取得し、`relay_limit`（デフォルト3）で件数制限

## ソースコード

**ファイル**: `src/engine/state.ts` (行 829-832)

```typescript
// Configure router
routerContext.getDefaultRelays = always(env.DEFAULT_RELAYS)
routerContext.getIndexerRelays = always(env.INDEXER_RELAYS)
routerContext.getSearchRelays = always(env.SEARCH_RELAYS)
routerContext.getLimit = () => getSetting("relay_limit")
```

## 説明
- `@welshman/router` の Router を設定し、リレー選択ロジックを定義する。
- `getIndexerRelays` に `env.INDEXER_RELAYS` を指定し、ユーザーの RELAYS(kind:10002) リストの取得元とする。
- `getLimit` は設定値 `relay_limit`（デフォルト3、state.ts 244行）を返し、選択するリレー件数を制限する。
- ホームタイムラインは `@welshman/feeds` + `@welshman/router` の `Router.ForUser()` / `FromPubkeys()` でアウトボックス選択を行う。
- RELAYS=kind:10002、MESSAGING_RELAYS=kind:10050 を用いる。

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/engine/state.ts

---
← [README](../README.md)
