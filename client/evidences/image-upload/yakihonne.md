← [README](../README.md)

# Yakihonne 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostr.build`（NIP-96）。NIP-96 と Blossom の両対応。同梱 NIP-96 リストは nostr.build / nostrmedia.com / nostrcheck.me / void.cat。Blossom はユーザーの kind:10063 先頭、未設定時は blossom.yakihonne.com にフォールバック。特殊値 "yakihonne" 選択時のみ独自 S3 バックエンド。

## ソースコード

**ファイル**: `src/Content/MediaUploaderServer.js` (行 1-6)

```javascript
export default  [
  ["nostr.build", "https://nostr.build/api/v2/nip96/upload"],
  ["nostrmedia.com", "https://nostrmedia.com/upload"],
  ["nostrcheck.me", "https://nostrcheck.me/api/v2/media"],
  ["void.cat", "https://void.cat/n96"],
];
```

## 説明
- `UploadFile.js` の `<input type="file">` → `FileUpload()`（`src/Helpers/Helpers.js:697`）がアップロードを実行し、返った URL を投稿本文/`imeta` タグに埋め込む。
- `FileUpload` は `localStorage` の `${pub}_media_service` で経路を切替。既定値 `"1"` = NIP-96 系 `regularServerFileUpload`、`"2"` = `blossomServerFileUpload`。
- NIP-96 経路の既定サーバーは `getSelectedServer()` が返す `MediaUploaderServer[0][1]` = nostr.build の NIP-96 エンドポイント。POST 時に kind:27235（NIP-98）認証イベントを Authorization ヘッダに付与し、`nip94_event.tags` から URL を取得。
- Blossom 経路（BUD-02）は `PUT {serverURL}/upload`。ユーザーの kind:10063 Blossom サーバーリストの先頭を使い、未設定時は `blossom.yakihonne.com` にフォールバック（ミラー機能あり）。
- 設定 `Settings/MediaUploader.js` でサービス切替・NIP-96 カスタムサーバー追加・Blossom サーバー管理（kind:10063 発行）が可能。NIP-96 選択が文字列 "yakihonne" のときのみ独自バックエンド `/api/v1/file-upload`（S3）を使う（既定同梱リストには含まれない）。

## 参考
- https://github.com/YakiHonne/web-app/blob/main/src/Content/MediaUploaderServer.js

---
← [README](../README.md)
