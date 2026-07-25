[<< back](../README-ja.md) | [Japanese] | [English](wot-relay-en.md)

# wot-relay

## 概要
- **言語**: Go
- **ベース**: Khatru
- **設定ファイル**: `.env.example`
- **リポジトリ**: https://github.com/bitvora/wot-relay
- **確認バージョン**: `7c5803ff3e765d2b553bce24d8bc2d0a0717fee6` (2026-04-22)

## limit パラメータ

**デフォルト最大limit**: [500](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L164)

**設定パラメータ**: UseEventstore 第2引数 (maxQueryLimit)

**動作**:
- 前回 (24b51de9) は明示的なlimit設定がなく『制限なし』だったが、本バージョンでは `relay.UseEventstore(db, 500)` を呼び出すことで khatru の maxQueryLimit が 500 に設定されている。
- khatru の QueryStored はこの maxQueryLimit をストアの QueryEvents に渡し、結果サイズの上限として強制する。
- ネゲントロピー (NIP-77) セッションの場合のみ maxQueryLimit×20 = 10,000 に拡大されるが、wot-relay は Negentropy を有効化していないため通常は 500 が上限。
- eventstore バックエンドは BadgerDB から LMDB (`fiatjaf.com/nostr/eventstore/lmdb`) に変更された。

**ソースコードの証拠**:
```go
relay.UseEventstore(db, 500)  // main.go:164
// khatru: maxLimit := maxQueryLimit (=500); negentropy時は ×20
```

## レート制限

| 項目 | 値 | ソース |
|------|-----|--------|
| 最大サブスクリプション数 | 制限なし | - |
| イベント送信レート | [5 events/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L168) (max 30 tokens) | main.go |
| フィルターレート | [5 filters/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L202) (max 30 tokens) | main.go |
| 接続レート | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L205) (max 30 tokens) | main.go |

**備考**: Khatruベース。khatruデフォルトより厳格

**ソースコード**:
```go
policies.EventIPRateLimiter(5, time.Minute*1, 30)      // 5 events/min, max 30 tokens
policies.FilterIPRateLimiter(5, time.Minute*1, 30)     // 5 filters/min, max 30 tokens
policies.ConnectionRateLimiter(10, time.Minute*2, 30)  // 10 connections/2min, max 30 tokens
```

**詳細**: khatru の policies パッケージのレートトークンバケットを使用。値は前バージョンから変更なし (5 events/min, 5 filters/min, 10 conn/2min, いずれも max 30 tokens)。最大サブスクリプション数は明示的に設定されていない (khatru継承、制限なし)。OnEvent には base64メディア拒否、WoT外拒否、暗号化DM (kind 4) 拒否も追加されている。OnRequest には NoEmptyFilters / NoComplexFilters (タグ2個超かつ要素4個超を拒否) が追加されている。

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | 強制なし |
| 最大過去オフセット | 強制なし |

**備考**: created_at の未来・過去オフセット検証は khatru の PreventTimestampsInTheFuture/Past を OnEvent に登録していないため強制されない (khatru継承=強制なし)。

### イベント保存/削除ポリシー

| 項目 | 値 |
|------|-----|
| 通常イベント最大経過時間 | [365日](https://github.com/bitvora/wot-relay/blob/7c5803f/.env.example#L27)後に削除 (ただしコードのデフォルトは [0=無効](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L283)) |

**備考**: 経過時間ベースの削除は MAX_AGE_DAYS で設定。.env.example の例では 365 だが、環境変数が未設定の場合のコード上のデフォルトは 0 (削除無効)。MAX_AGE_DAYS > 0 のとき ARCHIVE_KINDS 該当 kind のみが対象。

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | 制限なし | - |
| REQあたり最大フィルター数 | 制限なし | - |
| 最大authors数 (概算) | ~7,400 (WebSocketメッセージサイズ制限による) | - |

**備考**: khatru継承。フィルター値そのものの個数制限はないが、NoComplexFilters によりタグ2個超かつ (タグ+kind) 4個超のフィルターは拒否される。authors 数は最大メッセージサイズ 512,000 バイトが実質的上限 (1 pubkey ≈ 69 バイト → 約7,400)。

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大メッセージサイズ | [512,000バイト (500 KB)](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L140) | MaxMessageSize (khatru継承) |

**備考**: 最大メッセージサイズはkhatruのデフォルト 512,000 バイト (500 KB) を継承。wot-relay 側で上書きしていない (khatru NewRelay の MaxMessageSize: 512000)。

## 設定オプション

```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

## 特別な機能

- Web of Trustリレー (フォローしているユーザーのノートのみ保存)
- 設定可能なWoT深度と最小フォロワー数 (MINIMUM_FOLLOWERS、MAX_TRUST_NETWORK=40000、MAX_ONE_HOP_NETWORK=50000)
- オプションの他リレーからのアーカイブ同期 (ARCHIVAL_SYNC)、リアクションのアーカイブ可否 (ARCHIVE_REACTIONS)
- オプションの経過時間ベースのノート削除 (ARCHIVE_KINDS 該当 kind)

## サポートNIP

- NIP-1, 9, 11, 45
- khatru が DeleteEvent 設定で NIP-9、Count 設定で NIP-45 を NIP-11 応答に自動追加。基盤プロトコルとして NIP-1 / NIP-11 をサポート。Negentropy 無効のため NIP-77 は非対応。

## 警告

レート制限はリクエストのレートに適用され、結果サイズには適用されない。limit を指定しない単一リクエストでも maxQueryLimit の 500 イベントまで返される (Negentropy セッションのみ 10,000 だが、ここでは有効化されていない)。

---
[<< back](../README-ja.md)
