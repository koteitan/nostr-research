← [README](../README.md)

# 野雨 画像アップロード先

## 結論
- **アップロード先**: なし。画像/メディアのアップロード機能は未実装。

## ソースコード

**ファイル**: `index.html` (行 109-113)

```html
<button id="btnPublish" type="button" aria-label="投稿する">投稿</button>
</div>
<div class="composer">
  <textarea id="compose" placeholder="本文を入力......" aria-label="投稿本文"></textarea>
</div>
```

## 説明
- 野雨は縦書き短文専用クライアントで、投稿フォームは `<textarea id="compose">` のテキスト入力のみ。`<input type="file">` やドラッグ&ドロップ等の添付 UI は存在しない。
- アップロード関連コード（FileReader / FormData / multipart / NIP-96 / Blossom / api/v2/media 等）はリポジトリ全体に無い。
- 投稿処理 `publish(content)`（`nostr-core.js`）は文字列をそのまま kind:1 に載せて NIP-07 署名するだけ。本文中の URL 貼り付けはできるが、ローカルファイルのアップロード機能は無い（本文は最大108文字の短文専用）。

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/index.html

---
← [README](../README.md)
