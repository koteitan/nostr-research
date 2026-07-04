[<< back](../README.md) | [Japanese](README-ja.md) | [English]

# Nostr Client Research
A repository for researching implementation differences between Nostr clients.
Contact npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk or [github issue](https://github.com/koteitan/nostr-research/issues) to request research on specific implementation differences. Client suggestions and PRs are also welcome.

# [Bootstrap Relays](evidences/bootstrap-relay/)
There are various methods for determining which relays Nostr subscribes to, such as using kind:10002 or the outbox model. Much of this information is contained in relay metadata. To obtain this metadata, clients must first connect to some relay. We researched how each client determines its bootstrap relays.

*Last updated: 2026-06-25*

| Client | Bootstrap Relays | Notes |
|--------|-----------------|-------|
| [nostter](evidences/bootstrap-relay/nostter.md) | nos.lol, relay.damus.io (overridable via VITE_DEFAULT_RELAYS) | 4 Japanese relays added when locale is ja via web/src/routes/+layout.ts addDefaultRelays(localizedRelays.ja). |
| [Rabbit](evidences/bootstrap-relay/rabbit.md) | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | Japanese relays added when browser language is 'ja' (relay-jp.nostr.wirednet.jp, r.kojira.io, yabu.me). |
| [Lumilumi](evidences/bootstrap-relay/lumilumi.md) | directory.yabu.me, purplepag.es, indexer.coracle.social, nostr.wine | Bootstrap relays specialized for fetching kind:0/3/10002 to discover the user's relay list (NIP-65). Fallback when kind:10002 is missing: nos.lol, nostr.bitcoiner.social, nostr-pub.wellorder.net, relay.snort.social |
| [Nos Haiku](evidences/bootstrap-relay/nos-haiku.md) | directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social | indexerRelays fetch kind:10002; defaultRelays (nrelay.c-stellar.net, nostream.ocha.one, nostr.compile-error.net) used for generic fetch when kind:10002 is unavailable |
| [nullnull](evidences/bootstrap-relay/nullnull.md) | yabu.me | Single relay, overridable via NEXT_PUBLIC_DEFAULT_RELAY env var or the defaultRelay localStorage key |
| [Nosame](evidences/bootstrap-relay/nosame.md) | relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nostr.compile-error.net | Always fixed to Japanese relays (no locale branching). |
| [flowgazer](evidences/bootstrap-relay/flowgazer.md) | r.kojira.io | Single relay, overridable via the relayUrl localStorage key. The lite/ variant defaults to nos.lol. |
| [Yakihonne](evidences/bootstrap-relay/yakihonne.md) | relaysOnPlatform (6 relays) set as NDK explicitRelayUrls (enableOutboxModel: true) | Initialized in src/Helpers/NDKInstance.js. Includes Yakihonne's own relays (nostr-01/02). |
| [iris](evidences/bootstrap-relay/iris.md) | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, relay.primal.net | Switchable to test/local relays via VITE_USE_TEST_RELAY / VITE_USE_LOCAL_RELAY. |
| [Primal](evidences/bootstrap-relay/primal.md) | Via cache server (default wss://cache2.primal.net/v1) | Default priority relay is relay.primal.net; NIP-46 (remote signing) uses nrs.primal.net |
| [Coracle](evidences/bootstrap-relay/coracle.md) | DEFAULT_RELAYS=relay.damus.io, nos.lol / INDEXER_RELAYS=relay.damus.io, purplepag.es, indexer.coracle.social | Defined via .env. Connects to all DEFAULT/DVM/INDEXER/SEARCH relays on init. |
| [noStrudel](evidences/bootstrap-relay/nostrudel.md) | relay.primal.net, relay.damus.io (DEFAULT_FALLBACK_RELAYS); RECOMMENDED adds nos.lol | Profile/outbox lookup relays: purplepag.es, index.hzrd149.com, indexer.coracle.social. |
| [Amethyst](evidences/bootstrap-relay/amethyst.md) | nos.lol, nostr.mom, relay.primal.net, relay.damus.io, nostr.bitcoiner.social, nostr.oxtr.dev, directory.yabu.me | Constants.bootstrapInbox (7 relays) used as the fallback when no NIP-65 read relays exist. |
| [Damus](evidences/bootstrap-relay/damus.md) | relay.damus.io, nostr.land, nostr.wine, nos.lol | Regional relays added based on the user's locale (Japan: relay-jp.nostr.wirednet.jp/yabu.me/r.kojira.io, Thailand, Germany). Custom relays can be saved in UserDefaults. |
| [algia](evidences/bootstrap-relay/algia.md) | relay.nostr.band | When no relays are defined in the config file, relay.nostr.band is added once with Read/Write/Search flags all enabled. No locale-specific relays. |
| [kakoi](evidences/bootstrap-relay/kakoi.md) | yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band | Hardcoded list used when relays.json does not exist. No locale branching or env-var additions. |

# [Relays](evidences/relay/)
We researched how each client obtains relays for building the home timeline.

*Last updated: 2026-06-25*

| Client | Relay Source | Details |
|--------|-------------|---------|
| [nostter](evidences/relay/nostter.md) | kind:10002 (NIP-65) | Prefers kind:10002 and applies it to rxNostr.setDefaultRelays; falls back to kind:3 (Contacts) content. localStorage cache first, then subscribes via metadataRelays. |
| [Rabbit](evidences/relay/rabbit.md) | Settings -> localStorage | Home timeline relays read from RabbitConfig.relayUrls in localStorage. kind:10002 / outbox not used. |
| [Lumilumi](evidences/relay/lumilumi.md) | kind:10002 (NIP-65) or settings | Local settings priority, then NIP-65 (fetched via relaySearchRelays), finally defaultRelays. Old-style kind:3 content also interpreted. |
| [Nos Haiku](evidences/relay/nos-haiku.md) | Outbox model (NIP-65) | User's own kind:10002 set to setDefaultRelays; followees' write relays extracted via getReadRelaysWithOutboxModel. Falls back to defaultRelays only when kind:10002 is missing. |
| [nullnull](evidences/relay/nullnull.md) | Settings -> localStorage | Single relay, NIP-65 not used. fetchEvents adds FALLBACK_RELAYS (relay-jp.nostr.wirednet.jp, r.kojira.io, relay.damus.io). |
| [Nosame](evidences/relay/nosame.md) | Settings -> localStorage | From the relays localStorage key; falls back to DEFAULT_RELAYS when unset. NIP-65 / outbox not used. |
| [flowgazer](evidences/relay/flowgazer.md) | Settings -> localStorage | Single relay. NIP-65 / kind:10002 not used; saved to localStorage.relayUrl on change. |
| [Yakihonne](evidences/relay/yakihonne.md) | Outbox model (NIP-65) | Read/write relays extracted from the user's kind:10002; falls back to relaysOnPlatform when empty. Followee outbox relays built via useOutboxRelays.js. |
| [iris](evidences/relay/iris.md) | Outbox model (NIP-65) | NDK initialized with enableOutboxModel; OutboxTracker fetches each user's kind:10002. Initial connection uses user settings (or DEFAULT_RELAYS). NDK is vendored under src/lib/ndk. |
| [Primal](evidences/relay/primal.md) | Via cache server | User relay list fetched from the cache's get_user_relays (NIP-65 kind:10002, returned as Kind.UserRelays=10000139) into relaySettings; falls back to get_default_relays when empty. |
| [Coracle](evidences/relay/coracle.md) | kind:10002 (NIP-65) | Outbox model via @welshman/router (Router.ForUser()/FromPubkeys()). INDEXER_RELAYS fetch the relay list, limited by relay_limit (default 3). |
| [noStrudel](evidences/relay/nostrudel.md) | Outbox model (NIP-65) | Home timeline subscribes to the target users' outbox map; falls back to localSettings.fallbackRelays when outbox is unavailable. Uses applesauce-loaders createOutboxTimelineLoader. |
| [Amethyst](evidences/relay/amethyst.md) | kind:10002 (NIP-65) | Write/read relays from AdvertisedRelayListEvent (outboxFlow/inboxFlow); falls back to eventFinderRelays / bootstrapInbox. Indexer relays (DefaultIndexerRelayList) fetch other users' relay lists. |
| [Damus](evidences/relay/damus.md) | kind:10002 (NIP-65) | Fallback order: in-memory cache -> kind:10002 -> kind:3 -> UserDefaults -> Bootstrap relays. kind:10002 subscribed in real time; the in-memory cache is lastSetRelayList. |
| [algia](evidences/relay/algia.md) | kind:10002 (NIP-65) | Initial connection via config file (~/.config/algia/config.json); overwritten by kind:10002 read relays (only when rm is non-empty). Existing Search flags inherited. |
| [kakoi](evidences/relay/kakoi.md) | From settings | relays.json (GetEnabledRelays extracts enabled=true entries), falls back to default relays. kind:10002 / outbox not used; manually edited fixed list. |

# [Search Relays](evidences/search-relay/)
We researched which relays each client uses for search.

*Last updated: 2026-06-25*

| Client | Search Relays |
|--------|--------------|
| [nostter](evidences/search-relay/nostter.md) | nostr.wine, search.nos.today (overridable via VITE_SEARCH_RELAYS) |
| [Rabbit](evidences/search-relay/rabbit.md) | relay.nostr.band, search.nos.today |
| [Lumilumi](evidences/search-relay/lumilumi.md) | search.nos.today, nostr.wine, cagliostr.compile-error.net (user can override via kind:10007) |
| [Nos Haiku](evidences/search-relay/nos-haiku.md) | search.nos.today (channel kind:40/41 full-text search only) |
| [nullnull](evidences/search-relay/nullnull.md) | search.nos.today (overridable via NEXT_PUBLIC_SEARCH_RELAY) |
| [Nosame](evidences/search-relay/nosame.md) | None (full-text search not implemented, NIP-50 unsupported) |
| [flowgazer](evidences/search-relay/flowgazer.md) | None (NIP-50 full-text search not implemented) |
| [Yakihonne](evidences/search-relay/yakihonne.md) | search.nos.today, relay.ditto.pub, nostr.polyserv.xyz + user's kind:10007 relays |
| [iris](evidences/search-relay/iris.md) | No dedicated search relay (NIP-50 search to connected relays; profile search is local via Fuse.js) |
| [Primal](evidences/search-relay/primal.md) | Via cache server (cache: "search" / "user_search" / "advanced_search") |
| [Coracle](evidences/search-relay/coracle.md) | nostr.wine, search.nos.today |
| [noStrudel](evidences/search-relay/nostrudel.md) | relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine (user's kind:10007 list takes priority) |
| [Amethyst](evidences/search-relay/amethyst.md) | nostr.wine, relay.noswhere.com, search.nos.today, antiprimal.net, relay.ditto.pub (fallback when kind:10007 unset) |
| [Damus](evidences/search-relay/damus.md) | None (local full-text search via nostrdb, no search relay) |
| [algia](evidences/search-relay/algia.md) | relay.nostr.band (relays with Search:true in config) |
| [kakoi](evidences/search-relay/kakoi.md) | None (NIP-50 full-text search not implemented) |

# [Reactions](evidences/reaction-for-events/)
We researched how each client collects and crawls reactions for events.

*Last updated: 2026-06-25*

| Client | Collection Method |
|--------|------------------|
| [nostter](evidences/reaction-for-events/nostter.md) | One-shot subscription when the note detail page opens (kinds [1,6,7,9735] / #e) |
| [Rabbit](evidences/reaction-for-events/rabbit.md) | Batch fetch (2s interval, max 150) |
| [Lumilumi](evidences/reaction-for-events/lumilumi.md) | Batch subscribe for visible events (e/a tags, 1s debounce, kinds [7,6,16] and [9735]) |
| [Nos Haiku](evidences/reaction-for-events/nos-haiku.md) | Batch fetch (1s buffer, limit 500) + forward real-time subscription |
| [nullnull](evidences/reaction-for-events/nullnull.md) | Batch fetch (kind:7 by #e, limit 500) |
| [Nosame](evidences/reaction-for-events/nosame.md) | No reaction fetch (send only); kind:7 is send-only |
| [flowgazer](evidences/reaction-for-events/flowgazer.md) | Opportunistic count from stream + explicit fetch for own (#p/#e); no per-note subscription |
| [Yakihonne](evidences/reaction-for-events/yakihonne.md) | Per-note subscription by #e (kind 7/6/1/9735, incremental via since) + Dexie cache, WoT score filtering |
| [iris](evidences/reaction-for-events/iris.md) | Subscribe per event by #e (kind:7, closeOnEose); keeps only the latest per author |
| [Primal](evidences/reaction-for-events/primal.md) | Pre-aggregated data from cache server (NoteStats kind:10000100; kind:7 not subscribed directly) |
| [Coracle](evidences/reaction-for-events/coracle.md) | Per-note fetch on display (kind:7/9735 to Router.Replies(event) relays, aggregated by NoteReducer) |
| [noStrudel](evidences/reaction-for-events/nostrudel.md) | Per-event via ReactionsModel from local event store (kind:7) |
| [Amethyst](evidences/reaction-for-events/amethyst.md) | Per-event batch fetch + real-time subscription, aggregate note IDs per relay by 'e' tag (kinds 7/6/16/zap, limit 1000) |
| [Damus](evidences/reaction-for-events/damus.md) | Real-time subscription of own notifications via #p filter (kind 7/6/9735/1) |
| [algia](evidences/reaction-for-events/algia.md) | No reaction fetch (send only); like/unlike publishes/deletes kind:7 |
| [kakoi](evidences/reaction-for-events/kakoi.md) | No dedicated reaction collection; timeline subscription (kinds [1,6,7,16]) receives kind:7/6/16 together |

# [Image Upload](evidences/image-upload/)
Which media server (upload destination provider) each client uploads to when attaching images/media to a post. Clients without an image upload feature (Nosame, kakoi) are omitted.

*Last updated: 2026-07-03*

- ✅ (protocol columns) = supports uploading via that method. ✅ (provider columns) = bundles that provider as a built-in option.
- "Other" = methods besides NIP-96/Blossom. nullnull and noStrudel use nostr.build's own API (`/api/v2/upload/files`); Yakihonne has an own S3 backend (`/api/v1/file-upload`, for the special value); Amethyst supports NIP-95 (blob embedded in the event); flowgazer delegates to the external app ehagaki.
- "Provider" is aggregated per operator (`nostr.build` includes Blossom `blossom.nostr.build`; `nostrcheck.me` includes `cdn.nostrcheck.me`; `yabu.me` includes `share.yabu.me`; `sovbit` = `files.sovbit.host`/`cdn.sovbit.host`; `primal` = blossom.primal.net; `satellite` = cdn.satellite.earth; `nostpic` = nostpic.com; `nostrmedia` = nostrmedia.com; `yakihonne` = blossom.yakihonne.com).
- Most clients also allow adding arbitrary NIP-96/Blossom server URLs and syncing server lists via kind:10063 (Blossom) / kind:10096 (NIP-96). See each evidence file for details.

<table>
<thead>
<tr><th rowspan="2">Client</th><th colspan="3">Protocol</th><th colspan="12">Provider</th></tr>
<tr><th>NIP-96</th><th>Blossom</th><th>Other</th><th>nostr.build</th><th>nostrcheck.me</th><th>primal</th><th>yabu.me</th><th>sovbit</th><th>blossom.band</th><th>nostr.download</th><th>satellite</th><th>nostpic</th><th>void.cat</th><th>nostrmedia</th><th>yakihonne</th></tr>
</thead>
<tbody>
<tr><td><a href="evidences/image-upload/nostter.md">nostter</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/rabbit.md">Rabbit</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/lumilumi.md">Lumilumi</a></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nos-haiku.md">Nos Haiku</a></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nullnull.md">nullnull</a></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/flowgazer.md">flowgazer</a> *3</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/yakihonne.md">Yakihonne</a></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td></tr>
<tr><td><a href="evidences/image-upload/iris.md">iris</a> *1</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/primal.md">Primal</a></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/coracle.md">Coracle</a></td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/nostrudel.md">noStrudel</a></td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td></td><td></td><td></td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/amethyst.md">Amethyst</a> *2</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td></tr>
<tr><td><a href="evidences/image-upload/damus.md">Damus</a></td><td align="center">✅</td><td></td><td></td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
<tr><td><a href="evidences/image-upload/algia.md">algia</a> *4</td><td align="center">✅</td><td align="center">✅</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</tbody>
</table>

*1 iris: iris subscribers get `upload.iris.to` (Blossom) prepended to the defaults.
*2 Amethyst: in addition to the providers above, also bundles `24242.io` and `blossom.azzamo.media` (10 Blossom servers total).
*3 flowgazer: image posting is delegated to the external app ehagaki (`lokuyow.github.io/ehagaki`), so it bundles no provider of its own.
*4 algia: CLI. No bundled default; requires the `file-servers` config or `--server`.

# Frameworks
We researched the frameworks and libraries used in each client's implementation.

*Last updated: 2026-06-25*

| Client | Language | UI | Nostr Access | Other Libraries |
|--------|----------|-----|--------------|-----------------|
| nostter | TypeScript | Svelte 5 + SvelteKit (adapter-cloudflare) | rx-nostr, rx-nostr-crypto, nostr-tools, @rust-nostr/nostr-sdk, nostr-typedef | rxjs, @melt-ui/svelte + melt, svelte-i18n, svelte-persisted-store, dexie, light-bolt11-decoder, twitter-text, virtua, nip07-awaiter, Vite |
| Rabbit | TypeScript | SolidJS (solid-js/web render) + @solidjs/router + @solidjs/meta + Tailwind CSS | nostr-tools 2.10.1, nostr-wasm (custom usePool/useSubscription/useBatchedEvents wrappers) | @tanstack/solid-query, i18next + i18next-browser-languagedetector, zod, lodash, idb-keyval, bech32, bolt11, emoji-mart, webln, qrcode, @thisbeyond/solid-dnd, Vite + Vitest |
| Lumilumi | TypeScript | Svelte 5 + SvelteKit (adapter-cloudflare / adapter-auto, Vite PWA, TailwindCSS) | rx-nostr, rx-nostr-crypto, nostr-tools, nostr-typedef, @konemono/nostr-login, nip07-awaiter | @tanstack/svelte-query, melt / @melt-ui (svelte/pp), @konemono/svelte5-i18n, @konemono/nostr-content-parser, markdown-it, light-bolt11-decoder, qrcode, leaflet/sveaflet, three/@google/model-viewer |
| Nos Haiku | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, applesauce-core (EventStore + NIP-65 helpers), @rx-nostr/crypto | @konemono/nostr-login, nostr-zap, svelte-i18n, svelte-persisted-store, emoji-mart, canvas-confetti, mediabunny |
| nullnull | JavaScript | React 18 + Next.js 14 (Android via Capacitor, Tailwind CSS) | nostr-tools (SimplePool), rx-nostr, rxjs | nosskey-sdk (NIP-46 remote signing), @capacitor/* (core/cli/android/app) |
| Nosame | Vanilla JavaScript (ES Modules) | None (raw DOM manipulation / index.html + hand-written UIManager class) | Custom implementation (direct WebSocket relay connection, custom Bech32 for npub, NIP-07 window.nostr for signing). No nostr-tools or similar external libs | No external JS libraries / package.json. Uses only Google Fonts (Sawarabi Mincho) |
| flowgazer | JavaScript (Vanilla JS, ES2015+ classes) | None (raw DOM manipulation, multiple HTML entries: index.html etc.) | nostr-tools 2.17.0 (nostr.bundle.js via unpkg CDN) + custom modules relay-manager.js / data-store.js / event-bus.js / view-state.js (raw WebSocket REQ/EVENT) | Only external dep is nostr-tools. Derived from motherfucking-nostr-client (jiftechnify). WTFPL license. Bundles lite/ and okkake/ variant implementations |
| Yakihonne | JavaScript (JSX) | React 19 + Next.js 15 (App Router, next-pwa) | @nostr-dev-kit/ndk (NDK, outbox model), nostr-tools | @reduxjs/toolkit + react-redux, dexie + dexie-react-hooks + @nostr-dev-kit/ndk-cache-dexie (IndexedDB), i18next/next-i18next, @noble/hashes, @noble/secp256k1, react-virtuoso |
| iris | TypeScript | React 19 + Vite (custom stack router, no react-router, Web/PWA, no Tauri/Electron) | NDK (vendored in repo src/lib/ndk, @nostr-dev-kit/ndk fork) + nostr-tools + nostr-wasm, relay connection via Worker transport (NDKWorkerTransport) | zustand, dexie (IndexedDB), fuse.js, nostr-social-graph, @cashu/cashu-ts, nostr-double-ratchet, localforage, lodash |
| Primal | TypeScript (5.2) | SolidJS 1.9 + @solidjs/router, Vite 4 (vite-plugin-solid), Kobalte UI, Sass | Custom Primal cache server protocol (WebSocket + pako/zlib compression, cache: [...] REQ). nostr-tools 2.23 for signing/helpers, NIP-46 remote signing supported | pako (compression), @cookbook/solid-intl (i18n), tiptap/milkdown (editor), @cashu/cashu-ts, light-bolt11-decoder, blossom-client-sdk, dompurify, fuse.js/@nozbe/microfuzz, hls.js |
| Coracle | TypeScript | Svelte 4 + Vite (PWA, iOS/Android via Capacitor) | @welshman/* (app, net, router, feeds, util, signer, store, content, editor, lib), nostr-tools | @noble/curves, fuse.js, idb, marked, hls.js, tippy.js, @getalby/sdk, @getalby/bitcoin-connect, tailwindcss |
| noStrudel | TypeScript | React 19 + Chakra UI (Vite) | applesauce-* (core/loaders/relay/react/common etc., next version), nostr-tools | rxjs, nostr-idb (IndexedDB cache), hash-sum, applesauce-sqlite/wallet |
| Amethyst | Kotlin (Android / Kotlin Multiplatform) | Jetpack Compose / Compose Multiplatform 1.11.1 (Material3) | Custom quartz module (Nostr KMP library, protocol implementation), commons module (shared relay client) | kotlinx.coroutines + Flow, OkHttp 5.4.0, Jackson 2.22.0, secp256k1-kmp-jni 0.23.0, Koin/manual DI, navigation-compose, custom quic (pure-Kotlin QUIC/HTTP3/WebTransport) |
| Damus | Swift + C (iOS/macOS) | SwiftUI | Custom implementation (RelayPool / NostrNetworkManager) + nostrdb (C-based local event store / full-text search) | nostr-sdk-swift (rust-nostr), negentropy-swift (relay sync), secp256k1.swift, CryptoSwift, Kingfisher, swift-markdown-ui, GSPlayer, SwiftSoup, swift-collections, sentry-cocoa |
| algia | Go (go 1.24.1) | CLI (urfave/cli v2). Also has an MCP server mode (mark3labs/mcp-go) | nbd-wtf/go-nostr v0.52.3 (incl. sdk/pool) | fatih/color, mattn/go-colorable/go-isatty, mdp/qrterminal (QR display), mark3labs/mcp-go (MCP) |
| kakoi | C# (.NET 8 / net8.0-windows7.0) | Windows Forms (WinForms) + WebView2 | NNostr.Client (bundled and modified in repo) | Google_GenerativeAI (Gemini), Microsoft.Web.WebView2, NTextCat (language detection), SkiaSharp, Svg.Skia, CredentialManagement, SSTPLib (Ukagaka SSTP integration) |

# Researched Clients
- web:
  - [nostter](https://nostter.app/)
  - [Rabbit](https://rabbit.syusui.net/)
  - [Lumilumi](https://lumilumi.app/)
  - [Nos Haiku](https://nos-haiku.vercel.app/)
  - [nullnull](https://www.nullnull.app/)
  - [Nosame](https://invertedtriangle358.github.io/Nosame/)
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

# References
- Clients
  - nostter: https://github.com/SnowCait/nostter
  - Rabbit: https://github.com/syusui-s/rabbit
  - Lumilumi: https://github.com/TsukemonoGit/lumilumi
  - Nos Haiku: https://github.com/nikolat/nos-haiku
  - nullnull: https://github.com/tami1A84/null--nostr
  - Nosame: https://github.com/invertedtriangle358/Nosame
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
