← [README](../README.md)

# Coracle 画像アップロード先

## 結論
- **アップロード先**: 既定 `blossom.nostr.build`（Blossom）。`env.BLOSSOM_URLS`（blossom.nostr.build / blossom.primal.net / cdn.satellite.earth / void.cat）を候補に持ち、ユーザーの kind:10063 先頭があればそれを優先。

## ソースコード

**ファイル**: `src/engine/commands.ts` (行 82-97), `.env.template` (行 10)

```typescript
export const uploadFile = async (server: string, file: File, compressorOpts = {}) => {
  if (!file.type.match("image/(webp|gif)")) {
    file = await stripExifData(file, compressorOpts)
  }
  const hashes = [await sha256(await file.arrayBuffer())]
  const $signer = signer.get() || Nip01Signer.ephemeral()
  const authEvent = await $signer.sign(makeBlossomAuthEvent({action: "upload", server, hashes}))
  const res = await uploadBlob(server, file, {authEvent})
  ...
}
// .env.template:10
// BLOSSOM_URLS = https://blossom.nostr.build,https://blossom.primal.net,https://cdn.satellite.earth,https://void.cat
```

## 説明
- compose エディタ（`src/app/editor/index.ts`、ドラッグ&ドロップ/ペースト対応）とアバター等の `src/partials/ImageInput.svelte` から `uploadFile()` を呼ぶ。
- プロトコルは Blossom のみ。`makeBlossomAuthEvent({action:"upload", server, hashes})` で署名した認証イベントを付けて `uploadBlob(server, file, {authEvent})` を実行する（BUD-01/02 の `PUT /upload` 相当）。既定設定でも `upload_type: "blossom"`（`src/engine/state.ts:257`）。
- 既定のアップロード先は `env.BLOSSOM_URLS` の先頭 = `blossom.nostr.build`。実際に選ばれるのはユーザーの Blossom サーバーリスト先頭 `server` タグ、無ければ `first(env.BLOSSOM_URLS)`。
- 設定 `UserSettings.svelte` の "Blossom Provider URLs" フィールドで編集し、kind:10063（`BLOSSOM_SERVERS`）の `server` タグリストとして publish する。

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/engine/commands.ts#L82-L97

---
← [README](../README.md)
