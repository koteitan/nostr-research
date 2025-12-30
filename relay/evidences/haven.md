[<< back](../README-ja.md) | [Japanese] | [English](haven-en.md)

# haven

## 概要
- **言語**: Go
- **ベース**: Khatru
- **設定ファイル**: `.env.example`
- **リポジトリ**: https://github.com/bitvora/haven
- **確認バージョン**: `81781b36aaafe5895bb612a452b7c0f44cde6e67` (2025-10-26)

## limit パラメータ

**デフォルト最大limit**: 制限なし

**設定パラメータ**: 未設定

**動作**:
- Khatruフレームワークから動作を継承
- QueryEvents実装に `eventstore` ライブラリを使用
- `limit` 未指定時: **データベースから全てのマッチするイベントを返す**
- **結果サイズのハード制限なし** - 大規模データセットでは潜在的に危険

**ソースコードの証拠**:
```go
// init.go:134
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// github.com/fiatjaf/eventstore v0.17.1を使用
```

## レート制限

### リレータイプごとのレート制限

| リレータイプ | イベント送信レート | 接続レート | ソース |
|------------|-------------------|------------|--------|
| Private | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L61) (max 100 tokens) | [3 conn/5min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L66) | limits.go |
| Chat | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L72) (max 100 tokens) | [3 conn/3min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L77) | limits.go |
| Inbox | [10 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L83) (max 20 tokens) | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L88) | limits.go |
| Outbox | [10 events/60min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L94) (max 100 tokens) | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L99) | limits.go |

**備考**: 最大サブスクリプション数とフィルター/REQレートは個別設定なし

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし |
| 最大過去オフセット | 強制なし |

**備考**: khatruの動作を継承

### タイムアウト設定

| 項目 | 値 |
|------|-----|
| オーナーノートインポートフェッチタイムアウト | 30秒 (デフォルト) |
| タグ付きノートインポートフェッチタイムアウト | 120秒 (デフォルト) |
| Web of Trustフェッチタイムアウト | 30秒 (デフォルト) |

## サイズ制限

| 項目 | 値 |
|------|-----|
| 最大メッセージサイズ | Khatruの最大メッセージサイズを継承: 512,000バイト (500 KB) |

## 特別な機能

- 4つのリレーを1つに (Private, Chat, Inbox, Outbox)
- Blossomメディアサーバー
- Web of Trustフィルタリング
- クラウドバックアップ (S3互換)
- BadgerDBまたはLMDBストレージ

## 警告

レート制限はリクエストレートに適用され、結果サイズには適用されない。limitなしの単一リクエストでも無制限のイベントを返す可能性がある。

---
[<< back](../README-ja.md)
