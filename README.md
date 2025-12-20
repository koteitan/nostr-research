# Nostr Client Research
クライアントの実装の違いを研究するためのリポジトリです。
npub1f3w4x7dqvceeez8kuyq78md3lwhwfm0ra634llr0r3nykwjrs0hqvldhgk にこの実装の違いをレポートしてほしいと連絡を頂けると調査して掲載します。PR で追加してくれるのも歓迎です。

# Boot strap リレー
nostr が購読するリレーの決め方は、kind:10002 を使うもの、outbox モデルなど、さまざまがあり、それらの情報の多くはリレーのメタデータに含まれています。そのためには一度どこかのリレーに接続してメタデータを取得する必要があります。各クライアントがどのように Boot strap リレーを決定しているかを調査しました。

(表をここに挿入。(行=クライアント, 列=Boot strap リレーの決定方法))

# リレー
各クライアントがホームタイムラインを作るためのリレーをどこから取得しているかを調査しました。

(表をここに挿入。(行=クライアント, 列=リレーの取得方法))

# リアクションの取得方法
イベントについているリアクションの収集方法, クローリング方法を調査しました。

(表をここに挿入。(行=クライアント, 列=リアクションの取得方法. イベントがくる毎にバッチ取得のキューに入れる, 定期的にバッチ取得する, リアルタイムで購読する, など))

# 調査済みクライアント一覧
- web:
  - [nostter](https://nostter.app/)
  - [Rabbit](https://rabbit.syusui.net/)
  - [Lumilumi](https://lumilumi.app/)
  - [Yakihonne](https://yakihonne.com/)
  - [iris](https://iris.to/)
  - [Primal](https://primal.net/)
  - [Coracle](https://coracle.social/)
- mobile:
  - [Amethyst](https://play.google.com/store/apps/details?id=com.vitorpamplona.amethyst)
  - [Damus](https://damus.io/)

# 参考文献
- クライアント
  - nostter: https://github.com/SnowCait/nostter
  - Rabbit: https://github.com/syusui-s/rabbit
  - Lumilumi: https://github.com/TsukemonoGit/lumilumi
  - Yakihonne: https://github.com/YakiHonne
  - iris: https://github.com/irislib/iris-client
  - Primal: https://github.com/PrimalHQ/primal-web-app
  - Coracle: https://github.com/coracle-social/coracle
  - Amethyst: https://github.com/vitorpamplona/amethyst
  - Damus: https://github.com/damus-io/damus

