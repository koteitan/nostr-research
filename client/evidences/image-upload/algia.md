← [README](../README.md)

# algia 画像アップロード先

## 結論
- **アップロード先**: 同梱のデフォルトは無く**要設定**（未設定/未指定だとエラー）。設定 `file-servers` か `--server` フラグで URL を指定。Blossom と NIP-96 の両対応（プレフィックス無しの URL は Blossom 扱い）。

## ソースコード

**ファイル**: `file.go` (行 44-118)

```go
// parseFileServer ... defaulting to Blossom.
func parseFileServer(s string) fileServer {
	switch {
	case strings.HasPrefix(s, "nip96+"):
		return fileServer{URL: strings.TrimPrefix(s, "nip96+"), Type: typeNIP96}
	case strings.HasPrefix(s, "blossom+"):
		return fileServer{URL: strings.TrimPrefix(s, "blossom+"), Type: typeBlossom}
	default:
		return fileServer{URL: s, Type: typeBlossom}
	}
}
func fileServers(cCtx *cli.Context, cfg *Config) ([]fileServer, error) {
	if flags := cCtx.StringSlice("server"); len(flags) > 0 { ... }
	if len(cfg.FileServers) == 0 {
		return nil, errors.New("no media server configured; set \"file-servers\" in config or pass --server")
	}
	return cfg.FileServers, nil
}
```

## 説明
- algia は CLI クライアントで、`algia file upload ./photo.png` によりローカルファイルをアップロードして Blob URL を得る（`doFileUpload`, `file` サブコマンド）。GUI 添付ではなくコマンド駆動。
- プロトコルは Blossom（BUD-01/02: `PUT /upload`, sha256, `Nostr <base64(event)>` 認証）と NIP-96（`/.well-known/nostr/nip96.json` の `api_url` へ multipart POST）の両対応。go-nostr の `nipb0/blossom` を利用。
- アップロード先はコードに同梱デフォルトが無く、設定 `file-servers`（`Config.FileServers`）に URL を書くか `--server/-s` で指定する必要がある。未設定・未指定だと "no media server configured" エラー。
- URL 単体は Blossom 扱い、`nip96+`/`blossom+` プレフィックスや `{"url":...,"type":...}` でプロトコルを選択できる。kind:10063/10096 のサーバーリスト参照は無く、設定はローカル config（JSON）のみ。README のサンプル設定に `https://blossom.band` と `https://nostrcheck.me` が例示されているが、これは説明用でありコード上の既定値ではない。

## 参考
- https://github.com/mattn/algia/blob/main/file.go

---
← [README](../README.md)
