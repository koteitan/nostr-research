← [README](../README.md)

# Amethyst 画像アップロード先

## 結論
- **アップロード先**: 既定 `blossom.band`（Blossom）。同梱10サーバーは全て Blossom（blossom.band / cdn.nostrcheck.me / 24242.io / blossom.azzamo.media / blossom.yakihonne.com / blossom.primal.net / cdn.sovbit.host / nostr.download / cdn.satellite.earth / nostrmedia.com）。NIP-96・NIP-95 にも対応。

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/ui/actions/mediaServers/ServerName.kt` (行 38-50)

```kotlin
val DEFAULT_MEDIA_SERVERS: List<ServerName> =
    listOf(
        ServerName("Nostr.Build", "https://blossom.band/", ServerType.Blossom),
        ServerName("Nostrcheck.me", "https://cdn.nostrcheck.me", ServerType.Blossom),
        ServerName("24242.io", "https://24242.io/", ServerType.Blossom),
        ServerName("Azzamo", "https://blossom.azzamo.media", ServerType.Blossom),
        ServerName("YakiHonne", "https://blossom.yakihonne.com/", ServerType.Blossom),
        ServerName("Primal", "https://blossom.primal.net/", ServerType.Blossom),
        ServerName("Sovbit", "https://cdn.sovbit.host", ServerType.Blossom),
        ServerName("Nostr.Download", "https://nostr.download", ServerType.Blossom),
        ServerName("Satellite (Paid)", "https://cdn.satellite.earth", ServerType.Blossom),
        ServerName("NostrMedia (Paid)", "https://nostrmedia.com", ServerType.Blossom),
    )
```

## 説明
- `UploadOrchestrator` が `server.type` で分岐し 3 方式に対応。`ServerType.Blossom`（BUD-01/02 `PUT <server>/upload`, sha256 認証、quartz `BlossomUploader`）、`ServerType.NIP96`（`/.well-known/nostr/nip96.json` を読み `api_url` へ multipart POST + NIP-98 認証）、`ServerType.NIP95`（80KB 以下をイベント内 blob として埋め込み）。
- 同梱10サーバーは全て Blossom。既定選択先 `defaultFileServer` は初期値 `DEFAULT_MEDIA_SERVERS[0]` = `blossom.band`（表示名 "Nostr.Build"）。NIP-96 はユーザーが手動でサーバー URL を追加したときに使われる（既定同梱は無し）。
- `AccountSettings.defaultFileServer` で既定先を保持し、投稿 UI でサーバー選択可能。メディアサーバー一覧は kind:10063 Blossom Server List（quartz `BlossomServersEvent.KIND = 10063`）として発行/同期され、ユーザーの kind:10063 と既定をマージする。

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/ui/actions/mediaServers/ServerName.kt#L38-L50

---
← [README](../README.md)
