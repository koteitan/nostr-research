← [README](../README.md)

# Nostrism 画像アップロード先

## 結論
- **アップロード先**: NIP-96 のみ。初回起動時に既定サーバとして `nostrcheck.me` と `nostr.build` をシードする。
- 設定画面のワンタップ候補（プリセット）として `nostr.build` / `nostrcheck.me` / `nostpic.com` / `nostrmedia.com` / `files.sovbit.host` を提示し、有効なサーバを上から順に試す。Blossom は未対応。

## ソースコード

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt` (行 3723, 3640-3706)

```kotlin
/** [M11] 既定のメディアサーバ(NIP-96)。start() で insert-if-absent して投入する。 */
val DEFAULT_MEDIA_SERVERS = listOf("https://nostrcheck.me", "https://nostr.build")

suspend fun uploadImage(bytes: ByteArray, mime: String, filename: String = "image"): String? =
    withContext(Dispatchers.Default) {
        for (s in q.enabledMediaServers().executeAsList()) {   // 有効なサーバを上から順に試す
            val url = runCatching { uploadToServer(s.url, bytes, mime, filename) }.getOrNull()
            if (!url.isNullOrBlank()) return@withContext url
        }
        null
    }

private suspend fun uploadToServer(server: String, bytes: ByteArray, mime: String, filename: String): String? {
    val apiUrl = discoverApiUrl(base) ?: "$base/api/v1/media"      // /.well-known/nostr/nip96.json → api_url
    val parts = formData { append("file", bytes, ...) }           // multipart, part 名 "file"
    val resp = uploadHttp.post(apiUrl) {
        header(HttpHeaders.Authorization, nip98Header(apiUrl, "POST"))  // NIP-98 (kind:27235)
        setBody(MultiPartFormDataContent(parts))
    }
    return parseUploadResponse(resp.bodyAsText())                 // nip94_event.tags "url" / トップレベル "url"
}
```

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/ui/Presets.kt` (行 83-91)

```kotlin
/** NIP-96 メディアサーバー候補（画像アップロード先）。2026-07 に well-known 応答で生存確認済み。 */
val MEDIA_PRESETS: List<Preset> = listOf(
    Preset("https://nostr.build", PresetCategory.Image),
    Preset("https://nostrcheck.me", PresetCategory.General),
    Preset("https://nostpic.com", PresetCategory.Image),
    Preset("https://nostrmedia.com", PresetCategory.Image),
    Preset("https://files.sovbit.host", PresetCategory.General),
    // cdn.satellite.earth は 2026-07 時点で NIP-96 応答なし(404)のため除外。
)
```

## 説明
- プロトコルは NIP-96。`start()` 時に `DEFAULT_MEDIA_SERVERS`（nostrcheck.me, nostr.build）を insert-if-absent で DB へ投入し、これが既定の有効サーバになる。
- 設定 > メディアサーバー画面で `MEDIA_PRESETS` のプリセット（nostr.build / nostrcheck.me / nostpic.com / nostrmedia.com / files.sovbit.host）をワンタップ追加でき、任意 URL も追加できる。アップロードは「有効なサーバを上から順に試し、最初に成功した URL を返す」方式。
- 各サーバへは `/.well-known/nostr/nip96.json` を引いて `api_url` を解決（失敗時は `/api/v1/media`）、`multipart/form-data`（part 名 `file`）で POST し、Authorization に NIP-98（kind:27235, tags=[["u",url],["method","POST"]]）を付与する。レスポンスは `nip94_event` の tags の `url`、無ければトップレベル `url` を抽出する。
- 投稿画面（`ComposeSheet`）は画像を圧縮してからアップロードし、本文末尾へ URL を付与する。動画は同じ NIP-96 経路で（圧縮せず）アップロードする。
- Blossom（BUD-01/02, kind:24242, kind:10063）や NIP-96 サーバーリスト（kind:10096）の読み込みには非対応。

## 参考
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/ui/Presets.kt

---
← [README](../README.md)
