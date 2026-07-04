← [README](../README.md)

# nostter 画像アップロード先

## 結論
- **アップロード先**: 既定 `nostrcheck.me`（NIP-96）。同梱の選択肢は nostrcheck.me / nostr.build / files.sovbit.host / nostpic.com / yabu.me の5サーバー（すべて NIP-96）。

## ソースコード

**ファイル**: `web/src/lib/Constants.ts` (行 143-149), `web/src/lib/Media.ts` (行 14-17)

```typescript
// Constants.ts:143-149
export const fileStorageServers = [
	'https://nostrcheck.me', 'https://nostr.build', 'https://files.sovbit.host',
	'https://nostpic.com', 'https://yabu.me'
];
// Media.ts:14-17 — 既定は fileStorageServers[0] = nostrcheck.me
export function getMediaUploader(): string {
	return get(preferencesStore).mediaUploader ?? fileStorageServers[0];
}
```

## 説明
- NoteEditor / ChannelComposer / プロフィール編集の各 UI から `uploadFiles()` を呼び、ドラッグ&ドロップ・ペースト・ファイル選択でローカルファイルをアップロードする。
- プロトコルは NIP-96。`FileStorageServer.upload()` が `/.well-known/nostr/nip96.json` を fetch して `api_url` を取得し、NIP-98（kind:27235）署名イベントを `Authorization: Nostr <base64>` ヘッダに付けて POST する。Blossom は未使用。
- 既定のアップロード先は `getMediaUploader()` が返す `fileStorageServers[0]` = `nostrcheck.me`。
- 設定 `preferences/MediaUploader.svelte` のドロップダウンで同梱5サーバーから選択でき、選択値は `preferencesStore.mediaUploader`（アプリローカル）に保存される。kind:10096/10063 のイベント連動は無い。
- なお `NostrcheckMe.ts`（独自 API）と `VoidCat.ts`（void.cat）のクラスも存在するが、現在どこからも参照されておらず実アップロード経路では未使用（レガシー）。

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/media/FileStorageServer.ts

---
← [README](../README.md)
