[<< back](../README-ja.md) | [Japanese] | [English](wot-relay-en.md)

# wot-relay

## 概要
- **言語**: Go
- **ベース**: Khatru
- **設定ファイル**: `.env.example`
- **リポジトリ**: https://github.com/bitvora/wot-relay
- **確認バージョン**: `24b51de9fdd8631f4795be85983666e76f220960` (2025-06-16)

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
// main.go:145
relay.QueryEvents = append(relay.QueryEvents, db.QueryEvents)
// github.com/fiatjaf/eventstoreを使用
```

## レート制限

| 項目 | 値 | ソース |
|------|-----|--------|
| 最大サブスクリプション数 | 制限なし | - |
| イベント送信レート | [5 events/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L131) (max 30 tokens) | main.go |
| フィルターレート | [5 filters/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L137) (max 30 tokens) | main.go |
| 接続レート | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L141) (max 30 tokens) | main.go |

**備考**: Khatruベース。khatruデフォルトより厳格

**ソースコード**:
```go
policies.EventIPRateLimiter(5, time.Minute*1, 30)      // 5 events/min, max 30 tokens
policies.FilterIPRateLimiter(5, time.Minute*1, 30)     // 5 filters/min, max 30 tokens
policies.ConnectionRateLimiter(10, time.Minute*2, 30)  // 10 connections/2min, max 30 tokens
```

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし |
| 最大過去オフセット | 強制なし |

**備考**: khatruの動作を継承

### イベント保存/削除ポリシー

| 項目 | 値 |
|------|-----|
| 通常イベント最大経過時間 | [365日](https://github.com/bitvora/wot-relay/blob/24b51de9/.env.example#L27)後に削除 |

**備考**: MAX_AGE_DAYSで設定可能

## サイズ制限

| 項目 | 値 |
|------|-----|
| 最大メッセージサイズ | Khatruの最大メッセージサイズを継承: 512,000バイト (500 KB) |

## 設定オプション

```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

## 特別な機能

- Web of Trustリレー (フォローしているユーザーのノートのみ保存)
- 設定可能なWoT深度と最小フォロワー数
- オプションの他リレーからのアーカイブ同期
- オプションの経過時間ベースのノート削除

## 警告

レート制限はリクエストレートに適用され、結果サイズには適用されない。limitなしの単一リクエストでも無制限のイベントを返す可能性がある。

---
[<< back](../README-ja.md)
