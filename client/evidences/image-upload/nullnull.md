← [README](../README.md)

# ぬるぬる 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostr.build`（独自 API `/api/v2/upload/files`, NIP-98 認証）。他に NIP-96（share.yabu.me）と Blossom（任意サーバー、既定プリセット blossom.nostr.build）に対応。

## ソースコード

**ファイル**: `lib/nostr.js` (行 1394-1576)

```javascript
export function getUploadServer() {
  if (typeof window !== 'undefined') {
    return localStorage.getItem('uploadServer') || 'nostr.build'
  }
  return 'nostr.build'
}
export async function uploadImage(file) {
  const server = getUploadServer()
  if (server === 'nostr.build') {
    return uploadToNostrBuild(file)          // POST https://nostr.build/api/v2/upload/files (NIP-98)
  } else if (server === 'share.yabu.me') {
    return uploadToYabuMe(file)              // POST https://share.yabu.me/api/v2/media (NIP-96)
  } else {
    return uploadToBlossom(file, server)     // PUT <server>/upload (Blossom BUD-02, kind:24242)
  }
}
```

## 説明
- TalkTab / PostModal / TimelineTab / HomeTab の投稿 UI とプロフィール画像/バナーから `uploadImage()` を呼び、ローカルファイルをアップロードして URL を得る。
- 既定の送信先は `nostr.build`。ただし NIP-96 標準の `/api/v2/media` ではなく nostr.build 独自の `/api/v2/upload/files` エンドポイント（NIP-98 kind:27235 認証）を叩く custom-HTTP 実装。
- `share.yabu.me` を選ぶと NIP-96、それ以外の値は Blossom サーバー URL とみなし `PUT /upload`（sha256 + kind:24242 認証）を使う。
- 送信先は `MiniAppTab` の「アップロード設定」で変更可能（プリセット: nostr.build / やぶみ(share.yabu.me) / Blossom(blossom.nostr.build)、カスタム Blossom URL 入力可）。設定は `localStorage` の `uploadServer` キー。kind:10096/10063 のリレー公開リストは未対応。

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js#L1394-L1576

---
← [README](../README.md)
