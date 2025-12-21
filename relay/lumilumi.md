# Lumilumi リレー取得方法

## 結論
- **リレー取得方法**: kind:10002 (NIP-65) または設定から取得
- **フォールバック**: ローカル設定 → NIP-65 → Bootstrap リレー

## ソースコード

**ファイル**: `src/lib/stores/relays.ts`

```typescript
export const relaySearchRelays = [
  // kind 0 (ユーザのプロフィール) と kind 10002 (利用中のリレーリスト) に特化
  "wss://directory.yabu.me",    // kind0, 3, 10002特化
  "wss://purplepag.es",         // https://purplepag.es/what
  "wss://relay.nostr.band",
  "wss://indexer.coracle.social",
];

export const defaultRelays = [
  "wss://relay.nostr.band",
  "wss://nos.lol",
  "wss://nostr.bitcoiner.social",
];
```

## 説明

- ユーザのローカル設定を最優先
- ローカル設定がない場合は kind:10002 (NIP-65) でリレーリストを取得
- NIP-65 も取得できない場合は `defaultRelays` をフォールバック
- `relaySearchRelays` で NIP-65 イベントを検索

## 処理フロー

```
アプリ起動
  ↓
ローカル設定をチェック
  ↓
ローカル設定あり → それを使用
ローカル設定なし → relaySearchRelays で kind:10002 を取得
  ↓
NIP-65 あり → ユーザのリレー設定を使用
NIP-65 なし → defaultRelays をフォールバック
```

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/stores/relays.ts
