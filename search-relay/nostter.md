# nostter 検索機能のリレー調査結果

## 調査対象
- リポジトリ: https://github.com/SnowCait/nostter
- 調査日: 2025-12-21

## 検索機能で使用されているリレー

### 検索専用リレー
nostterの検索機能では、以下の2つの専用リレーを使用しています。

**定義場所**: `/web/src/lib/Constants.ts`

```typescript
export const searchRelays = ['wss://relay.nostr.band/', 'wss://search.nos.today/'];
```

### 検索リレーの使用方法

**実装場所**: `/web/src/lib/Search.ts`

Search クラスの `fetch` メソッド内で以下のロジックにより、検索クエリに `search` パラメータが含まれている場合は専用の検索リレーを使用し、それ以外の場合はユーザーの読み取りリレーを使用します。

```typescript
async fetch(filter: Filter): Promise<EventItem[]> {
    const events = await fetchEvents(
        [filter],
        filter.search !== undefined ? searchRelays : get(readRelays)
    );
    // ...
}
```

## 検索リレーの詳細

### 1. relay.nostr.band
- URL: `wss://relay.nostr.band/`
- 用途: 検索機能、およびデフォルトリレーの一つとしても使用
- 特徴: 汎用的な検索に対応

### 2. search.nos.today
- URL: `wss://search.nos.today/`
- 用途: 検索専用
- 特徴: 検索に特化したリレー

## その他のリレー設定

nostterでは検索リレー以外にも、以下のような用途別リレーを定義しています。

### デフォルトリレー
```typescript
export const defaultRelays = [
    { url: 'wss://relay.nostr.band/', read: true, write: true },
    { url: 'wss://nos.lol/', read: true, write: true },
    { url: 'wss://relay.damus.io/', read: true, write: true }
];
```

### メタデータリレー（プロフィール情報用）
```typescript
export const metadataRelays = [
    'wss://purplepag.es/',
    'wss://user.kindpag.es/',
    'wss://directory.yabu.me/'
];
```

### トレンドリレー
```typescript
export const trendRelays = ['wss://nostrbuzzs-relay.fly.dev/'];
```

### パブリックリレー（スパム対策済み）
```typescript
export const publicRelays = [
    'wss://nostr.wine/',
    'wss://relay.snort.social/',
    'wss://wot.utxo.one/',
    'wss://nostrelites.org/'
];
```

### 日本語ローカライズ用リレー
```typescript
export const localizedRelays = {
    ja: [
        { url: 'wss://relay-jp.nostr.wirednet.jp/', read: true, write: true },
        { url: 'wss://yabu.me/', read: true, write: true },
        { url: 'wss://r.kojira.io/', read: true, write: true },
        { url: 'wss://nrelay-jp.c-stellar.net/', read: true, write: true }
    ]
};
```

## まとめ

nostterの検索機能は、以下の2つの専用検索リレーを使用しています：

1. **wss://relay.nostr.band/** - 汎用検索リレー（デフォルトリレーとしても使用）
2. **wss://search.nos.today/** - 検索専用リレー

これらのリレーは、検索クエリに `search` パラメータが含まれている場合に自動的に選択され、効率的な全文検索を提供します。検索パラメータがない場合は、ユーザーが設定した通常の読み取りリレーが使用されます。
