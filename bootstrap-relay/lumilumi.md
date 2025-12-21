# Lumilumi Bootstrap Relay

## 結論
- **Bootstrap リレー (relaySearchRelays)**: directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social
- **フォールバック (defaultRelays)**: relay.nostr.band, nos.lol, nostr.bitcoiner.social

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

## 使い分け

### relaySearchRelays（Bootstrap）
- ユーザ情報の検索・取得に特化
- Kind 0（プロフィール）、Kind 3（コンタクト）、Kind 10002（リレーリスト）の取得
- `getRelayList()` でユーザのリレー設定を取得する時に使用
- アプリ初期化時に最初に接続するリレー

### defaultRelays（フォールバック）
- 一般的なイベント取得のフォールバック
- ユーザがリレー設定を持たない場合のデフォルト
- タイムライン、検索、イベント投稿に使用

## 処理フロー

```
アプリ起動
  ↓
relaySearchRelays でユーザの Kind 10002 を取得
  ↓
ユーザのリレー設定が見つかった → それを使用
見つからなかった → defaultRelays をフォールバックとして使用
```

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/stores/relays.ts
