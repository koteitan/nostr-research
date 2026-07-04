← [README](../README.md)

# Lumilumi 画像アップロード先

## 結論
- **アップロード先**: 既定 `blossom.band`（Blossom）。NIP-96 と Blossom の両対応。同梱 NIP-96 5件（nostrcheck.me / nostr.build / files.sovbit.host / nostpic.com / yabu.me）、Blossom 5件（blossom.band / cdn.nostrcheck.me / nostr.download / blossom.primal.net / cdn.satellite.earth）。

## ソースコード

**ファイル**: `src/lib/func/constants.ts` (行 58-129)

```typescript
export const nip96MediaUploader = [
  "https://nostrcheck.me", "https://nostr.build",
  "https://files.sovbit.host", "https://nostpic.com", "https://yabu.me",
];
export const blossomMediaUploader = [
  "https://blossom.band", "https://cdn.nostrcheck.me", "https://nostr.download",
  "https://blossom.primal.net", "https://cdn.satellite.earth",
];
export const initUploaderOption: UploaderOption = {
  type: "blossom",
  address: blossomMediaUploader[0],   // => https://blossom.band
};
```

## 説明
- `filesUpload()`（`src/lib/func/upload.ts:409`）が `uploader.type` で分岐。`"nip96"` は NIP-98 認証トークン付きで NIP-96 の `api_url` へ POST（`readServerConfig` で `/.well-known/nostr/nip96.json` を解決）、`"blossom"` は `nostr-tools/nipb7` の `BlossomClient.uploadFile()` で `PUT /upload` を実行する。
- 既定のアップロード先は `initUploaderOption`（constants.ts:126-129）で Blossom の `blossom.band`。
- 設定 `UploaderSelect.svelte` のドロップダウンで選択し `localStorage`（`STORAGE_KEYS.UPLOADER`）に保存。さらにユーザー自身の kind:10063（Blossom サーバーリスト, BUD-03 の `server` タグ）を取得して候補の先頭にマージする。NIP-96 の kind:10096 は候補への取り込みには使わず、ハードコード候補のみ。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/func/constants.ts

---
← [README](../README.md)
