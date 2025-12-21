# Nostr Client Research
クライアントの実装の違いを研究するためのリポジトリです。
npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk か [github issue](https://github.com/koteitan/nostr-client-research/issues) に「この実装の違いをレポートしてほしい」と連絡を頂けると調査して掲載します。クライアントの提案もOK。PR で追加してくれるのも歓迎です。

# Bootstrap リレー
nostr が購読するリレーの決め方は、kind:10002 を使うもの、outbox モデルなど、さまざまがあり、それらの情報の多くはリレーのメタデータに含まれています。そのためには一度どこかのリレーに接続してメタデータを取得する必要があります。各クライアントがどのように Boot strap リレーを決定しているかを調査しました。

*最終更新: 2025-12-21*

| クライアント | Bootstrap リレー | 備考 |
|-------------|-----------------|------|
| nostter | relay.nostr.band, nos.lol, relay.damus.io | 日本語設定時に日本リレー追加 (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net) |
| Rabbit | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | ブラウザ言語が日本語の場合に日本リレー追加 (relay-jp.nostr.wirednet.jp, holybea.com, r.kojira.io, yabu.me) |
| Lumilumi | directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social | フォールバック: relay.nostr.band, nos.lol, nostr.bitcoiner.social |
| Nos Haiku | directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social | |
| ぬるぬる | yabu.me | |
| 野雨 | relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, relay.barine.co | |
| flowgazer | r.kojira.io | |
| Yakihonne | nostr-01.yakihonne.com, nostr-02.yakihonne.com, relay.damus.io, relay.nostr.band, relay.nsec.app, monitorlizard.nostr1.com | |
| iris | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, nos.lol | |
| Primal | キャッシュサーバー経由 (cache.primal.net) | ユーザーはキャッシュインスタンスを選択可能 |
| Coracle | relay.damus.io, nos.lol (環境変数 VITE_DEFAULT_RELAYS) | |
| noStrudel | relay.primal.net, relay.damus.io, nos.lol | |
| Amethyst | relay.nostr.band, relay.damus.io, relay.primal.net, nostr.mom, nos.lol, nostr.bitcoiner.social, nostr.oxtr.dev | bootstrapInbox (7リレー) |
| Damus | relay.damus.io, nostr.land, nostr.wine, nos.lol | デバイスロケールで地域リレー追加 (日本: wirednet.jp, yabu.me, kojira.io / タイ: siamstr.com / ドイツ: einundzwanzig.space) |
| algia | relay.nostr.band | |
| kakoi | yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band | |

# リレー
各クライアントがホームタイムラインを作るためのリレーをどこから取得しているかを調査しました。

*最終更新: 2025-12-21*

| クライアント | リレー取得方法 | 詳細 |
|-------------|---------------|------|
| nostter   | kind:10002 (NIP-65) |  |
| Rabbit    | 設定→localStorage |
| Lumilumi  | kind:10002 (NIP-65) または設定から取得 | ローカル設定優先、なければ NIP-65、最後にbootstrap リレー |
| Nos Haiku | Outboxモデル (NIP-65) |  |
| ぬるぬる  | 設定→localStorage | 単一リレー |
| 野雨      | 設定→localStorage | bootstrap リレーにフォールバック |
| flowgazer | 設定→localStorage |単一リレー |
| Yakihonne | Outboxモデル (NIP-65) | outbox 無効時は bootstrap リレーにフォールバック |
| iris | Outboxモデル (NIP-65) |  |
| Primal | 読み取り: キャッシュサーバー経由、書き込み: kind:10002 (NIP-65) | サーバー選択可能、ユーザーのリレー設定は kind:10002 で管理 |
| Coracle | kind:10002 (NIP-65) | Negentropy 対応、リレー品質スコアリング、複数カテゴリのリレー管理 |
| noStrudel | Outboxモデル (NIP-65) | applesauce-relay 使用 |
| Amethyst | kind:10002 (NIP-65) | 用途別リレーセット (kind:10002、DM、検索、インデクサー、ブロック) |
| Damus | kind:10002 (NIP-65) | kind:10002 → kind:3 → UserDefaults → Bootstrap リレーの順で取得 |
| algia | kind:10002 (NIP-65) | 設定ファイル (~/.config/algia/config.json) で初期接続、kind:10002で上書き |
| kakoi | 設定から取得 | relays.json、起動時に自動接続 |

# 検索
各クライアントが検索に使用するリレーを調査しました。

*最終更新: 2025-12-21*

| クライアント | 検索リレー |
|-------------|-----------|
| nostter | relay.nostr.band, search.nos.today |
| Rabbit | relay.nostr.band, search.nos.today |
| Lumilumi | directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social |
| Nos Haiku | search.nos.today |
| ぬるぬる | search.nos.today |
| 野雨 | なし |
| flowgazer | なし |
| Yakihonne | search.nos.today, relay.nostr.band, relay.ditto.pub, nostr.polyserv.xyz |
| iris | ローカル検索 (キャッシュから検索) |
| Primal | キャッシュサーバー経由 |
| Coracle | relay.nostr.band, nostr.wine, search.nos.today |
| noStrudel | relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine |
| Amethyst | relay.nostr.band, nostr.wine, relay.noswhere.com, search.nos.today |
| Damus | フルテキスト検索なし (ハッシュタグ・ユーザーのみ) |
| algia | relay.nostr.band (search フラグ) |
| kakoi | なし |

# リアクションの取得方法
イベントについているリアクションの収集方法, クローリング方法を調査しました。

*最終更新: 2025-12-21*

| クライアント | 収集方法 |
|-------------|---------|
| nostter | 詳細ページを開いた時に購読開始 |
| Rabbit | バッチ取得 (2秒間隔、最大150件) |
| Lumilumi | 投稿ごとに購読 |
| Nos Haiku | リアルタイム購読 |
| ぬるぬる | バッチ取得 (limit: 500) |
| 野雨 | リアクション取得なし（送信のみ） |
| flowgazer | 自分の投稿へのリアクションを取得 |
| Yakihonne | バッチ取得 + Dexie キャッシュ、WoT スコアでフィルタリング |
| iris | イベント毎に購読 (closeOnEose) |
| Primal | キャッシュサーバーから事前集計済みデータを取得 |
| Coracle | コンテキストイベントから kind:7 をフィルタリング |
| noStrudel | イベント毎に購読 |
| Amethyst | バッチ取得 + リアルタイム購読、リレーごとにノートIDを集約 |
| Damus | 通知フィルターで kind:7 取得、LikeCounter で重複排除 |
| algia | リアクション取得なし（送信のみ） |
| kakoi | リアクション収集なし |

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
