← [README](../README.md)

# Rabbit リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage

## ソースコード

**ファイル**: `src/core/useConfig.ts` (行 150-153)

```typescript
const storage = createStorageWithSerializer(() => window.localStorage, serializer, deserializer);
const [config, setConfig] = createRoot(() =>
  createStoreWithStorage('RabbitConfig', InitialConfig(), storage),
);
```

## 説明

- リレー設定は `localStorage` の `RabbitConfig` キーに保存
- kind:10002 (NIP-65) は使用していない
- 初期リレーは `relaysGlobal` + 日本語なら `relaysInJP` を追加

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/core/useConfig.ts

---
← [README](../README.md)
