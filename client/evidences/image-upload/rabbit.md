← [README](../README.md)

# Rabbit 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostr.build`（NIP-96）。同梱の選択肢は nostr.build / nostrcheck.me / files.sovbit.host / nostpic.com / void.cat / yabu.me の6サーバー（すべて NIP-96）。任意の NIP-96 サーバー URL も追加可能。

## ソースコード

**ファイル**: `src/utils/imageUpload.ts` (行 31-62), `src/core/useConfig.ts` (行 140)

```typescript
export const defaultFileServers = [
  { type: 'nip96', name: 'nostr.build',       serverUrl: 'https://nostr.build/' },
  { type: 'nip96', name: 'nostrcheck.me',     serverUrl: 'https://nostrcheck.me/' },
  { type: 'nip96', name: 'files.sovbit.host', serverUrl: 'https://files.sovbit.host/' },
  { type: 'nip96', name: 'nostpic.com',       serverUrl: 'https://nostpic.com/' },
  { type: 'nip96', name: 'void.cat',          serverUrl: 'https://void.cat/' },
  { type: 'nip96', name: 'yabu.me',           serverUrl: 'https://yabu.me/' },
] satisfies FileServerDefinition[];
// useConfig.ts:140 — 既定選択は defaultFileServers[0] = nostr.build
```

## 説明
- アップロードは NIP-96。`imageUpload.ts` が `nostr-tools/nip96` の `readServerConfig` で `/.well-known/nostr/nip96.json` を読み、`api_url` へ multipart POST する。認証は NIP-98（`nostr-tools/nip98` の `getToken`、`window.nostr` で署名）。Blossom / kind:10063 は未実装。
- 既定の送信先は `defaultFileServers[0]` = `nostr.build`。route96 系（void.cat 等）の相対 `api_url` にも `buildApiUrl()` で対応。
- 設定モーダル `FileServerSection.tsx` で同梱6サーバーから選択（`setFileServer`）、または任意の NIP-96 サーバー URL をカスタム追加（`addCustomFileServer`）できる。設定はローカル保存で、kind:10096/10063 イベントは読み書きしない。
- レスポンスの `nip94_event.tags` から URL を取り出し、`imeta` タグを組み立てて投稿に付与する（`useUploadFilesMutation.ts`）。

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/utils/imageUpload.ts

---
← [README](../README.md)
