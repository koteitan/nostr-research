[<< 戻る](../README-ja.md) | [Japanese] | [English](README-en.md)

# リレーインスタンス NIP-11 情報

Last Checked: 2026-06-26

## 概要

各リレーインスタンスの NIP-11 制限情報を調査した結果。

## 制限情報一覧

| リレー | ソフトウェア | max_message_length | max_subscriptions | max_limit | max_filters | max_subid_length | max_event_tags | max_content_length |
|--------|-------------|-------------------|-------------------|-----------|-------------|-----------------|----------------|-------------------|
| [r.kojira.io](https://r.kojira.io) | strfry no-git-commits | 131,072 | 30 | 1,000 | - | - | - | - |
| [relay.damus.io](https://relay.damus.io) | strfry 1.1.0-1-g691a533f11eb | 1,000,000 | 200 | 500 | - | - | - | - |
| [relay-jp.nostr.wirednet.jp](https://relay-jp.nostr.wirednet.jp) | strfry 1.0.4-2-g5a950be | 131,072 | 8 | 200 | - | - | - | - |
| [nostr.compile-error.net](https://nostr.compile-error.net) | nostr-relay 0.0.240 | 524,288 | 20 | 500 | - | 100 | 100 | 16,384 |
| [nrelay-jp.c-stellar.net](https://nrelay-jp.c-stellar.net) | strfry 1.0.4 | 1,049,600 | 50 | 500 | - | - | - | - |
| [nostream.ocha.one](https://nostream.ocha.one) | nostream 1.25.2 | 524,288 | 1,000 | 5,000 | 2,500 | 256 | 2,500 | 65,536 |
| [yabu.me](https://yabu.me) | strfry 1.0.4 | 1,310,720 | 50 | 500 | - | - | - | - |
| [relay.primal.net](https://relay.primal.net) | strfry 1.0.3-1-g60d35a6 | 1,000,000 | 20 | 500 | - | - | - | - |
| [relay.snort.social](https://relay.snort.social) | strfry 1.1.0-98-gb80cda3 | 131,072 | 100 | 500 | - | - | - | - |
| [snowflare.cc](https://snowflare.cc) | snowflare 0.1.0 | - | 20 | 500 | 10 | 50 | - | - |
| [relay.westernbtc.com](https://relay.westernbtc.com) | strfry no-git-commits | 131,072 | 20 | 500 | - | - | - | - |

**注**: `-` は NIP-11 で公開されていない項目
