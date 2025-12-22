← [README](../README.md)

# Amethyst リアクション取得方法の調査結果

調査日: 2025年12月21日
リポジトリ: https://github.com/vitorpamplona/amethyst

## 調査概要

Amethyst（NostrプロトコルのAndroidクライアント）におけるリアクション（kind:7）の取得・表示方法について調査しました。

## 1. タイムラインにリアクション数を表示しているか

**結論: YES - タイムラインに表示している**

### 実装の詳細

`ReactionsRow.kt`コンポーネントがノートの下部にリアクション情報を表示します。

**表示される情報:**
- 返信数（ReplyCounter）
- ブースト数（BoostText）
- リアクション数（ObserveLikeText）
- ザップ額（ObserveZapAmountText）

**表示タイミング:**
```kotlin
// NoteCompose.kt内
if (isNotRepost) {
    if (makeItShort) {
        Spacer(modifier = DoubleVertSpacer)
    } else {
        ReactionsRow(
            baseNote = baseNote,
            showReactionDetail = notBoostedNorQuote,
            addPadding = !isBoostedNote,
            editState = editState,
            accountViewModel = accountViewModel,
            nav = nav,
        )
    }
}
```

ノート本体の下部に配置され、`makeItShort`がfalseの場合に表示されます。

### UI実装の特徴

**数値フォーマット:**
- 大きな数値は短縮形（G/M/k）に変換
- `TextCount`関数で整形

**アニメーション:**
- パフォーマンスモードが無効の場合、`AnimatedContent`でスライドアニメーション付き数値更新

**関連ファイル:**
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/ui/note/ReactionsRow.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/ui/note/NoteCompose.kt`

## 2. リアクション（kind:7）の取得方法

**結論: イベント単位のバッチ取得 + リアルタイム購読**

### アーキテクチャ概要

Amethystは3層アーキテクチャを採用：
1. **UIレイヤー**: State/ViewModel/Composition
2. **サービスレイヤー**: Nostrリレーとの接続
3. **モデル/リポジトリレイヤー**: メモリ内のOOグラフ、LiveData/Flowでのイベント配信

### データフロー

```
UI (ReactionsRow)
  ↓ observeNoteReactions()
EventObservers.kt
  ↓ EventFinderFilterAssemblerSubscription
EventFinderFilterAssembler
  ↓ EventWatcherSubAssembler
FilterRepliesAndReactionsToNotes
  ↓ リレーへのREQコマンド
Nostrリレー
  ↓ イベント受信
LocalCache
  ↓ Note.addReaction()
NoteFlowSet.reactions.invalidateData()
  ↓ Flowの更新
UI更新
```

### リアクション監視の実装

**EventObservers.kt** (`/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/EventObservers.kt`)

```kotlin
@Composable
fun observeNoteReactions(note: Note, accountViewModel: AccountViewModel): State<Map<String, List<Note>>?> {
    val subscription = EventFinderFilterAssemblerSubscription(note, accountViewModel)

    return remember(note) { LocalCache.getNoteIfExists(note.idHex) }
        ?.flow()
        ?.reactions
        ?.stateFlow
        ?.collectAsStateWithLifecycle()
        ?: remember { mutableStateOf(null) }
}
```

**特徴:**
- `@Composable`関数として実装
- `EventFinderFilterAssemblerSubscription`で中継購読
- ローカルキャッシュからのFlowを監視
- `collectAsStateWithLifecycle()`で状態化

### バッチ取得の仕組み

**FilterRepliesAndReactionsToNotes.kt** (`/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/FilterRepliesAndReactionsToNotes.kt`)

**対象イベント種別:**
```kotlin
RepliesAndReactionsKinds = listOf(
    TextNoteEvent.KIND,         // 1 (通常ノート)
    ReactionEvent.KIND,         // 7 (リアクション) ★
    RepostEvent.KIND,           // 6 (リポスト)
    GenericRepostEvent.KIND,    // 16 (汎用リポスト)
    // ...その他
)
```

**フィルタ生成ロジック:**
```kotlin
fun filterRepliesAndReactionsToNotes(
    notes: Set<Note>,
    account: User,
    relayInfo: RelayPresence,
): List<TypedFilter>
```

**バッチ取得の特徴:**
- リレーごとにノートIDを集約（`mapOfSet`使用）
- 1つのリレーに対して3つの異なるフィルタを生成
- レート制限: 各フィルタに`limit`を設定（1000または100件）
- `since`パラメータで前回同期以降のイベントのみ取得
- 複数ノートのリアクションをまとめて取得

### EOSE（End of Stored Events）管理

**EventWatcherSubAssembler.kt** (`/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/EventWatcherSubAssembler.kt`)

```kotlin
updateFilter(key: Set<EventFinderQueryState>): List<TypedFilter>? {
    if (key.isEmpty()) { return null }

    val notes = distinct(key)
    val latestEOSEs = mutableMapOf<String, EOSETime>()

    notes.forEach { note ->
        relayPresenceList.forEach { relay ->
            val eose = eoseManager.getLatestEOSE(note, relay)
            if (eose != null) {
                latestEOSEs.getOrPut(relay.relayUrl) { eose }
                    .updateIfOlder(eose)
            }
        }
    }

    // フィルタ生成
}
```

**特徴:**
- 各ノートのリレー別EOSE時刻を追跡
- 最小値を算出して効率的なフィルタリング
- 重複リクエストを防止

### ローカルキャッシュでの処理

**LocalCache.kt** (`/amethyst/src/main/java/com/vitorpamplona/amethyst/model/LocalCache.kt`)

```kotlin
fun consume(
    event: ReactionEvent,
    relay: NormalizedRelayUrl?,
    wasVerified: Boolean,
): Boolean {
    val note = getOrCreateNote(event.id)
    // Already processed this event.
    if (note.event != null) return true

    if (wasVerified || justVerify(event)) {
        val author = getOrCreateUser(event.pubKey)
        val repliesTo = computeReplyTo(event)
        note.loadEvent(event, author, repliesTo)
        repliesTo.forEach { it.addReaction(note) }
        refreshNewNoteObservers(note)
        return true
    }
    return false
}
```

**キャッシュ戦略:**
- `getOrCreateNote()`でメモリ内キャッシュを管理
- `LargeSoftCache`でメモリ効率を最適化
- リレーハイントも同時に記録
- 重複イベントは無視（`note.event != null`チェック）

### Note.kt でのリアクション管理

**Note.kt** (`/amethyst/src/main/java/com/vitorpamplona/amethyst/model/Note.kt`)

```kotlin
// リアクション保持
val reactions: Map<String, List<Note>>

// リアクション追加
fun addReaction(note: Note) {
    val content = note.event?.content()?.firstFullCharOrEmoji(ImmutableListOfLists())
    if (content != null) {
        reactions[content]?.let {
            if (!it.contains(note)) {
                reactions[content] = it + note
                flowSet?.reactions?.invalidateData()
            }
        } ?: run {
            reactions[content] = listOf(note)
            flowSet?.reactions?.invalidateData()
        }
    }
}

// リアクション数カウント
fun countReactions(): Int = reactions.values.sumOf { it.size }
```

**リアクション監視の仕組み:**
- `NoteFlowSet`がメタデータごとに`NoteBundledRefresherFlow`を保持
  - metadata、reports、relays、**reactions**、boosts、replies、zapsなど
- データ変更時に`flowSet?.reactions?.invalidateData()`を呼び出し
- Observerパターンで自動的にUI更新を通知

**NoteBundledRefresherFlow:**
```kotlin
class NoteBundledRefresherFlow {
    val stateFlow: MutableStateFlow<NoteState>

    fun invalidateData() {
        // 状態を更新してFlowを発火
    }

    fun hasObservers(): Boolean {
        // 購読者の有無を確認
    }
}
```

## 3. 取得のタイミング

**結論: タイムライン取得時に同時に取得 + UIコンポーネント表示時に購読**

### タイミングの詳細

#### (1) タイムライン取得時

フィードのノートを取得する際、`EventFinderFilterAssembler`が自動的に：
1. 表示対象のノートを収集
2. リレーごとにバッチフィルタを生成
3. リアクション・返信・ザップなどをまとめて購読

#### (2) UIコンポーネント表示時

`ReactionsRow`がCompose UIとして表示される際：
```kotlin
@Composable
fun ReactionsRow(...) {
    // observeNoteReactions()が購読を開始
    val reactions by observeNoteReactions(baseNote, accountViewModel)
    val replyCount by observeNoteReplyCount(baseNote, accountViewModel)
    val boostCount by observeNoteRepostCount(baseNote, accountViewModel)
    val zapAmount by observeNoteZapAmount(baseNote, accountViewModel)

    // 状態変化に応じて自動的にUI更新
}
```

**購読のライフサイクル:**
- Composableの表示開始時に`EventFinderFilterAssemblerSubscription`が購読開始
- `remember(note, account)`でキャッシュ
- Composableが破棄されると購読も終了

#### (3) リアルタイム更新

ノートが画面に表示されている間：
- リレーから新しいリアクションイベントを受信
- `LocalCache.consume()`で処理
- `Note.addReaction()`でメモリ内グラフを更新
- `flowSet?.reactions?.invalidateData()`でFlowを発火
- `collectAsStateWithLifecycle()`で自動的にUI更新

### パフォーマンス最適化

**サンプリングとデバウンス:**
```kotlin
// 高頻度更新のカウンターにはサンプリングを適用
observeNoteReplyCount(note, accountViewModel): State<Int> {
    return note.flow()
        .replies
        .stateFlow
        .sample(200) // 200msごとにサンプリング
        .distinctUntilChanged()
        .collectAsStateWithLifecycle()
}
```

**最適化の特徴:**
- `sample(200)`: 200msごとにサンプリング
- `distinctUntilChanged()`: 値が変わった時のみ更新
- 返信数・リアクション数などの高頻度更新に適用

## まとめ

### リアクション取得の特徴

1. **表示方式**: タイムラインの各ノート下部にリアクション数を表示
2. **取得方式**: イベント単位のバッチ取得 + リアルタイム購読のハイブリッド
3. **タイミング**:
   - タイムライン取得時に同時にバッチ取得
   - UIコンポーネント表示時に購読開始
   - リアルタイムでリアクション更新を受信
4. **効率化**:
   - リレーごとにノートをグループ化してバッチリクエスト
   - EOSE管理で重複リクエストを防止
   - サンプリング・デバウンスでUI更新を最適化
   - ローカルキャッシュで重複処理を防止

### アーキテクチャの強み

- **リアクティブ**: Kotlin Flow/StateFlowによる宣言的なUI更新
- **効率的**: バッチ取得とリアルタイム購読の組み合わせ
- **スケーラブル**: メモリ内OOグラフで複雑な関係性を管理
- **パフォーマンス**: サンプリング、デバウンス、キャッシュによる最適化

### 参考ソース

- GitHub - Amethyst: https://github.com/vitorpamplona/amethyst
- Nostr Protocol: https://nostr.how/en/the-protocol
- NIP-25 (Reactions): https://github.com/nostr-protocol/nips/blob/master/25.md

## 関連ファイル一覧

### UIレイヤー
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/ui/note/ReactionsRow.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/ui/note/NoteCompose.kt`

### サービスレイヤー（リレークライアント）
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/EventObservers.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/EventFinderFilterAssembler.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/EventFinderFilterAssemblerSubscription.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/EventWatcherSubAssembler.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/FilterRepliesAndReactionsToNotes.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/loaders/NoteEventLoaderSubAssembler.kt`

### モデルレイヤー
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/Note.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/LocalCache.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/Account.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip25Reactions/ReactionAction.kt`

### フィードフロー
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/topNavFeeds/noteBased/NoteFeedFlow.kt`
- `/amethyst/src/main/java/com/vitorpamplona/amethyst/model/topNavFeeds/allFollows/AllFollowsFeedFlow.kt`

---
← [README](../README.md)
