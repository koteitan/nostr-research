← [README](../README.md)

# Damus 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostr.build`（NIP-96）。選択肢は nostr.build / nostrcheck.me の2択（enum ハードコード）。

## ソースコード

**ファイル**: `damus/Shared/Media/Models/MediaUploader.swift` (行 21-101), `damus/Features/Settings/Models/UserSettingsStore.swift` (行 112)

```swift
enum MediaUploader: String, CaseIterable, MediaUploaderProtocol, StringCodable {
    case nostrBuild
    case nostrcheck
    var postAPI: String {
        switch self {
        case .nostrBuild:  return "https://nostr.build/api/v2/nip96/upload"
        case .nostrcheck:  return "https://nostrcheck.me/api/v2/media"
        }
    }
}
// UserSettingsStore.swift:112 — 既定は @StringSetting default_value: .nostrBuild
```

## 説明
- 投稿画面（`PostView.swift`）やプロフィール画像編集からローカルファイルをアップロードして URL を得る。
- プロトコルは NIP-96。エンドポイントは nostr.build が `/api/v2/nip96/upload`、nostrcheck.me が `/api/v2/media`。両者とも `requiresNip98 = true` で、`AttachMediaUtility.swift` が `create_nip98_signature` で NIP-98 Authorization ヘッダを付与し、レスポンスの `nip94_event` の tags から URL を取り出す。Blossom は未実装。
- 既定のアップロード先は `default_media_uploader` 設定の既定値 `.nostrBuild` = `nostr.build`。
- 設定 > Appearance の「Image uploader」Picker で nostr.build と nostrcheck.me の2択から選べる。任意 URL 入力や kind:10096/10063 のサーバーリスト読み込みには非対応。

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Shared/Media/Models/MediaUploader.swift

---
← [README](../README.md)
