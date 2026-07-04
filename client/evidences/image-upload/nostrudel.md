← [README](../README.md)

# noStrudel 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostr.build`（独自 API v2, NIP-98 認証）。設定で Blossom に切替可能で、その場合はユーザーの kind:10063 サーバーリスト全てへ送信（未設定時 UI は nostr.download / blossom.primal.net を候補提示）。

## ソースコード

**ファイル**: `src/hooks/use-upload-file.ts` (行 30-61), `src/helpers/app-settings.ts` (行 97)

```typescript
if (mediaUploadService === "blossom" && mediaServers.length) {
  const blob = await simpleMultiServerUpload(
    mediaServers.map((s) => s.toString()),
    safeFile,
    signer,
  );
  ...
} else if (mediaUploadService === "nostr.build") {
  const response = await nostrBuildUploadImage(safeFile, signer);
  ...
}
// app-settings.ts:97 — DEFAULT_APP_SETTINGS の mediaUploadService は "nostr.build"
```

## 説明
- 投稿にローカル画像/動画をアップロードする `useUploadFile` フックが、設定値 `mediaUploadService` で 2 方式に分岐する。
- 既定値は `"nostr.build"`。この場合 `src/helpers/media-upload/nostr-build.ts` で `https://nostr.build/api/v2/upload/files` へ multipart POST し、NIP-98（`nip98.getToken`）の Authorization トークンを付与する独自 API（標準 NIP-96 の `/.well-known` や `/api/v2/media` は使わない）。
- `"blossom"` を選ぶと `blossom-client-sdk` の `multiServerUpload` でユーザーの kind:10063 Blossom サーバーリスト全てへ `PUT /upload` する。同梱の自動デフォルトサーバーは無く、未設定時は UI が nostr.download と blossom.primal.net を追加候補として提示する。
- 設定 > Post 画面のセレクトで nostr.build / Blossom を切替、Blossom サーバーは kind:10063 リスト編集画面で追加/削除/デフォルト指定できる。

## 参考
- https://github.com/hzrd149/nostrudel/blob/master/src/hooks/use-upload-file.ts

---
← [README](../README.md)
