[<< back](../README.md) | [Japanese](README-ja.md) | [English]

# Nostr Client Research
A repository for researching implementation differences between Nostr clients.
Contact npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk or [github issue](https://github.com/koteitan/nostr-research/issues) to request research on specific implementation differences. Client suggestions and PRs are also welcome.

# Bootstrap Relays
There are various methods for determining which relays Nostr subscribes to, such as using kind:10002 or the outbox model. Much of this information is contained in relay metadata. To obtain this metadata, clients must first connect to some relay. We researched how each client determines its bootstrap relays.

*Last updated: 2025-12-21*

| Client | Bootstrap Relays | Notes |
|--------|-----------------|-------|
| [nostter](bootstrap-relay/nostter.md) | relay.nostr.band, nos.lol, relay.damus.io | Japanese relays added when language is Japanese (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net) |
| [Rabbit](bootstrap-relay/rabbit.md) | relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp | Japanese relays added when browser language is Japanese (relay-jp.nostr.wirednet.jp, holybea.com, r.kojira.io, yabu.me) |
| [Lumilumi](bootstrap-relay/lumilumi.md) | directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social | Fallback: relay.nostr.band, nos.lol, nostr.bitcoiner.social |
| [Nos Haiku](bootstrap-relay/nos-haiku.md) | directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social | |
| [nullnull](bootstrap-relay/nullnull.md) | yabu.me | |
| [Nosame](bootstrap-relay/nosame.md) | relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, relay.barine.co | |
| [flowgazer](bootstrap-relay/flowgazer.md) | r.kojira.io | |
| [Yakihonne](bootstrap-relay/yakihonne.md) | nostr-01.yakihonne.com, nostr-02.yakihonne.com, relay.damus.io, relay.nostr.band, relay.nsec.app, monitorlizard.nostr1.com | |
| [iris](bootstrap-relay/iris.md) | temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, nos.lol | |
| [Primal](bootstrap-relay/primal.md) | Via cache server (cache.primal.net) | Users can select cache instance |
| [Coracle](bootstrap-relay/coracle.md) | relay.damus.io, nos.lol (env VITE_DEFAULT_RELAYS) | |
| [noStrudel](bootstrap-relay/nostrudel.md) | relay.primal.net, relay.damus.io, nos.lol | |
| [Amethyst](bootstrap-relay/amethyst.md) | relay.nostr.band, relay.damus.io, relay.primal.net, nostr.mom, nos.lol, nostr.bitcoiner.social, nostr.oxtr.dev | bootstrapInbox (7 relays) |
| [Damus](bootstrap-relay/damus.md) | relay.damus.io, nostr.land, nostr.wine, nos.lol | Regional relays added based on device locale (Japan: wirednet.jp, yabu.me, kojira.io / Thailand: siamstr.com / Germany: einundzwanzig.space) |
| [algia](bootstrap-relay/algia.md) | relay.nostr.band | |
| [kakoi](bootstrap-relay/kakoi.md) | yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band | |

# Relays
We researched how each client obtains relays for building the home timeline.

*Last updated: 2025-12-21*

| Client | Relay Source | Details |
|--------|-------------|---------|
| [nostter](relay/nostter.md) | kind:10002 (NIP-65) |  |
| [Rabbit](relay/rabbit.md) | Settings -> localStorage |
| [Lumilumi](relay/lumilumi.md) | kind:10002 (NIP-65) or settings | Local settings priority, then NIP-65, finally bootstrap relays |
| [Nos Haiku](relay/nos-haiku.md) | Outbox model (NIP-65) |  |
| [nullnull](relay/nullnull.md) | Settings -> localStorage | Single relay |
| [Nosame](relay/nosame.md) | Settings -> localStorage | Fallback to bootstrap relays |
| [flowgazer](relay/flowgazer.md) | Settings -> localStorage | Single relay |
| [Yakihonne](relay/yakihonne.md) | Outbox model (NIP-65) | Fallback to bootstrap relays when outbox disabled |
| [iris](relay/iris.md) | Outbox model (NIP-65) |  |
| [Primal](relay/primal.md) | Read: via cache server, Write: kind:10002 (NIP-65) | Server selectable, user relay settings managed via kind:10002 |
| [Coracle](relay/coracle.md) | kind:10002 (NIP-65) | Negentropy support, relay quality scoring, multiple relay categories |
| [noStrudel](relay/nostrudel.md) | Outbox model (NIP-65) | Uses applesauce-relay |
| [Amethyst](relay/amethyst.md) | kind:10002 (NIP-65) | Purpose-specific relay sets (kind:10002, DM, search, indexer, block) |
| [Damus](relay/damus.md) | kind:10002 (NIP-65) | Fallback order: kind:10002 -> kind:3 -> UserDefaults -> Bootstrap relays |
| [algia](relay/algia.md) | kind:10002 (NIP-65) | Initial connection via config file (~/.config/algia/config.json), overwritten by kind:10002 |
| [kakoi](relay/kakoi.md) | From settings | relays.json, auto-connect on startup |

# Search
We researched which relays each client uses for search.

*Last updated: 2025-12-21*

| Client | Search Relays |
|--------|--------------|
| [nostter](search-relay/nostter.md) | relay.nostr.band, search.nos.today |
| [Rabbit](search-relay/rabbit.md) | relay.nostr.band, search.nos.today |
| [Lumilumi](search-relay/lumilumi.md) | relay.nostr.band, search.nos.today |
| [Nos Haiku](search-relay/nos-haiku.md) | search.nos.today |
| [nullnull](search-relay/nullnull.md) | search.nos.today |
| [Nosame](search-relay/nosame.md) | None |
| [flowgazer](search-relay/flowgazer.md) | None |
| [Yakihonne](search-relay/yakihonne.md) | search.nos.today, relay.nostr.band, relay.ditto.pub, nostr.polyserv.xyz |
| [iris](search-relay/iris.md) | Local search (from cache) |
| [Primal](search-relay/primal.md) | Via cache server |
| [Coracle](search-relay/coracle.md) | relay.nostr.band, nostr.wine, search.nos.today |
| [noStrudel](search-relay/nostrudel.md) | relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine |
| [Amethyst](search-relay/amethyst.md) | relay.nostr.band, nostr.wine, relay.noswhere.com, search.nos.today |
| [Damus](search-relay/damus.md) | No full-text search (hashtags and users only) |
| [algia](search-relay/algia.md) | relay.nostr.band (search flag) |
| [kakoi](search-relay/kakoi.md) | None |

# Reaction Collection Methods
We researched how each client collects and crawls reactions for events.

*Last updated: 2025-12-21*

| Client | Collection Method |
|--------|------------------|
| [nostter](reaction-for-events/nostter.md) | Subscribe when detail page opened |
| [Rabbit](reaction-for-events/rabbit.md) | Batch fetch (2s interval, max 150) |
| [Lumilumi](reaction-for-events/lumilumi.md) | Subscribe per post |
| [Nos Haiku](reaction-for-events/nos-haiku.md) | Batch fetch (1s buffer, limit: 500) + real-time subscription |
| [nullnull](reaction-for-events/nullnull.md) | Batch fetch (limit: 500) |
| [Nosame](reaction-for-events/nosame.md) | No reaction fetch (send only) |
| [flowgazer](reaction-for-events/flowgazer.md) | Fetch reactions to own posts |
| [Yakihonne](reaction-for-events/yakihonne.md) | Batch fetch + Dexie cache, WoT score filtering |
| [iris](reaction-for-events/iris.md) | Subscribe per event (closeOnEose) |
| [Primal](reaction-for-events/primal.md) | Pre-aggregated data from cache server |
| [Coracle](reaction-for-events/coracle.md) | Filter kind:7 from context events |
| [noStrudel](reaction-for-events/nostrudel.md) | Subscribe per event |
| [Amethyst](reaction-for-events/amethyst.md) | Batch fetch + real-time subscription, aggregate note IDs per relay |
| [Damus](reaction-for-events/damus.md) | Get kind:7 via notification filter, dedupe with LikeCounter |
| [algia](reaction-for-events/algia.md) | No reaction fetch (send only) |
| [kakoi](reaction-for-events/kakoi.md) | No reaction collection |

# Frameworks
We researched the frameworks and libraries used in each client's implementation.

*Last updated: 2025-12-21*

| Client | Language | UI | Nostr Access | Other Libraries |
|--------|----------|-----|--------------|-----------------|
| nostter | TypeScript | Svelte + SvelteKit | rx-nostr, nostr-tools, @rust-nostr/nostr-sdk | rxjs, Melt UI, svelte-i18n |
| Rabbit | TypeScript | SolidJS | nostr-tools, nostr-wasm | @tanstack/solid-query, TailwindCSS, i18next, zod |
| Lumilumi | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, @konemono/nostr-login | TanStack Query, Leaflet, markdown-it, Melt UI |
| Nos Haiku | TypeScript | Svelte 5 + SvelteKit | rx-nostr, nostr-tools, applesauce-core, nostr-login, nostr-zap | svelte-i18n, emoji-mart, mediabunny |
| nullnull | JavaScript | Next.js 14 + React 18 | nostr-tools, nostr-login, nosskey-sdk, rx-nostr | Tailwind CSS, rxjs |
| Nosame | JavaScript | Vanilla JS | None (custom + NIP-07) | None |
| flowgazer | JavaScript | Vanilla JS (modular) | nostr-tools | marked.js |
| Yakihonne | JavaScript | React + Next.js | NDK, nostr-tools | Redux Toolkit, Dexie, i18next, react-markdown |
| iris | TypeScript | React + Tauri | nostr-tools, nostr-social-graph, nostr-wasm | Zustand, Dexie, DaisyUI, Leaflet |
| Primal | TypeScript | SolidJS | nostr-tools | Tiptap, Milkdown, HLS.js, @cashu/cashu-ts |
| Coracle | TypeScript | Svelte | @welshman/*, nostr-tools | TailwindCSS, Capacitor, Bitcoin Connect, Fuse.js |
| noStrudel | TypeScript | React 19 + Chakra UI | nostr-tools, applesauce-* | React Router, CodeMirror, Chart.js, Leaflet |
| Amethyst | Kotlin | Jetpack Compose | quartz (in-house) | OkHttp, Coil, Media3, secp256k1-kmp |
| Damus | Swift + C | SwiftUI | nostrdb (in-house) | Flatbuffers, Lightning Network integration |
| algia | Go | CLI | go-nostr | None |
| kakoi | C# | Windows Forms (.NET 8.0) | NNostr.Client | SSTPLib, NTextCat, SkiaSharp, Gemini AI |

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
