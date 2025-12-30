[<< back](../README-ja.md) | [Japanese] | [English](strfry-en.md)

# strfry

## 概要
- **言語**: C++
- **設定ファイル**: `strfry.conf`
- **リポジトリ**: https://github.com/hoytech/strfry
- **確認バージョン**: `542552ab0f5234f808c52c21772b34f6f07bec65` (2025-01-10)

## limit パラメータ

**デフォルト最大limit**: [500](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91)

**設定パラメータ**: `relay.maxFilterLimit`

```conf
relay {
    # フィルターごとに返せる最大レコード数
    maxFilterLimit = 500
}
```

**動作**:
- クライアントが `limit: N` でイベントをリクエストした場合、strfryはフィルターごとに **min(N, 500)** 件のイベントを返す
- クライアントがlimitを指定しないか、500より大きいlimitを指定した場合、strfryは500に制限
- これはサブスクリプションごとではなく、フィルターごとの制限

## レート制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大サブスクリプション数 | [20](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L89) | `relay.maxSubsPerConnection` |
| イベント送信レート | デフォルトでは未設定 | 設定可能 |
| フィルター/REQレート | 未設定 | 設定可能 |
| 接続レート | 未設定 | 設定可能 |

**備考**: 全ての制限は設定可能

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+900秒 (15分)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L24) |
| 最大過去オフセット | [-94,608,000秒 (約3年)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L27) |

### イベント保存/削除ポリシー

| 項目 | 値 |
|------|-----|
| 一時イベント経過時間 | [60秒](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L30)より古い場合拒否 |
| 一時イベント生存期間 | [300秒](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L33)後に自動削除 |
| 通常イベント最大経過時間 | - |

**備考**: Kind 20000-29999のみ

## サイズ制限

| 項目 | 値 |
|------|-----|
| イベントサイズ | 65,536バイト (64 KB) - 正規化JSON |
| WebSocketペイロード | 131,072バイト (128 KB) |
| タグ値サイズ | 1,024バイト |
| REQごとの最大フィルター数 | 200 |

## サポートNIP

1, 2, 4, 9, 11, 22, 28, 40, 70, 77

---
[<< back](../README-ja.md)
