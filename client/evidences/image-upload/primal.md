← [README](../README.md)

# Primal 画像アップロード先

## 結論
- **アップロード先**: 既定 `blossom.primal.net`（Blossom）。ユーザーがサーバー未設定なら `primalBlossom` にフォールバック。追加サーバーへはミラーリングする。

## ソースコード

**ファイル**: `src/components/Uploader/UploaderBlossom.tsx` (行 51-53, 178-254), `src/constants.ts` (行 549)

```typescript
// UploaderBlossom.tsx
import { BlossomClient } from "blossom-client-sdk";
const mainServer = () => accountStore.blossomServers[0] || primalBlossom;   // L51-53
const auth = await BlossomClient.createUploadAuth(signEvent, file, { message: 'media upload' });
const uploadUrl = url.endsWith('/') ? `${url}upload` : `${url}/upload`;
xhr.open('PUT', uploadUrl, true);   // Blossom PUT /upload
// constants.ts:549
export const primalBlossom = 'https://blossom.primal.net';
```

## 説明
- 投稿作成ボックス `EditBox.tsx` が現行アップローダー `<UploaderBlossom>` を使用。旧来の `<Uploader>`（Primal 独自 WebSocket チャンク方式）は import されておらず死にコード。ReadsEditor / EditProfile / CreateAccount 等も同じ `UploaderBlossom` を使う。
- プロトコルは Blossom。`blossom-client-sdk` で `BlossomClient.createUploadAuth`（kind:24242 認可イベント）を作り、`X-SHA-256` ヘッダ付きで `PUT /upload`（先に HEAD で `/media` を試し、失敗時 `/upload`）へ送信。追加サーバーには `BlossomClient.mirrorBlob` でミラー。NIP-96 は不使用。
- 既定先は `primalBlossom` = `blossom.primal.net`。ユーザー未設定時に `mainServer()` がこの値へフォールバック。primal 既定サーバーへのアップロード時のみ会員ティア別サイズ上限を適用（regular 100MB / premium 1GB / legend 10GB）。
- 設定 `Settings/Blossom.tsx` でメインサーバー+ミラーを追加/削除でき、`accountStore.blossomServers` を kind:10063 Blossom サーバーリストとして送信/取得する。

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/components/Uploader/UploaderBlossom.tsx

---
← [README](../README.md)
