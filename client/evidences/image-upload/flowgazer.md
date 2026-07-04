← [README](../README.md)

# flowgazer 画像アップロード先

## 結論
- **アップロード先**: 外部アプリ **ehagaki** への委譲（`https://lokuyow.github.io/ehagaki/` を iframe 埋め込み）。flowgazer 自身は NIP-96/Blossom を実装せず、同梱のメディアサーバーも持たない。実際のアップロード先・プロトコルは ehagaki 側（別オリジン）で決まる。

## ソースコード

**ファイル**: `kit-ten.html` (行 433-435, 646-663)

```html
<iframe id="ehagaki-iframe"
    src="https://lokuyow.github.io/ehagaki/?parentOrigin=https://ompomz.github.io/flowgazer/kit-ten.html"
    allow="local-network-access; local-network; loopback-network" frameborder="0" width="600" height="400">
```
```javascript
const STORAGE_PREFIX = 'ehagaki.embed.storage.v1:';
const ALLOWED_KEYS = new Set([ /* ... */ 'uploadEndpoint', /* ... */ ]);
```

## 説明
- flowgazer は独自の画像アップロード実装を持たず、画像投稿アプリ「ehagaki」（作者 Lokuyow）を iframe として埋め込み、アップロード機能を丸ごと委譲している（kit-ten.html:433-435）。実際のメディアサーバーへの送信・プロトコル・既定プロバイダは ehagaki 側（別オリジン）にあり、flowgazer リポジトリのコードには含まれない。
- flowgazer の役割は postMessage ブリッジのみ。署名（`window.nostrAuth.signEvent`）、pubkey の受け渡し、localStorage 委譲を行う。nostr.build / blossom.band 等の実ホスト名はハードコードしていない。
- アップロード先エンドポイントは委譲された設定として `ehagaki.embed.storage.v1:uploadEndpoint` キーで localStorage に永続化される（設定 UI・既定値の決定は ehagaki 内）。
- 本体の `index.html` の投稿 UI は textarea による kind:1 プレーンテキスト投稿のみ。機能する埋め込み（iframe + ehagakiManager JS）は `kit-ten.html` にある。

## 参考
- https://github.com/ompomz/flowgazer/blob/main/kit-ten.html

---
← [README](../README.md)
