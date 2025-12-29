← [README](../README.md)

# Yakihonne 検索機能で使用されているリレー調査結果

## 調査対象
- リポジトリ: https://github.com/YakiHonne/web-app
- 調査日: 2025-12-21

## 検索用リレー

Yakihonneの検索機能では、以下の4つのリレーが使用されています。

### 検索専用リレーリスト

ソースコード: `src/Content/Relays.js`

```javascript
const searchRelays = [
  "wss://search.nos.today",
  "wss://relay.nostr.band",
  "wss://relay.ditto.pub",
  "wss://nostr.polyserv.xyz",
];
```

## 検索機能の実装詳細

### 1. リレー設定の管理

- **設定ファイル**: `src/(PagesComponents)/Settings/SearchRelays.js`
- **デフォルトリレー**: 上記4つのリレーが推奨リレーとして設定されている
- **永続化**: ユーザーの検索リレー設定は kind 10007 イベントとして Nostr ネットワークに保存される

### 2. 検索実装

ソースコード: `src/(PagesComponents)/Search.js`

検索機能は以下のように実装されています:

```javascript
const userSearchRelays = useSelector((state) => state.userSearchRelays);

const searchForContent = async () => {
  let tag = searchKeyword.replaceAll("#", "");
  let tags = [
    tag,
    `${String(tag).charAt(0).toUpperCase() + String(tag).slice(1)}`,
    tag.toUpperCase(),
    tag.toLowerCase(),
    `#${tag}`,
    `#${tag.toUpperCase()}`,
    `#${tag.toLowerCase()}`,
    `#${String(tag).charAt(0).toUpperCase() + String(tag).slice(1)}`,
  ];
  let filter = {
    limit: 100,
    "#t": tags,
    until: lastTimestamp ? lastTimestamp - 1 : lastTimestamp,
  };
  if (selectedTab === "notes") filter.kinds = [1];
  if (selectedTab === "articles") filter.kinds = [30023];
  if (selectedTab === "videos") filter.kinds = [34235, 21, 22];

  let content = await getDataForSearch(
    [filter, { ...filter, search: searchKeyword, "#t": undefined }],
    100,
    200,
    userSearchRelays
  );
  // ...
};
```

### 3. 検索対象のイベント種別

- **Notes (投稿)**: kind 1
- **Articles (記事)**: kind 30023
- **Videos (動画)**: kind 34235, 21, 22

### 4. 検索方法

- **ハッシュタグ検索**: タグ(`#t`)を使用して、様々な大文字小文字のバリエーションで検索
- **キーワード検索**: `search`パラメータを使用した全文検索
- **ユーザー検索**: キャッシュAPI (`https://cache-v2.yakihonne.com`) を併用

## その他のリレー設定

参考までに、Yakihonneで使用されている他のリレーも記載します。

### プラットフォームリレー (relaysOnPlatform)

```javascript
const relaysOnPlatform = [
  "wss://nostr-01.yakihonne.com",
  "wss://nostr-02.yakihonne.com",
  "wss://relay.damus.io",
  "wss://relay.nostr.band",
  "wss://relay.nsec.app",
  "wss://monitorlizard.nostr1.com/",
];
```

### SSG用リレー (SSGRelays)

```javascript
const SSGRelays = [
  "wss://nostr-01.yakihonne.com",
  "wss://nostr-02.yakihonne.com",
  "wss://relay.damus.io",
  "wss://relay.nostr.band",
  "wss://nos.lol",
  "wss://relay.primal.net",
];
```

## 環境変数

`.env.example` から確認できる関連設定:

```
NEXT_PUBLIC_CACHE_URL=wss://cache2.primal.net/v1
NEXT_PUBLIC_API_CACHE_BASE_URL=https://cache-v2.yakihonne.com
```

## まとめ

Yakihonneの検索機能は、以下の4つの検索特化型リレーを使用しています:

1. **wss://search.nos.today** - nos.today の検索リレー
2. **wss://relay.nostr.band** - Nostr Band の検索対応リレー
3. **wss://relay.ditto.pub** - Ditto の検索リレー
4. **wss://nostr.polyserv.xyz** - polyserv の検索リレー

これらのリレーは、ハッシュタグ検索とキーワード検索の両方をサポートしており、Notes、Articles、Videosなど複数のイベント種別の検索に対応しています。ユーザーは設定画面から検索リレーをカスタマイズすることも可能です。

---
← [README](../README.md)
