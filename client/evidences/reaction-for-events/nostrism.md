← [README](../README.md)

# Nostrism リアクション取得方法

## 結論
- **通常タイムラインではリアクション数を集計しない**（フォロー中/著者フィードは kind:1/6/16 のみ購読し、kind:7 は購読しない）。
- **パブリックチャット（kind:42）のみ**、表示中メッセージ群へ `#e` で kind:7 をバッチ購読（最大300 id, limit 500）し、Slack 風に集約表示する。
- 通知カラムは自分宛（`#p`）の kind:1/6/16/7/9735 を購読。自分の kind:7 はタイムライン混在用に別途購読する。

## ソースコード

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt` (行 904, 1172-1176, 1213, 315)

```kotlin
// フォロー中カラム（行 904）: リアクション数は出さないので kind:7 は購読しない
subscribeAll(columnId, Filter(kinds = listOf(1, 6, 16), authors = withMe, limit = 100))

// パブリックチャット: 表示中メッセージ群への kind:7 を購読（Slack 風集約表示）。id 群が変わるたび貼り直す（行 1172-1176）
fun subscribeChannelReactions(subId: String, messageIds: List<String>) {
    if (messageIds.isEmpty()) return
    openColumns.add(subId)
    subscribeAll(subId, Filter(kinds = listOf(7), eTags = messageIds.take(300), limit = 500))
}

// 通知カラム（行 1213）: 自分宛（#p=自分）
subscribeAll(columnId, Filter(kinds = listOf(1, 6, 16, 7, 9735), pTags = listOf(me), limit = 200))

// 自分のリアクション（行 315）: TL 混在用
subscribeAll("myreactions", Filter(kinds = listOf(7), authors = listOf(me), limit = 100))
```

## 説明
- コード冒頭に「集計クエリ(reactionsForTargets/engagementForTargets)は購読/DBを無駄に使うため使用しない」とあり、フォロー中タイムラインではリアクション数バッジを表示しない設計。行 904 のコメントも「リアクション数は出さないので kind:7 は購読しない」と明記する。スレッド購読 (`subscribeThread`) も kind:1 の返信のみで kind:7 は含めない。
- **パブリックチャット** (`ChannelRoomColumn` → `subscribeChannelReactions`): 表示中の kind:42 メッセージ id 群（最大300件）に対し `#e` で kind:7 をまとめて購読し（limit 500）、`ReactionRow` で絵文字ごとの件数チップとして集約表示する。表示 id が変わるたび REQ を貼り直す。
- **通知**: 自分宛（`#p` = 自分）の kind:1/6/16/7/9735 を購読し、リアクション/リポスト/リプライ/メンション/Zap として表示する。
- **自分のリアクション**: kind:7（authors=自分, limit 100）を購読し、♡状態の反映と自分のリアクション表示に使う。
- Zap（kind:9735）はノート id 群へ `#e` で別購読（最大300 id, limit 500）。

## 参考
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt

---
← [README](../README.md)
