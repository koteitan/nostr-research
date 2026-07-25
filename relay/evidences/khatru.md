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

フレームワーク。デフォルトのレート制限は [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) を適用したときに有効になる。最大サブスクリプション数はフレームワークレベルでは強制されない。

| 項目 | 値 | ソース |
|------|-----|--------|
| 最大サブスクリプション数 | 制限なし | - |
| イベントレート | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L12) (max 10 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |
| フィルターレート | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L17) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |
| 接続レート | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L21) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし |
| 最大過去オフセット | 強制なし |
| Ephemeral イベント経過時間 | - |
| Ephemeral イベント寿命 | - |
| 通常イベント最大経過時間 | - |

**備考**: フレームワークはデフォルトでは `created_at` を検証しない。`policies` パッケージは [`PreventTimestampsInTheFuture`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/events.go#L98) / [`PreventTimestampsInThePast`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/events.go#L88) ヘルパーを提供するが、`ApplySaneDefaults` では適用されない。イベント保存/削除ポリシーは実装依存。

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | 制限なし | - |
| REQあたり最大フィルター数 | 制限なし | - |
| 最大authors数 (概算) | ~7,400 (WebSocket制限による) | - |

**備考**: フレームワークにデフォルトのフィルター値制限なし。メッセージサイズ制限 ([512,000バイト (500 KB)](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45)) が実質的な上限となる。

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大メッセージサイズ | [512,000バイト (500 KB)](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45) | `MaxMessageSize` |

## サポートNIP

該当なし (フレームワークのため固有のNIPリストなし)

## フレームワーク機能

- カスタムイベント/フィルター受け入れポリシー
- カスタムAUTHハンドラー
- プラガブルストレージバックエンド
- `policies` パッケージの組み込みポリシーヘルパー

---
[<< back](../README-ja.md)
