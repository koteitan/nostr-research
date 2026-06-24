← [README](../README.md)

# Rabbit リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage (RabbitConfig)。NIP-65 (kind:10002) は不使用

## ソースコード

**ファイル**: `src/core/useConfig.ts` (行 169-172)

```typescript
const storage = createStorageWithSerializer(() => window.localStorage, serializer, deserializer);
const [config, setConfig] = createRoot(() =>
  createStoreWithStorage('RabbitConfig', InitialConfig(), storage),
);
```

## 説明
- ホームタイムラインのリレーは localStorage の `RabbitConfig.relayUrls` から取得する。
- 初期値は `relaysGlobal`（日本語設定なら `relaysInJP` を追加）。
- ユーザーが `addRelay` / `removeRelay` で編集できる。
- kind:10002 / outbox モデルは使わない。

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/core/useConfig.ts

---
← [README](../README.md)
