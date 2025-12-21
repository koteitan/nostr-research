# nostter リアクション取得方法

## 結論
- **リアクション取得方法**: 詳細ページを開いた時に購読開始

## ソースコード

**ファイル**: `web/src/routes/(app)/[slug=note]/+page.svelte` (行 138-139)

```typescript
req.emit([
	{
		kinds: [1, 6, 7, 9735],
		'#e': [eventId]
	}
]);
```

## 説明

- 詳細ページ (`/[slug=note]`) を開いた時にリアクションを購読
- `kinds: [1, 6, 7, 9735]` で返信、リポスト、リアクション、Zap を取得
- `#e: [eventId]` で特定のイベントに対するリアクションをフィルタ

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/routes/(app)/[slug=note]/+page.svelte
