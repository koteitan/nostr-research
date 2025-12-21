# Damus検索機能リレー調査結果

調査日: 2025-12-21

## 調査概要

Damus (https://github.com/damus-io/damus) の検索機能で使用されているリレーを調査しました。

## 現在の検索機能の状況

### 検索可能な項目

Damusで現在検索可能なのは以下のみです:
- ハッシュタグ
- ユーザー名
- 公開鍵(pubkey)
- naddr、nprofile、nevent等のNostr識別子

**注意**: フルテキスト検索は現時点では実装されていません。

## NIP-50検索リレーの実装状況

### NIP-50とは

NIP-50はNostrの検索機能を定義する仕様です。多くのNostrユースケースでは、タグやIDによる構造化クエリに加えて、一般的な検索機能が必要とされます。

### Damusの実装状況

**重要な発見**:

nsign.appに掲載された機能リクエストによると:
- Damusにkind:1イベントの検索機能を追加するPRリクエストが存在
- 検索タブと検索ボックスは既に存在するが、NIP-50を使用してリレーに検索文字列を転送し、結果をレンダリングする必要がある
- NIP-50を実装しているリレーは多くないが、**wss://relay.nostr.band** が良い選択肢として推奨されている
- ユーザーが検索リレーを選択できる方法（おそらく設定画面）を提供すべき

参照: https://www.nsign.app/post/1682949266/

### 推奨される検索リレー

NIP-50準拠の検索リレーとして以下が推奨されています:

1. **wss://relay.nostr.band** (最も推奨)
2. **wss://nostr.wine**
3. **wss://relay.noswhere.com**

## その他の関連情報

### Damusが公開しているリレー

Damus公式アカウントが使用しているリレー:
- wss://nos.lol
- wss://eden.nostr.land
- wss://pyramid.fiatjaf.com
- wss://relay.nostr.band

### 一般的なリレー推奨設定

Nostrエキスパートが推奨する設定:
- **Outboxリレー**: nos.lol、nostr.mom、relay.damus.io等の大規模な国際的リレーを1つ
- **検索リレー**: relay.nostr.band、nostr.wine、relay.noswhere.com (NIP-50準拠必須)

### ContentView.swiftでの検索実装

`damus/ContentView.swift`内で以下のように検索ビューが実装されています:

```swift
case .search:
    if #available(iOS 16.0, *) {
        SearchHomeView(damus_state: damus_state!,
                      model: SearchHomeModel(damus_state: damus_state!))
            .scrollDismissesKeyboard(.immediately)
```

### RelayFiltersメカニズム

検索機能が利用するリレーは以下のメカニズムで管理されている可能性があります:
- `RelayFilters`クラス
- `relay_filters`プロパティ (DamusStateに含まれる)
- `RelayModelCache` (リレー情報のキャッシュ)

## 結論

### 主要な発見

1. **現在のDamus検索は限定的**: フルテキスト検索ではなく、ハッシュタグとユーザー識別子のみ対応

2. **NIP-50実装は未完了**: NIP-50検索機能の追加リクエストが存在するが、完全な実装はまだ

3. **推奨検索リレー**: **wss://relay.nostr.band** が最も推奨される検索リレー

4. **将来の実装計画**: ユーザーが検索リレーを設定画面で選択できるようにする計画

### Amethystとの比較

Amethyst (Androidクライアント) では既にNIP-50検索リレーの設定が可能で、検索リレーが設定されていないと@によるユーザータグ付けや検索が機能しません。Damusも同様の機能を実装予定と思われます。

## 参考資料

- [Damus GitHub Repository](https://github.com/damus-io/damus)
- [Damus ContentView.swift](https://github.com/damus-io/damus/blob/master/damus/ContentView.swift)
- [Add NIP-50 search to Damus](https://www.nsign.app/post/1682949266/)
- [Relay Setup 101 by Vitor Pamplona](https://vitor.npub.pro/post/relay-setup/)
- [NIP-50 Specification](https://nips.nostr.com/50)
- [Damus Official Documentation](https://nostr.com/clients/damus)
