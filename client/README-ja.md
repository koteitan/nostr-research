[<< back](../README-ja.md) | [Japanese] | [English](README-en.md)

# Nostr Client Research
クライアントの実装の違いを研究するためのリポジトリです。
npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk か [github issue](https://github.com/koteitan/nostr-research/issues) に「この実装の違いをレポートしてほしい」と連絡を頂けると調査して掲載します。クライアントの提案もOK。PR で追加してくれるのも歓迎です。

## 目次
- [Bootstrap リレー](#bootstrap-リレー)
- [リレー](#リレー)
- [検索リレー](#検索リレー)
- [リアクション](#リアクション)
- [画像アップロード](#画像アップロード)
- [フレームワーク](#フレームワーク)
- [調査済みクライアント一覧](#調査済みクライアント一覧)
- [参考文献](#参考文献)

# [Bootstrap リレー](evidences/bootstrap-relay/)
nostr が購読するリレーの決め方は、kind:10002 を使うもの、outbox モデルなど、さまざまがあり、それらの情報の多くはリレーのメタデータに含まれています。そのためには一度どこかのリレーに接続してメタデータを取得する必要があります。各クライアントがどのように Boot strap リレーを決定しているかを調査しました。

*最終更新: 2026-06-25*

| クライアント | Bootstrap リレー | 備考 |
|-------------|-----------------|------|
| [nostter](evidences/bootstrap-relay/nostter.md) | nos.lol, relay.damus.io (環境変数 VITE_DEFAULT_RELAYS で上書き可) | 日本語ロケール時のみ web/src/routes/+layout.ts の addDefaultRelays(localizedRelays.ja) で日本リレー4件を追加する。 |
| [Rabbit](evidences/bootstrap-relay/rabbit.md) | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | ブラウザ言語が 'ja' の場合に日本語リレー追加 (relay-jp.nostr.wirednet.jp, r.kojira.io, yabu.me)。判定は src/core/useConfig.ts:129-135 の initialRelays()。 |
| [Lumilumi](evidences/bootstrap-relay/lumilumi.md) | directory.yabu.me, purplepag.es, indexer.coracle.social, nostr.wine | relaySearchRelays は kind:0/3/10002 取得用ブートストラップ、defaultRelays は kind:10002 未検出時のフォールバック (nos.lol, nostr.bitcoiner.social, nostr-pub.wellorder.net, relay.snort.social)。 |
| [Nos Haiku](evidences/bootstrap-relay/nos-haiku.md) | directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social | indexerRelays で kind:10002 を取得。kind:10002 未取得時の汎用イベント取得には defaultRelays (nrelay.c-stellar.net, nostream.ocha.one, nostr.compile-error.net) を使用。 |
| [ぬるぬる](evidences/bootstrap-relay/nullnull.md) | yabu.me | デフォルトは wss://yabu.me。環境変数 NEXT_PUBLIC_DEFAULT_RELAY で上書き可、未設定時は localStorage の defaultRelay キーで変更。単一リレーアーキテクチャ。 |
| [野雨](evidences/bootstrap-relay/nosame.md) | relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nostr.compile-error.net | ロケール分岐はなく常に日本リレー固定。 |
| [flowgazer](evidences/bootstrap-relay/flowgazer.md) | r.kojira.io (localStorage の relayUrl で上書き可) | 起動時に1リレーへ接続する単一リレー構成。relay-manager.js は WebSocket 1本のみ管理。NIP-65/複数リレー非対応。lite/ 版は別途 nos.lol を既定にしている。 |
| [Yakihonne](evidences/bootstrap-relay/yakihonne.md) | relaysOnPlatform 6リレー (NDK explicitRelayUrls, enableOutboxModel: true) | src/Helpers/NDKInstance.js (行6-14) で初期化。yakihonne 自身のリレー (nostr-01/02) を含む。 |
| [iris](evidences/bootstrap-relay/iris.md) | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, relay.primal.net | 環境変数 VITE_USE_TEST_RELAY / VITE_USE_LOCAL_RELAY でテスト用・ローカルに切替可能。 |
| [Primal](evidences/bootstrap-relay/primal.md) | キャッシュサーバー経由 (wss://cache2.primal.net/v1) | 優先リレー既定値は wss://relay.primal.net、NIP-46 では wss://nrs.primal.net。優先リレー取得は src/lib/relays.ts getPreConfiguredRelays()。 |
| [Coracle](evidences/bootstrap-relay/coracle.md) | DEFAULT_RELAYS: relay.damus.io, nos.lol / INDEXER_RELAYS: relay.damus.io, purplepag.es, indexer.coracle.social | .env.template で定義。初期化時に DEFAULT/DVM/INDEXER/SEARCH 全リレーへ接続。 |
| [noStrudel](evidences/bootstrap-relay/nostrudel.md) | DEFAULT_FALLBACK_RELAYS: relay.primal.net, relay.damus.io / lookup: purplepag.es, index.hzrd149.com, indexer.coracle.social | RECOMMENDED_FALLBACK_RELAYS は +nos.lol。lookup (プロフィール・outbox 取得) リレーは purplepag.es, index.hzrd149.com, indexer.coracle.social。 |
| [Amethyst](evidences/bootstrap-relay/amethyst.md) | bootstrapInbox 7リレー (nos.lol, nostr.mom, relay.primal.net, relay.damus.io, nostr.bitcoiner.social, nostr.oxtr.dev, directory.yabu.me) | Constants.kt (commons/defaults) で定義。消費は Nip65RelayListState.normalizeNIP65ReadRelayListWithBackup() (63行)。 |
| [Damus](evidences/bootstrap-relay/damus.md) | relay.damus.io, nostr.land, nostr.wine, nos.lol | グローバル4リレー + 地域別 (日本: relay-jp.nostr.wirednet.jp/yabu.me/r.kojira.io、タイ、ドイツ)。UserDefaults にカスタムリレー保存可能 (save_bootstrap_relays / load_bootstrap_relays)。 |
| [algia](evidences/bootstrap-relay/algia.md) | relay.nostr.band | ブートストラップは設定ファイル(~/.config/algia/config.json)の relays。空の場合のみ relay.nostr.band を1件だけ Read/Write/Search 全フラグ有効で追加。ロケール別の追加リレーは無し。 |
| [kakoi](evidences/bootstrap-relay/kakoi.md) | yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band | LoadRelays() 内。relays.json が存在しない場合に返す日本語圏中心の固定リスト。ロケール分岐や環境変数による追加は無し。 |
| [Nostrism](evidences/bootstrap-relay/nostrism.md) | 日本語: relay-jp.shino3.net (本アプリ運営), yabu.me, relay.damus.io, nos.lol / その他言語: relay.damus.io, nos.lol | 初回起動時のみ端末言語で切替えてリレー表へシード（既存ユーザーの設定は上書きしない）。別途 INDEXER_RELAYS (purplepag.es, relay.nostr.band, relay.damus.io, nos.lol, relay.primal.net) で kind:0/10002 を取得。 |

# [リレー](evidences/relay/)
各クライアントがホームタイムラインを作るためのリレーをどこから取得しているかを調査しました。

*最終更新: 2026-06-25*

| クライアント | リレー取得方法 | 詳細 |
|-------------|---------------|------|
| [nostter](evidences/relay/nostter.md) | kind:10002 (NIP-65) | kind:10002 を優先取得し rxNostr.setDefaultRelays に適用。なければ kind:3 (Contacts) の content から取得。まず localStorage キャッシュ、無ければ metadataRelays から購読。 |
| [Rabbit](evidences/relay/rabbit.md) | 設定→localStorage | RabbitConfig.relayUrls から取得。初期値は relaysGlobal (+日本語なら relaysInJP)。ユーザーが addRelay/removeRelay で編集する。kind:10002 / outbox は不使用。 |
| [Lumilumi](evidences/relay/lumilumi.md) | kind:10002 (NIP-65) または設定から取得 | ローカル設定優先 (lumiSetting.useRelaySet !== "0")、"0" なら relaySearchRelays 経由で kind:10002 をフェッチして適用、見つからなければ defaultRelays。kind:10002 が無い旧形式は kind:3 content からも解釈。 |
| [Nos Haiku](evidences/relay/nos-haiku.md) | Outboxモデル (NIP-65) | ユーザ自身の kind:10002 を setDefaultRelays に設定し、フォロイーの write リレーを getReadRelaysWithOutboxModel / getOutboxes で抽出。kind:10002 が無い場合のみ defaultRelays にフォールバック。 |
| [ぬるぬる](evidences/relay/nullnull.md) | 設定→localStorage | 単一リレー、NIP-65 不使用。ホームタイムラインは getReadRelays()=[getDefaultRelay()]。fetchEvents 内では FALLBACK_RELAYS (relay-jp.nostr.wirednet.jp, r.kojira.io, relay.damus.io) を追加。 |
| [野雨](evidences/relay/nosame.md) | 設定→localStorage | NIP-65 不使用。localStorage の relays キーから取得し、未保存時は DEFAULT_RELAYS にフォールバック。全リレーへ同一フィルタで購読。 |
| [flowgazer](evidences/relay/flowgazer.md) | 設定→localStorage | 単一リレー、NIP-65/kind:10002 未使用。接続中の単一リレーから取得 (buildStreamPhaseFilters)。ユーザーが UI で変更すると localStorage.relayUrl に保存して再接続。 |
| [Yakihonne](evidences/relay/yakihonne.md) | Outboxモデル (NIP-65) | AppInit.js 行242-258 で read/write リレーを抽出し setUserRelays、空なら relaysOnPlatform にフォールバック。useOutboxRelays.js でフォロー先の kind:10002 から outboxRelays を構築。 |
| [iris](evidences/relay/iris.md) | Outboxモデル (NIP-65) | NDK を enableOutboxModel で初期化し、OutboxTracker が各ユーザーの kind:10002 を取得。初期接続先はユーザー設定リレー（無ければ DEFAULT_RELAYS）。NDK は src/lib/ndk にベンダリングされている。 |
| [Primal](evidences/relay/primal.md) | キャッシュサーバー経由 | ユーザーのリレー一覧は get_user_relays (kind:10002 を Kind.UserRelays=10000139 として返却) から取得し relaySettings に反映。空の場合は get_default_relays。フィードは基本キャッシュサーバーが配信。 |
| [Coracle](evidences/relay/coracle.md) | Outboxモデル (kind:10002 NIP-65) | @welshman/router がユーザーの RELAYS(10002) からリレーを選択。INDEXER_RELAYS でそのリレーリストを取得し relay_limit (デフォルト3) で件数制限。Router.ForUser()/FromPubkeys() でアウトボックス選択。 |
| [noStrudel](evidences/relay/nostrudel.md) | Outboxモデル (NIP-65) | ホームタイムラインは対象ユーザーの outbox マップに対して購読。outbox が取得できない場合は localSettings.fallbackRelays。useOutboxTimelineLoader + outboxSubscriptionsService を使用。 |
| [Amethyst](evidences/relay/amethyst.md) | Outboxモデル (kind:10002 NIP-65) | 書き込み/読み込みリレーは AdvertisedRelayListEvent から取得し outboxFlow / inboxFlow で公開。未設定時は eventFinderRelays / bootstrapInbox にフォールバック。Indexer リレー (DefaultIndexerRelayList: purplepag.es, indexer.coracle.social, user.kindpag.es, directory.yabu.me, profiles.nostr1.com) で他ユーザーのリレーリスト取得。 |
| [Damus](evidences/relay/damus.md) | kind:10002 (NIP-65) | フォールバック順: メモリ内キャッシュ (lastSetRelayList) → kind:10002 → kind:3 → UserDefaults → Bootstrap リレー。listenAndHandleRelayUpdates() で kind:10002 をリアルタイム購読し、より新しいリストのみ反映。 |
| [algia](evidences/relay/algia.md) | kind:10002 (NIP-65) | 設定ファイルの relays で初期接続し、その read リレーから kind:10002 を取得して上書き。rm が非空のときのみ cfg.Relays を上書き。--relay 指定時(tempRelay)は取得しない。 |
| [kakoi](evidences/relay/kakoi.md) | 設定から取得 | relays.json (GetEnabledRelays が enabled=true のみ抽出, 行215-227) から取得。kind:10002 や outbox モデルの探索は無く、ユーザーが手動編集する固定リスト方式。 |
| [Nostrism](evidences/relay/nostrism.md) | Outboxモデル (kind:10002 NIP-65) | 自分の kind:10002 をリレー設定として使用（Settings で編集・公開、初回は言語別デフォルトをシード）。フォロー中カラムは自分の read リレーから kind:1/6/16 を購読。著者カラム（著者数≤3）は著者の kind:10002 の write リレーからも追加購読（outbox、上限16）。 |

# [検索リレー](evidences/search-relay/)
各クライアントが検索に使用するリレーを調査しました。

*最終更新: 2026-06-25*

| クライアント | search.nos.today | relay.nostr.band | nostr.wine | relay.noswhere.com | relay.ditto.pub | filter.nostr.wine | relay.noswhere.sh | cagliostr.compile-error.net | nostr.polyserv.xyz | antiprimal.net | 備考 |
|-------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|------|
| [nostter](evidences/search-relay/nostter.md) | ✅ | | ✅ | | | | | | | | 環境変数 `VITE_SEARCH_RELAYS`（カンマ区切り）で上書き可。 |
| [Rabbit](evidences/search-relay/rabbit.md) | ✅ | ✅ | | | | | | | | | `relaysForSearching` の固定2件。 |
| [Lumilumi](evidences/search-relay/lumilumi.md) | ✅ | | ✅ | | | | | ✅ | | | kind:10007 があればそちらを優先。`relay.nostr.band` はコメントアウト中。 |
| [Nos Haiku](evidences/search-relay/nos-haiku.md) | ✅ | | | | | | | | | | チャンネル (kind:40/41) の検索のみ。 |
| [ぬるぬる](evidences/search-relay/nullnull.md) | ✅ | | | | | | | | | | 環境変数 `NEXT_PUBLIC_SEARCH_RELAY` で上書き可。 |
| [野雨](evidences/search-relay/nosame.md) | | | | | | | | | | | なし（全文検索機能が未実装、NIP-50 非対応）。 |
| [flowgazer](evidences/search-relay/flowgazer.md) | | | | | | | | | | | なし（NIP-50 全文検索は未実装）。 |
| [Yakihonne](evidences/search-relay/yakihonne.md) | ✅ | | | | ✅ | | | | ✅ | | ユーザの kind:10007 リレーを追加。NIP-50 search + #t タグ検索。 |
| [iris](evidences/search-relay/iris.md) | | | | | | | | | | | 専用リレーなし（接続中リレーへ NIP-50 + #t を送信）。プロフィール検索はローカル Fuse.js。 |
| [Primal](evidences/search-relay/primal.md) | | | | | | | | | | | 専用リレーなし。キャッシュサーバー経由 (search / user_search / advanced_search)。 |
| [Coracle](evidences/search-relay/coracle.md) | ✅ | | ✅ | | | | | | | | `env.SEARCH_RELAYS` を Router.Search() 経由で使用。 |
| [noStrudel](evidences/search-relay/nostrudel.md) | ✅ | ✅ | | ✅ | | ✅ | | | | | ユーザーの検索リレーリスト (kind:10007 系) があればそちらを優先。 |
| [Amethyst](evidences/search-relay/amethyst.md) | ✅ | | ✅ | ✅ | ✅ | | | | | ✅ | kind:10007 (SearchRelayListEvent) 未設定時のフォールバック。 |
| [Damus](evidences/search-relay/damus.md) | | | | | | | | | | | なし（ローカル nostrdb の全文検索 ndb_text_search）。 |
| [algia](evidences/search-relay/algia.md) | | ✅ | | | | | | | | | 設定ファイルで Search:true かつ Read:true のリレー。既定は relay.nostr.band。 |
| [kakoi](evidences/search-relay/kakoi.md) | | | | | | | | | | | なし（NIP-50 全文検索は未実装）。 |
| [Nostrism](evidences/search-relay/nostrism.md) | ✅ | ✅ | | | | | ✅ | | | | 接続中リレーではなく専用リレー群へ問い合わせ（未対応リレーでも動く）。 |

# [リアクション](evidences/reaction-for-events/)
イベントについているリアクションの収集方法, クローリング方法を調査しました。

*最終更新: 2026-06-25*

| クライアント | 収集方法 |
|-------------|---------|
| [nostter](evidences/reaction-for-events/nostter.md) | ノート詳細ページを開いた時に kinds:[1,6,7,9735] / #e でワンショット購読 |
| [Rabbit](evidences/reaction-for-events/rabbit.md) | バッチ取得 (2秒間隔、最大150件)。#e タグで kind:7 を設定リレーへ subscribeMany |
| [Lumilumi](evidences/reaction-for-events/lumilumi.md) | 表示中ノートを1秒デバウンスでバッチ購読 (kind:7/6/16, 9735) |
| [Nos Haiku](evidences/reaction-for-events/nos-haiku.md) | バッチ取得 (1秒バッファ, limit:500) + 前方リアルタイム購読 |
| [ぬるぬる](evidences/reaction-for-events/nullnull.md) | バッチ取得 (kind:7 を #e でまとめて limit:500) |
| [野雨](evidences/reaction-for-events/nosame.md) | リアクション取得なし（送信のみ） |
| [flowgazer](evidences/reaction-for-events/flowgazer.md) | ストリームから機会的にカウント + 自分宛(#p/#e)を明示取得 |
| [Yakihonne](evidences/reaction-for-events/yakihonne.md) | ノートごとに #e で kind:7/6/1/9735 を since 増分取得 + Dexie キャッシュ、WoT フィルタ |
| [iris](evidences/reaction-for-events/iris.md) | イベントごとに #e で kind:7 を購読 (closeOnEose) |
| [Primal](evidences/reaction-for-events/primal.md) | キャッシュサーバーが事前集計 (NoteStats kind:10000100 として返却) |
| [Coracle](evidences/reaction-for-events/coracle.md) | ノート表示時に各イベント単位で取得 (per-note、REACTION/ZAP_RESPONSE) |
| [noStrudel](evidences/reaction-for-events/nostrudel.md) | イベントごとに ReactionsModel でローカル event store から取得 |
| [Amethyst](evidences/reaction-for-events/amethyst.md) | イベント単位のバッチ取得 (リレー別 #e タグ、limit=1000) + リアルタイム購読 |
| [Damus](evidences/reaction-for-events/damus.md) | #p フィルタで自分宛通知をリアルタイム購読 (kind:7/6/9735/1) |
| [algia](evidences/reaction-for-events/algia.md) | リアクション取得なし（送信専用） |
| [kakoi](evidences/reaction-for-events/kakoi.md) | 専用収集なし (タイムライン購読 kind:[1,6,7,16] で一括受信のみ) |
| [Nostrism](evidences/reaction-for-events/nostrism.md) | 通常TLは集計なし（kind:1/6/16のみ購読、kind:7は購読しない）。パブリックチャット(kind:42)のみ表示中メッセージへ #e で kind:7 をバッチ購読(最大300 id, limit 500)して集約表示。通知は自分宛(#p) の kind:7/9735 等 |

# [画像アップロード](evidences/image-upload/)
各クライアントが投稿に画像/メディアを添付する際、どのメディアサーバー（アップロード先プロバイダ）へアップロードするかを調査しました。

*最終更新: 2026-07-03*

<table>
<thead>
<tr><th rowspan="2">クライアント</th><th colspan="3">プロトコル</th><th colspan="12">プロバイダ</th></tr>
<tr><th>NIP-96</th><th>Blossom</th><th>その他</th><th>nostr.build</th><th>nostrcheck.me</th><th>primal</th><th>yabu.me</th><th>sovbit</th><th>blossom.band</th><th>nostr.download</th><th>satellite</th><th>nostpic</th><th>void.cat</th><th>nostrmedia</th><th>yakihonne</th></tr>
</thead>
<tbody>
<tr><td><a href="evidences/image-upload/nostter.md">nostter</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/rabbit.md">Rabbit</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/lumilumi.md">Lumilumi</a></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nos-haiku.md">Nos Haiku</a></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nullnull.md">ぬるぬる</a></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/flowgazer.md">flowgazer</a> ※3</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/yakihonne.md">Yakihonne</a></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td></tr>
<tr><td><a href="evidences/image-upload/iris.md">iris</a> ※1</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/primal.md">Primal</a></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/coracle.md">Coracle</a></td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nostrudel.md">noStrudel</a></td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/amethyst.md">Amethyst</a> ※2</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td></tr>
<tr><td><a href="evidences/image-upload/damus.md">Damus</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/algia.md">algia</a> ※4</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nostrism.md">Nostrism</a> ※5</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td></tr>
</tbody>
</table>

※1 iris: iris サブスクライバは既定の先頭に `upload.iris.to`（Blossom）が加わる。
※2 Amethyst: 上記プロバイダに加え `24242.io` と `blossom.azzamo.media` も同梱（Blossom 計10サーバー）。
※3 flowgazer: 画像投稿は外部アプリ ehagaki (`lokuyow.github.io/ehagaki`) に委譲するため、同梱プロバイダを持たない。
※4 algia: CLI。同梱の既定は無く、`file-servers` 設定か `--server` 指定が必須。
※5 Nostrism: 既定で有効なのは nostrcheck.me と nostr.build の2件。nostpic.com / nostrmedia.com / files.sovbit.host は設定画面のワンタップ候補（プリセット）。

# フレームワーク
各クライアントの実装に使用されているフレームワーク・ライブラリを調査しました。

*最終更新: 2026-06-25*

| クライアント | 言語 | UI | nostr アクセス | その他ライブラリ |
|-------------|------|-----|---------------|-----------------|
| nostter | TypeScript | Svelte 5 + SvelteKit (adapter-cloudflare) | rx-nostr, rx-nostr-crypto, nostr-tools, @rust-nostr/nostr-sdk, nostr-typedef | rxjs, @melt-ui/svelte + melt, svelte-i18n, svelte-persisted-store, dexie, light-bolt11-decoder, twitter-text, virtua, nip07-awaiter, Vite |
| Rabbit | TypeScript | SolidJS (solid-js/web render) + @solidjs/router + @solidjs/meta + Tailwind CSS | nostr-tools 2.10.1, nostr-wasm (自前の usePool/useSubscription/useBatchedEvents ラッパー) | @tanstack/solid-query, i18next + i18next-browser-languagedetector, zod, lodash, idb-keyval, bech32, bolt11, emoji-mart, webln, qrcode, @thisbeyond/solid-dnd, Vite + Vitest |
| Lumilumi | TypeScript | Svelte 5 + SvelteKit (adapter-cloudflare / adapter-auto, Vite PWA, TailwindCSS) | rx-nostr, rx-nostr-crypto, nostr-tools, nostr-typedef, @konemono/nostr-login, nip07-awaiter | @tanstack/svelte-query, melt / @melt-ui (svelte/pp), @konemono/svelte5-i18n, @konemono/nostr-content-parser, markdown-it, light-bolt11-decoder, qrcode, leaflet/sveaflet, three/@google/model-viewer |
| Nos Haiku | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, applesauce-core (EventStore + NIP-65 helpers), @rx-nostr/crypto | @konemono/nostr-login, nostr-zap, svelte-i18n, svelte-persisted-store, emoji-mart, canvas-confetti, mediabunny |
| ぬるぬる | JavaScript | React 18 + Next.js 14 (Capacitor で Android 化, Tailwind CSS) | nostr-tools (SimplePool), rx-nostr, rxjs | nosskey-sdk (NIP-46 リモート署名), @capacitor/* (core/cli/android/app) |
| 野雨 | Vanilla JavaScript (ES Modules) | フレームワークなし（素の DOM 操作 / index.html + 手書き UIManager クラス） | 自前実装（WebSocket で直接リレー接続、独自 Bech32 実装で npub 変換、署名は NIP-07 window.nostr）。nostr-tools 等の外部ライブラリ不使用 | 外部 JS ライブラリ・package.json なし。Google Fonts (Sawarabi Mincho) のみ利用 |
| flowgazer | JavaScript (Vanilla JS, ES2015+ クラス) | フレームワークなし（素の DOM 操作、複数 HTML エントリ: index.html ほか） | nostr-tools 2.17.0（unpkg CDN の nostr.bundle.js）＋ 自前モジュール relay-manager.js / data-store.js / event-bus.js / view-state.js（生の WebSocket で REQ/EVENT を直接送受信） | 外部依存は nostr-tools のみ。motherfucking-nostr-client (jiftechnify) 派生。WTFPL ライセンス。lite/ と okkake/ の派生実装も同梱 |
| Yakihonne | JavaScript (JSX) | React 19 + Next.js 15 (App Router, next-pwa) | @nostr-dev-kit/ndk (NDK, outbox model), nostr-tools | @reduxjs/toolkit + react-redux, dexie + dexie-react-hooks + @nostr-dev-kit/ndk-cache-dexie (IndexedDB), i18next/next-i18next, @noble/hashes, @noble/secp256k1, react-virtuoso |
| iris | TypeScript | React 19 + Vite (独自スタックルーター、react-router不使用、Web/PWAアプリ、Tauri/Electron依存なし) | NDK (リポジトリ内ベンダリング src/lib/ndk、@nostr-dev-kit/ndk フォーク) + nostr-tools + nostr-wasm、リレー接続は Worker トランスポート(NDKWorkerTransport) | zustand, dexie (IndexedDB), fuse.js, nostr-social-graph, @cashu/cashu-ts, nostr-double-ratchet, localforage, lodash |
| Primal | TypeScript (5.2) | SolidJS 1.9 + @solidjs/router, Vite 4 (vite-plugin-solid), Kobalte UI, Sass | 独自Primalキャッシュサーバープロトコル (WebSocket + pako/zlib圧縮、cache: [...] REQ)。署名・補助に nostr-tools 2.23、NIP-46 リモート署名対応 | pako (圧縮), @cookbook/solid-intl (i18n), tiptap/milkdown (エディタ), @cashu/cashu-ts, light-bolt11-decoder, blossom-client-sdk, dompurify, fuse.js/@nozbe/microfuzz, hls.js |
| Coracle | TypeScript | Svelte 4 + Vite (PWA, Capacitor でiOS/Android) | @welshman/* (app, net, router, feeds, util, signer, store, content, editor, lib), nostr-tools | @noble/curves, fuse.js, idb, marked, hls.js, tippy.js, @getalby/sdk, @getalby/bitcoin-connect, tailwindcss |
| noStrudel | TypeScript | React 19 + Chakra UI (Vite) | applesauce-* (core/loaders/relay/react/common 等, next版), nostr-tools | rxjs, nostr-idb (IndexedDBキャッシュ), hash-sum, applesauce-sqlite/wallet |
| Amethyst | Kotlin (Android / Kotlin Multiplatform) | Jetpack Compose / Compose Multiplatform 1.11.1 (Material3) | 自作 quartz モジュール (Nostr KMP ライブラリ、プロトコル実装)、commons モジュール (共有リレークライアント) | kotlinx.coroutines + Flow, OkHttp 5.4.0, Jackson 2.22.0, secp256k1-kmp-jni 0.23.0, Koin/手動DI, navigation-compose, 自作 quic (純Kotlin QUIC/HTTP3/WebTransport) |
| Damus | Swift + C (iOS/macOS) | SwiftUI | 自前実装 (RelayPool / NostrNetworkManager) + nostrdb (Cベースのローカルイベントストア/全文検索) | nostr-sdk-swift (rust-nostr), negentropy-swift (リレー同期), secp256k1.swift, CryptoSwift, Kingfisher, swift-markdown-ui, GSPlayer, SwiftSoup, swift-collections, sentry-cocoa |
| algia | Go (go 1.24.1) | CLI (urfave/cli v2)。MCP サーバーモード(mark3labs/mcp-go)も搭載 | nbd-wtf/go-nostr v0.52.3 (sdk/pool 含む) | fatih/color, mattn/go-colorable/go-isatty, mdp/qrterminal (QR表示), mark3labs/mcp-go (MCP) |
| kakoi | C# (.NET 8 / net8.0-windows7.0) | Windows Forms (WinForms) + WebView2 | NNostr.Client (リポジトリ内に同梱・改変版) | Google_GenerativeAI (Gemini), Microsoft.Web.WebView2, NTextCat (言語判定), SkiaSharp, Svg.Skia, CredentialManagement, SSTPLib (伺か SSTP 連携) |
| Nostrism | Kotlin (Kotlin Multiplatform) | Compose Multiplatform 1.11 (Material3 + material-icons-extended, Android/iOS/iPad + Desktop 対応の Deck 型 UI) | 自前実装（`:nostr-core` プロトコル層 + `nostr/` リレープール(Ktor WebSocket)、`crypto/Nip01`・`Nip19` 自作、EC は secp256k1-kmp） | SQLDelight (SSOT DB, cache-first, マイグレーション), Ktor client (okhttp/darwin/cio), Coil3 (画像 + coil-gif), kotlinx.coroutines/Flow, kotlinx.serialization, kotlincrypto sha2, multiplatform-settings, androidx.media3 (動画), androidx.credentials (Nosskey/passkey), Android Keystore + NIP-46/NIP-55 署名 |

# 調査済みクライアント一覧
- web:
  - [nostter](https://nostter.app/)
  - [Rabbit](https://rabbit.syusui.net/)
  - [Lumilumi](https://lumilumi.app/)
  - [Nos Haiku](https://nos-haiku.vercel.app/)
  - [ぬるぬる](https://www.nullnull.app/)
  - [野雨](https://invertedtriangle358.github.io/Nosame/)
  - [flowgazer](https://ompomz.github.io/flowgazer/)
  - [Yakihonne](https://yakihonne.com/)
  - [iris](https://iris.to/)
  - [Primal](https://primal.net/)
  - [Coracle](https://coracle.social/)
  - [noStrudel](https://nostrudel.ninja/)
- mobile:
  - [Amethyst](https://play.google.com/store/apps/details?id=com.vitorpamplona.amethyst)
  - [Damus](https://damus.io/)
  - [Nostrism](https://github.com/ShinoharaTa/nostr-andloid-native-client) (Kotlin Multiplatform / Android・iOS・iPad + Desktop、Deck 型)
- cli:
  - [algia](https://github.com/mattn/algia)
- desktop:
  - [kakoi](https://nokakoi.com/)

# 参考文献
- クライアント
  - nostter: https://github.com/SnowCait/nostter
  - Rabbit: https://github.com/syusui-s/rabbit
  - Lumilumi: https://github.com/TsukemonoGit/lumilumi
  - Nos Haiku: https://github.com/nikolat/nos-haiku
  - ぬるぬる: https://github.com/tami1A84/null--nostr
  - 野雨: https://github.com/invertedtriangle358/Nosame
  - flowgazer: https://github.com/ompomz/flowgazer
  - Yakihonne: https://github.com/YakiHonne/web-app
  - iris: https://github.com/irislib/iris-client
  - Primal: https://github.com/PrimalHQ/primal-web-app
  - Coracle: https://github.com/coracle-social/coracle
  - noStrudel: https://github.com/hzrd149/nostrudel
  - Amethyst: https://github.com/vitorpamplona/amethyst
  - Damus: https://github.com/damus-io/damus
  - algia: https://github.com/mattn/algia
  - kakoi: https://github.com/betonetojp/kakoi
  - Nostrism: https://github.com/ShinoharaTa/nostr-andloid-native-client
- NIPs
  - NIP-25 (Reactions): https://github.com/nostr-protocol/nips/blob/master/25.md
  - NIP-65 (Relay List Metadata): https://github.com/nostr-protocol/nips/blob/master/65.md
