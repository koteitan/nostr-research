[<< back](../README-ja.md) | [Japanese] | [English](haven-en.md)

# haven

## 概要
- **言語**: Go
- **ベース**: Khatru
- **設定ファイル**: `.env.example`
- **リポジトリ**: https://github.com/bitvora/haven
- **確認バージョン**: `8d26f9e6dfe4f6e43332d30bbf26064675f08559` (2026-06-18)

## limit パラメータ

**デフォルト最大limit**: [制限なし](https://github.com/bitvora/haven/blob/8d26f9e/init.go#L164) (khatru継承)

**設定パラメータ**: 未設定

**動作**:
- haven自体は `limit` パラメータの独自デフォルト上限を定義していない
- 各リレー(Private/Chat/Inbox/Outbox)のQueryEventsは `eventstore` ライブラリ(github.com/fiatjaf/eventstore v0.17.5)のQueryEventsをそのまま登録し、Khatruフレームワークの動作を継承
- `limit` 未指定時: **データベースから全てのマッチするイベントを返す可能性がある**
- **結果サイズのハード制限なし** - 大規模データセットでは潜在的に高負荷

**ソースコードの証拠**:
```go
// init.go:164
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// github.com/fiatjaf/eventstore v0.17.5 を使用
```

## レート制限

### リレータイプごとのレート制限

havenは4つのリレータイプ(Private/Chat/Inbox/Outbox)を単一バイナリで提供し、それぞれが独立したイベント送信レート(`EventIPRateLimiter`)と接続レート(`ConnectionRateLimiter`)を持つ。これらのレートはリクエスト頻度を制限するものであり、1リクエストが返す結果サイズには適用されない。最大サブスクリプション数とフィルター/REQレートはhaven側で個別に設定されておらず、Khatruの動作を継承する。トークンバケット方式で、interval(分)ごとにtokensトークンを補充し、上限はmaxTokens。接続レートのmaxTokensは全タイプで9。

| リレータイプ | 最大サブスクリプション数 | イベント送信レート | フィルター/REQレート | 接続レート | 備考 | ソース |
|------------|----------------------|-------------------|--------------------|------------|------|--------|
| Private | Khatru継承 | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L61) (max 100 tokens) | Khatru継承 | [3 conn/5min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L66) (max 9 tokens) | 認証 + ホワイトリスト必須リレー | limits.go |
| Chat | Khatru継承 | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L72) (max 100 tokens) | Khatru継承 | [3 conn/3min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L77) (max 9 tokens) | WoT内ユーザー向けチャット | limits.go |
| Inbox | Khatru継承 | [10 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L83) (max 20 tokens) | Khatru継承 | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L88) (max 9 tokens) | インボックスリレー | limits.go |
| Outbox | Khatru継承 | [10 events/60min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L94) (max 100 tokens) | Khatru継承 | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L99) (max 9 tokens) | 公開メッセージ/メディア配信用 | limits.go |

**備考**: 最大サブスクリプション数とフィルター/REQレートはhaven側で個別設定なし(Khatru継承)。

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし (khatru継承) |
| 最大過去オフセット | 強制なし (khatru継承) |

**備考**: haven自体はcreated_atの未来/過去オフセット検証を実装しておらず、khatruの動作を継承するためデフォルトでは時刻検証は行われない。なお、これらは保存/配信ポリシーであり、インポート/WoTフェッチには独自の独立したタイムアウトがある(下記参照)。

### タイムアウト設定

| 項目 | 値 |
|------|-----|
| オーナーノートインポートフェッチタイムアウト | 60秒 (デフォルト、IMPORT_OWNER_NOTES_FETCH_TIMEOUT_SECONDS) |
| タグ付きノートインポートフェッチタイムアウト | 120秒 (デフォルト、IMPORT_TAGGED_NOTES_FETCH_TIMEOUT_SECONDS) |
| Web of Trustフェッチタイムアウト | 30秒 (デフォルト、WOT_FETCH_TIMEOUT_SECONDS) |

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | 制限なし (khatru継承) | - |
| REQあたり最大フィルター数 | 制限なし (khatru継承) | - |
| 最大authors数 (概算) | ~7,400 (WebSocketメッセージサイズ制限による) | - |

**備考**: havenはフィルター値数・REQあたりフィルター数の上限を独自設定していない。khatruのMaxMessageSize(512,000バイト)が実質的な上限となり、authorsに換算すると概算で約7,400件。なおAllowComplexFilters/AllowEmptyFiltersはリレータイプ単位で制御され、Chat/Inbox/Outboxでは空フィルター・複雑フィルターを拒否する。

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大メッセージサイズ | [512,000バイト (500 KB)](https://github.com/fiatjaf/khatru/blob/v0.19.1/relay.go) (khatru継承) | khatru MaxMessageSize |

## サポートNIP

明示的なSupportedNIPsリストは未設定 (khatru/NIP-11デフォルトに依存)

## 特別な機能

- 4つのリレーを1つに (Private, Chat, Inbox, Outbox)
- Blossomメディアサーバー
- Web of Trustフィルタリング
- クラウドバックアップ (S3互換)
- BadgerDBまたはLMDBストレージ (LMDBのデフォルトマップサイズは約273GB)

## 警告

レート制限はリクエストレートに適用され、結果サイズには適用されない。limitなしの単一リクエストでも無制限のイベントを返す可能性がある。

---
[<< back](../README-ja.md)
