← [README](../README.md)

# Nos Haiku 画像アップロード先

## 結論
- **アップロード先**: 既定 `yabu.me`（NIP-96）。NIP-96 と Blossom の両対応。同梱 NIP-96 5件（yabu.me / nostpic.com / nostr.build / nostrcheck.me / files.sovbit.host）、Blossom 5件（blossom.band / cdn.nostrcheck.me / nostr.download / blossom.primal.net / cdn.satellite.earth）。

## ソースコード

**ファイル**: `src/lib/config.ts` (行 44-57), `src/lib/store.ts` (行 37-38)

```typescript
export const defaultUploaderURLsNip96 = [
	'https://yabu.me', 'https://nostpic.com', 'https://nostr.build',
	'https://nostrcheck.me', 'https://files.sovbit.host'
];
export const defaultUploaderURLsBlossom = [
	'https://blossom.band', 'https://cdn.nostrcheck.me', 'https://nostr.download',
	'https://blossom.primal.net', 'https://cdn.satellite.earth'
];
// store.ts:37-38 — 既定は uploaderSelected = defaultUploaderURLsNip96[0] (=yabu.me), uploaderType='nip96'
```

## 説明
- `CreateEntry.svelte` の file input（`onchange={uploadFileExec}`, 660行目）がローカルファイルをアップロードし、返った URL を本文に挿入する。
- 2 プロトコル対応。`uploadFileExec`（CreateEntry.svelte:134-155）が `uploaderType` で分岐。`'nip96'` は `uploadByNip96`（`readServerConfig` + `uploadFile`, NIP-98 認証）、`'blossom'` は `nostr-tools/nipb7` の `BlossomClient.uploadFile()`（BUD-02, `PUT /upload`）。動画/音声は mediabunny で mp4 に圧縮してから送信。
- 既定送信先は `defaultUploaderURLsNip96[0]` = `yabu.me`。
- 設定 `Settings.svelte` で NIP-96/Blossom のリストから選択でき、リストはユーザーの kind:10096（NIP-96 File Server Preferences）/ kind:10063（Blossom server list）があればそれを優先、無ければ config の既定値にフォールバックする。

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/config.ts

---
← [README](../README.md)
