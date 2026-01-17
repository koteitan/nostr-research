[<< back](../README-ja.md) | [Japanese] | [English](README-en.md)

# Nostr Client Research
クライアントの実装の違いを研究するためのリポジトリです。
npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk か [github issue](https://github.com/koteitan/nostr-research/issues) に「この実装の違いをレポートしてほしい」と連絡を頂けると調査して掲載します。クライアントの提案もOK。PR で追加してくれるのも歓迎です。

# [Bootstrap リレー](evidences/bootstrap-relay/)
nostr が購読するリレーの決め方は、kind:10002 を使うもの、outbox モデルなど、さまざまがあり、それらの情報の多くはリレーのメタデータに含まれています。そのためには一度どこかのリレーに接続してメタデータを取得する必要があります。各クライアントがどのように Boot strap リレーを決定しているかを調査しました。

*最終更新: 2025-12-21*

| クライアント | Bootstrap リレー | 備考 |
|-------------|-----------------|------|
| [nostter](evidences/bootstrap-relay/nostter.md) | relay.nostr.band, nos.lol, relay.damus.io | 日本語設定時に日本リレー追加 (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net) |
| [Rabbit](evidences/bootstrap-relay/rabbit.md) | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | ブラウザ言語が日本語の場合に日本リレー追加 (relay-jp.nostr.wirednet.jp, holybea.com, r.kojira.io, yabu.me) |
| [Lumilumi](evidences/bootstrap-relay/lumilumi.md) | directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social | フォールバック: relay.nostr.band, nos.lol, nostr.bitcoiner.social |
| [Nos Haiku](evidences/bootstrap-relay/nos-haiku.md) | directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social | |
| [ぬるぬる](evidences/bootstrap-relay/nullnull.md) | yabu.me | [ハードコードリレー一覧](evidences/relay/nullnull.md#ハードコードされたリレーurl) |
| [野雨](evidences/bootstrap-relay/nosame.md) | relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, relay.barine.co | |
| [flowgazer](evidences/bootstrap-relay/flowgazer.md) | r.kojira.io | |
| [Yakihonne](evidences/bootstrap-relay/yakihonne.md) | nostr-01.yakihonne.com, nostr-02.yakihonne.com, relay.damus.io, relay.nostr.band, relay.nsec.app, monitorlizard.nostr1.com | |
| [iris](evidences/bootstrap-relay/iris.md) | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, nos.lol | |
| [Primal](evidences/bootstrap-relay/primal.md) | キャッシュサーバー経由 (cache.primal.net) | ユーザーはキャッシュインスタンスを選択可能 |
| [Coracle](evidences/bootstrap-relay/coracle.md) | relay.damus.io, nos.lol (環境変数 VITE_DEFAULT_RELAYS) | |
| [noStrudel](evidences/bootstrap-relay/nostrudel.md) | relay.primal.net, relay.damus.io, nos.lol | |
| [Amethyst](evidences/bootstrap-relay/amethyst.md) | relay.nostr.band, relay.damus.io, relay.primal.net, nostr.mom, nos.lol, nostr.bitcoiner.social, nostr.oxtr.dev | bootstrapInbox (7リレー) |
| [Damus](evidences/bootstrap-relay/damus.md) | relay.damus.io, nostr.land, nostr.wine, nos.lol | デバイスロケールで地域リレー追加 (日本: wirednet.jp, yabu.me, kojira.io / タイ: siamstr.com / ドイツ: einundzwanzig.space) |
| [algia](evidences/bootstrap-relay/algia.md) | relay.nostr.band | |
| [kakoi](evidences/bootstrap-relay/kakoi.md) | yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band | |

# [リレー](evidences/relay/)
各クライアントがホームタイムラインを作るためのリレーをどこから取得しているかを調査しました。

*最終更新: 2025-12-21*

| クライアント | リレー取得方法 | 詳細 |
|-------------|---------------|------|
| [nostter](evidences/relay/nostter.md) | kind:10002 (NIP-65) |  |
| [Rabbit](evidences/relay/rabbit.md) | 設定→localStorage |
| [Lumilumi](evidences/relay/lumilumi.md) | kind:10002 (NIP-65) または設定から取得 | ローカル設定優先、なければ NIP-65、最後にbootstrap リレー |
| [Nos Haiku](evidences/relay/nos-haiku.md) | Outboxモデル (NIP-65) |  |
| [ぬるぬる](evidences/relay/nullnull.md) | 設定→localStorage | 単一リレー、[ハードコードリレー一覧](evidences/relay/nullnull.md#ハードコードされたリレーurl) |
| [野雨](evidences/relay/nosame.md) | 設定→localStorage | bootstrap リレーにフォールバック |
| [flowgazer](evidences/relay/flowgazer.md) | 設定→localStorage |単一リレー |
| [Yakihonne](evidences/relay/yakihonne.md) | Outboxモデル (NIP-65) | outbox 無効時は bootstrap リレーにフォールバック |
| [iris](evidences/relay/iris.md) | Outboxモデル (NIP-65) |  |
| [Primal](evidences/relay/primal.md) | 読み取り: キャッシュサーバー経由、書き込み: kind:10002 (NIP-65) | サーバー選択可能、ユーザーのリレー設定は kind:10002 で管理 |
| [Coracle](evidences/relay/coracle.md) | kind:10002 (NIP-65) | Negentropy 対応、リレー品質スコアリング、複数カテゴリのリレー管理 |
| [noStrudel](evidences/relay/nostrudel.md) | Outboxモデル (NIP-65) | applesauce-relay 使用 |
| [Amethyst](evidences/relay/amethyst.md) | kind:10002 (NIP-65) | 用途別リレーセット (kind:10002、DM、検索、インデクサー、ブロック) |
| [Damus](evidences/relay/damus.md) | kind:10002 (NIP-65) | kind:10002 → kind:3 → UserDefaults → Bootstrap リレーの順で取得 |
| [algia](evidences/relay/algia.md) | kind:10002 (NIP-65) | 設定ファイル (~/.config/algia/config.json) で初期接続、kind:10002で上書き |
| [kakoi](evidences/relay/kakoi.md) | 設定から取得 | relays.json、起動時に自動接続 |

# [検索リレー](evidences/search-relay/)
各クライアントが検索に使用するリレーを調査しました。

*最終更新: 2025-12-21*

| クライアント | 検索リレー |
|-------------|-----------|
| [nostter](evidences/search-relay/nostter.md) | relay.nostr.band, search.nos.today |
| [Rabbit](evidences/search-relay/rabbit.md) | relay.nostr.band, search.nos.today |
| [Lumilumi](evidences/search-relay/lumilumi.md) | relay.nostr.band, search.nos.today |
| [Nos Haiku](evidences/search-relay/nos-haiku.md) | search.nos.today |
| [ぬるぬる](evidences/search-relay/nullnull.md) | search.nos.today |
| [野雨](evidences/search-relay/nosame.md) | なし |
| [flowgazer](evidences/search-relay/flowgazer.md) | なし |
| [Yakihonne](evidences/search-relay/yakihonne.md) | search.nos.today, relay.nostr.band, relay.ditto.pub, nostr.polyserv.xyz |
| [iris](evidences/search-relay/iris.md) | ローカル検索 (キャッシュから検索) |
| [Primal](evidences/search-relay/primal.md) | キャッシュサーバー経由 |
| [Coracle](evidences/search-relay/coracle.md) | relay.nostr.band, nostr.wine, search.nos.today |
| [noStrudel](evidences/search-relay/nostrudel.md) | relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine |
| [Amethyst](evidences/search-relay/amethyst.md) | relay.nostr.band, nostr.wine, relay.noswhere.com, search.nos.today |
| [Damus](evidences/search-relay/damus.md) | フルテキスト検索なし (ハッシュタグ・ユーザーのみ) |
| [algia](evidences/search-relay/algia.md) | relay.nostr.band (search フラグ) |
| [kakoi](evidences/search-relay/kakoi.md) | なし |

# [リアクション](evidences/reaction-for-events/)
イベントについているリアクションの収集方法, クローリング方法を調査しました。

*最終更新: 2025-12-21*

| クライアント | 収集方法 |
|-------------|---------|
| [nostter](evidences/reaction-for-events/nostter.md) | 詳細ページを開いた時に購読開始 |
| [Rabbit](evidences/reaction-for-events/rabbit.md) | バッチ取得 (2秒間隔、最大150件) |
| [Lumilumi](evidences/reaction-for-events/lumilumi.md) | 投稿ごとに購読 |
| [Nos Haiku](evidences/reaction-for-events/nos-haiku.md) | バッチ取得 (1秒バッファ, limit: 500) + リアルタイム購読 |
| [ぬるぬる](evidences/reaction-for-events/nullnull.md) | バッチ取得 (limit: 500) |
| [野雨](evidences/reaction-for-events/nosame.md) | リアクション取得なし（送信のみ） |
| [flowgazer](evidences/reaction-for-events/flowgazer.md) | 自分の投稿へのリアクションを取得 |
| [Yakihonne](evidences/reaction-for-events/yakihonne.md) | バッチ取得 + Dexie キャッシュ、WoT スコアでフィルタリング |
| [iris](evidences/reaction-for-events/iris.md) | イベント毎に購読 (closeOnEose) |
| [Primal](evidences/reaction-for-events/primal.md) | キャッシュサーバーから事前集計済みデータを取得 |
| [Coracle](evidences/reaction-for-events/coracle.md) | コンテキストイベントから kind:7 をフィルタリング |
| [noStrudel](evidences/reaction-for-events/nostrudel.md) | イベント毎に購読 |
| [Amethyst](evidences/reaction-for-events/amethyst.md) | バッチ取得 + リアルタイム購読、リレーごとにノートIDを集約 |
| [Damus](evidences/reaction-for-events/damus.md) | 通知フィルターで kind:7 取得、LikeCounter で重複排除 |
| [algia](evidences/reaction-for-events/algia.md) | リアクション取得なし（送信のみ） |
| [kakoi](evidences/reaction-for-events/kakoi.md) | リアクション収集なし |

# フレームワーク
各クライアントの実装に使用されているフレームワーク・ライブラリを調査しました。

*最終更新: 2025-12-21*

| クライアント | 言語 | UI | nostr アクセス | その他ライブラリ |
|-------------|------|-----|---------------|-----------------|
| nostter | TypeScript | Svelte + SvelteKit | rx-nostr, nostr-tools, @rust-nostr/nostr-sdk | rxjs, Melt UI, svelte-i18n |
| Rabbit | TypeScript | SolidJS | nostr-tools, nostr-wasm | @tanstack/solid-query, TailwindCSS, i18next, zod |
| Lumilumi | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, @konemono/nostr-login | TanStack Query, Leaflet, markdown-it, Melt UI |
| Nos Haiku | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, applesauce-core, nostr-login, nostr-zap | svelte-i18n, emoji-mart, mediabunny |
| ぬるぬる | JavaScript | Next.js 14 + React 18 | nostr-tools, nostr-login, nosskey-sdk, rx-nostr | Tailwind CSS, rxjs |
| 野雨 | JavaScript | Vanilla JS | なし (自前実装 + NIP-07) | なし |
| flowgazer | JavaScript | Vanilla JS (モジュラー) | nostr-tools | marked.js |
| Yakihonne | JavaScript | React + Next.js | NDK, nostr-tools | Redux Toolkit, Dexie, i18next, react-markdown |
| iris | TypeScript | React + Tauri | nostr-tools, nostr-social-graph, nostr-wasm | Zustand, Dexie, DaisyUI, Leaflet |
| Primal | TypeScript | SolidJS | nostr-tools | Tiptap, Milkdown, HLS.js, @cashu/cashu-ts |
| Coracle | TypeScript | Svelte | @welshman/*, nostr-tools | TailwindCSS, Capacitor, Bitcoin Connect, Fuse.js |
| noStrudel | TypeScript | React 19 + Chakra UI | nostr-tools, applesauce-* | React Router, CodeMirror, Chart.js, Leaflet |
| Amethyst | Kotlin | Jetpack Compose | quartz (自社ライブラリ) | OkHttp, Coil, Media3, secp256k1-kmp |
| Damus | Swift + C | SwiftUI | nostrdb (自社ライブラリ) | Flatbuffers, Lightning Network統合 |
| algia | Go | CLI | go-nostr | なし |
| kakoi | C# | Windows Forms (.NET 8.0) | NNostr.Client | SSTPLib, NTextCat, SkiaSharp, Gemini AI |

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
- NIPs
  - NIP-25 (Reactions): https://github.com/nostr-protocol/nips/blob/master/25.md
  - NIP-65 (Relay List Metadata): https://github.com/nostr-protocol/nips/blob/master/65.md
