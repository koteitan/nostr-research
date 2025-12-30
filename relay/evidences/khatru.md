[<< back](../README-ja.md) | [Japanese] | [English](khatru-en.md)

# khatru

## 概要
- **言語**: Go
- **種類**: フレームワーク
- **設定**: コードベース、設定ファイルなし
- **リポジトリ**: https://github.com/fiatjaf/khatru
- **確認バージョン**: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)

## limit パラメータ

**デフォルト最大limit**: デフォルト制限なし

**設定パラメータ**: 該当なし

**動作**:
- フレームワークに組み込みの最大limit強制なし
- 開発者は独自のlimitポリシーを実装する必要がある
- フレームワークは `LimitZero` フラグを処理して `limit: 0` のクエリをスキップ
- 実際のlimit強制はKhatruを使用するリレー実装に依存
- レート制限ヘルパーを提供するが、結果制限キャップはなし

**ソースコードの証拠**:
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

## レート制限

### デフォルトレート制限 (`ApplySaneDefaults`経由)

| 項目 | 値 | ソース |
|------|-----|--------|
| 最大サブスクリプション数 | 制限なし | - |
| イベントレート | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L12) (max 10 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |
| フィルターレート | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L17) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |
| 接続レート | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L21) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし |
| 最大過去オフセット | 強制なし |

**備考**: フレームワークはデフォルトでは強制しない

## サイズ制限

| 項目 | 値 |
|------|-----|
| 最大メッセージサイズ | 512,000バイト (500 KB) |

## フレームワーク機能

- カスタムイベント/フィルター受け入れポリシー
- カスタムAUTHハンドラー
- プラガブルストレージバックエンド
- `policies` パッケージの組み込みポリシーヘルパー

---
[<< back](../README-ja.md)
