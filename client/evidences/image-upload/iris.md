← [README](../README.md)

# iris 画像アップロード先

## 結論
- **アップロード先**: 既定 `blossom.band`（Blossom）。Blossom 3件（blossom.band / blossom.primal.net / blossom.nostr.build）＋ NIP-96 1件（cdn.nostrcheck.me）を同梱。iris サブスクライバは先頭に `upload.iris.to` が加わる。複数サーバーを順に試行し最初に成功したものを使う。

## ソースコード

**ファイル**: `src/pages/settings/mediaservers-utils.ts` (行 3-41)

```typescript
export const MEDIASERVERS = {
  iris:        { url: "https://upload.iris.to",     protocol: "blossom" },
  blossom_band:{ url: "https://blossom.band",       protocol: "blossom" },
  primal:      { url: "https://blossom.primal.net", protocol: "blossom" },
  nostr_build: { url: "https://blossom.nostr.build",protocol: "blossom" },
  nostr_check: { url: "https://cdn.nostrcheck.me",  protocol: "nip96" },
}
export function getDefaultServers(isSubscriber) {
  return isSubscriber
    ? [iris, blossom_band, primal, nostr_build, nostr_check]
    : [blossom_band, primal, nostr_build, nostr_check]
}
```

## 説明
- compose / チャット UI が `useFileUpload` → `processFile` → `uploadFile`（`src/shared/upload.ts`）を呼び、成功 URL と `imeta` タグを生成する。
- 2 プロトコル対応。`server.protocol` が `"blossom"` なら `uploadToBlossom`（`PUT ${url}/upload`, sha256, kind:24242 認証）、`"nip96"` なら `uploadToNip96`（FormData を POST, kind:27235 NIP-98 認証）。既定5件中4件が Blossom、1件が NIP-96。
- 既定先は `mediaservers-utils.ts` にハードコード。非サブスクライバは `blossom.band` を先頭に4件、サブスクライバは `upload.iris.to` を先頭に足した5件。複数サーバーを順に試行する。
- 設定 `src/pages/settings/Mediaservers.tsx` でサーバー追加/削除・プロトコル選択・デフォルト選択・Restore Defaults が可能。設定は zustand の `user-storage`（localStorage）に保存され、kind:10096/10063 の Nostr イベントとしては同期されない。

## 参考
- https://github.com/irislib/iris-client/blob/main/src/pages/settings/mediaservers-utils.ts

---
← [README](../README.md)
