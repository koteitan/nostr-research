# Nostr Client Research
クライアントの実装の違いを研究するためのリポジトリです。
npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk にこの実装の違いをレポートしてほしいと連絡を頂けると調査して掲載します。PR で追加してくれるのも歓迎です。

# Boot strap リレー
nostr が購読するリレーの決め方は、kind:10002 を使うもの、outbox モデルなど、さまざまがあり、それらの情報の多くはリレーのメタデータに含まれています。そのためには一度どこかのリレーに接続してメタデータを取得する必要があります。各クライアントがどのように Boot strap リレーを決定しているかを調査しました。

| クライアント | Bootstrap リレー | 地域対応 |
|-------------|-----------------|---------|
| nostter | relay.nostr.band, nos.lol, relay.damus.io | 日本語設定時に日本リレー追加 (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net) |
| Rabbit | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | ブラウザ言語が日本語の場合に日本リレー追加 (relay-jp.nostr.wirednet.jp, holybea.com, r.kojira.io, yabu.me) |
| Lumilumi | relay.nostr.band, nos.lol, nostr.bitcoiner.social | 検索用リレー別途設定 (directory.yabu.me, purplepag.es) |
| Yakihonne | nostr-01.yakihonne.com, nostr-02.yakihonne.com, relay.damus.io, relay.nostr.band, relay.nsec.app, monitorlizard.nostr1.com | 自社リレーを優先 |
| iris | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, nos.lol | 自社リレー (iris.to) を優先 |
| Primal | キャッシュサーバー経由 (PRIMAL_CACHE_URL) | サーバーから get_default_relays で取得 |
| Coracle | relay.damus.io, nos.lol (環境変数 VITE_DEFAULT_RELAYS) | 複数カテゴリ (indexer, DVM, search, signer) |
| Amethyst | nos.lol, nostr.mom, nostr.bitcoiner.social | 用途別リレー (DM用: auth.nostr1.com, 0xchat.com / 検索用: nostr.band, nos.today / インデクサー: purplepag.es, coracle) |
| Damus | relay.damus.io, nostr.land, nostr.wine, nos.lol | デバイスロケールで地域リレー追加 (日本: wirednet.jp, yabu.me, kojira.io / タイ: siamstr.com / ドイツ: einundzwanzig.space) |

# リレー
各クライアントがホームタイムラインを作るためのリレーをどこから取得しているかを調査しました。

| クライアント | リレー取得方法 | 詳細 |
|-------------|---------------|------|
| nostter | NIP-65 (kind:10002) | rx-nostr 使用、リレーリストイベント受信時に updateRelays() で更新 |
| Rabbit | 設定から取得 | config().relayUrls を使用、全リレーに同時クエリ |
| Lumilumi | NIP-65 (kind:10002) + kind:3 フォールバック | 新しいイベントを優先、rx-nostr 使用 |
| Yakihonne | NDK + Outbox モデル | enableOutboxModel: true、addExplicitRelays() で動的追加 |
| iris | NDK + Outbox モデル | ユーザーリレー設定 + WebRTC P2P トランスポート対応 |
| Primal | キャッシュサーバー | 中央集権的なキャッシュサーバーアーキテクチャ、リレーはサーバー側で管理 |
| Coracle | @welshman ライブラリ | Negentropy 対応、リレー品質スコアリング、複数カテゴリのリレー管理 |
| Amethyst | NIP-65 (kind:10002) | 用途別リレーセット (NIP-65リスト、DM、検索、インデクサー、ブロック) |
| Damus | NIP-65 (kind:10002) | RelayList 構造体で read/write 設定管理、UserDefaults に pubkey 毎に保存 |

# リアクションの取得方法
イベントについているリアクションの収集方法, クローリング方法を調査しました。

| クライアント | 取得方法 | バッチ処理 | フィルター |
|-------------|---------|-----------|-----------|
| nostter | 通知ストリームの一部として取得 | なし (ストリーム購読) | kinds: [7] (notificationsFilterKinds に含む) |
| Rabbit | バッチ取得 (useBatchedEvents) | interval: 2000ms, batchSize: 150 | kinds: [7], #e タグでイベント指定 |
| Lumilumi | ページネーション付きバッチ取得 | limit: 500 | kinds: [1, 42, 6, 7, 9735, 1111], #e/#a/#q タグ |
| Yakihonne | バッチ取得 (useNoteStats) | 5種類のクエリを1リクエストに | kinds: [7, 6, 1, 1, 9735]、WoT スコアでフィルタリング、Dexie キャッシュ |
| iris | イベント毎に購読 | closeOnEose: true (NDK) | kinds: [7], #e タグ、著者 pubkey で重複排除 |
| Primal | キャッシュサーバーから事前集計 | サーバー側で集計済み | postStats オブジェクトで likes, reposts, replies, zaps を取得 |
| Coracle | 通知として取得 | conservative pulling (since フィルター使用)、pubkey 50件/300ms | reactionKinds, #p タグ |
| Amethyst | NIP-25 実装 (quartz ライブラリ) | - | ReactionEvent.kt で kind:7 処理 |
| Damus | nostrdb 経由 | - | NIP-25 対応、nostrdb でイベント保存・検索 |

# 調査済みクライアント一覧
- web:
  - [nostter](https://nostter.app/)
  - [Rabbit](https://rabbit.syusui.net/)
  - [Lumilumi](https://lumilumi.app/)
  - [Yakihonne](https://yakihonne.com/)
  - [iris](https://iris.to/)
  - [Primal](https://primal.net/)
  - [Coracle](https://coracle.social/)
- mobile:
  - [Amethyst](https://play.google.com/store/apps/details?id=com.vitorpamplona.amethyst)
  - [Damus](https://damus.io/)

# 参考文献
- クライアント
  - nostter: https://github.com/SnowCait/nostter
  - Rabbit: https://github.com/syusui-s/rabbit
  - Lumilumi: https://github.com/TsukemonoGit/lumilumi
  - Yakihonne: https://github.com/YakiHonne/web-app
  - iris: https://github.com/irislib/iris-client
  - Primal: https://github.com/PrimalHQ/primal-web-app
  - Coracle: https://github.com/coracle-social/coracle
  - Amethyst: https://github.com/vitorpamplona/amethyst
  - Damus: https://github.com/damus-io/damus
- NIPs
  - NIP-25 (Reactions): https://github.com/nostr-protocol/nips/blob/master/25.md
  - NIP-65 (Relay List Metadata): https://github.com/nostr-protocol/nips/blob/master/65.md
