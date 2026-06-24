← [README](../README.md)

# Primal リレー取得方法

## 結論
- **リレー取得方法**: ホームタイムラインはPrimalキャッシュサーバー経由。ユーザーのリレー一覧はキャッシュの get_user_relays (NIP-65 kind:10002 を Kind.UserRelays=10000139 として返却) から取得し relaySettings に反映。空の場合は get_default_relays を使用

## ソースコード

**ファイル**: `src/lib/profile.ts` (行 352-360)

```typescript
export const getRelays = async (pubkey: string | undefined, subid: string) => {
  if (!pubkey) return;

  sendMessage(JSON.stringify([
    "REQ",
    subid,
    {cache: ["get_user_relays", { pubkey }]},
  ]));
};
```

## 説明
- フィードは基本的にPrimalキャッシュサーバーが配信し、クライアントが直接リレーへ接続するわけではない。
- ユーザーのリレー一覧はキャッシュサーバーの `get_user_relays` で取得し、NIP-65 の kind:10002 を `Kind.UserRelays=10000139` として返却する。
- `handleUserRelaysEvent` (accountStore.ts 行2436) が `extractRelayConfigFromTags` でタグを `relaySettings` 化する。
- リレー一覧が空の場合は `getDefaultRelays` / `handleDefaultRelaysEvent` (accountStore.ts 行810, 2466) のフォールバックで `get_default_relays` を使用する。

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/profile.ts

---
← [README](../README.md)
