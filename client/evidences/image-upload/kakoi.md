← [README](../README.md)

# kakoi 画像アップロード先

## 結論
- **アップロード先**: アプリ内アップロードは無し。「画像」ボタンは外部ブラウザで NIP-96 学習用 Web ツール（既定 `https://nikolat.github.io/nostr-learn-nip96/`）を開くだけで、kakoi 自身はメディアサーバーへ送信しない。

## ソースコード

**ファイル**: `kakoi/FormPostBar.cs` (行 127-135), `kakoi/Setting.cs` (行 57)

```csharp
private void ButtonPicture_Click(object sender, EventArgs e)
{
    var app = new ProcessStartInfo
    {
        FileName = Setting.PictureUploadUrl,
        UseShellExecute = true
    };
    Process.Start(app);
}
// Setting.cs:57
public string PictureUploadUrl { get; set; } = "https://nikolat.github.io/nostr-learn-nip96/";
```

## 説明
- kakoi（C#/.NET WinForms デスクトップ）の投稿バーには「画像」ボタンがあるが、ローカルファイルをメディアサーバーへ直接アップロードする機能は無い。ボタンは `Process.Start`（`UseShellExecute=true`）で `Setting.PictureUploadUrl` を既定ブラウザで開くだけ。ユーザーはその外部 Web ページで画像をアップロードし、得た URL を手動で本文に貼り付ける運用。
- 既定 URL は `https://nikolat.github.io/nostr-learn-nip96/`（NIP-96 アップロード学習用 Web アプリ）。実質の転送は NIP-96 だが、それを行うのは外部 Web ツール側であり kakoi のコードではない。
- ソース全体に NIP-96/Blossom のクライアント実装や multipart/PutAsync/PostAsync 等のアップロードコードは無い（唯一の HttpClient 利用はアバター画像のダウンロード専用）。開く URL は `kakoi.config` で変更可能（GUI 設定画面には該当欄は無い）。

## 参考
- https://github.com/betonetojp/kakoi/blob/master/kakoi/FormPostBar.cs#L127-L135

---
← [README](../README.md)
