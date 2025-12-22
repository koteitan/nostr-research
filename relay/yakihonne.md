← [README](../README.md)

# yakihonne リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (kind:10002 NIP-65)

## ソースコード

**ファイル**: `src/Helpers/NDKInstance.js`

```javascript
const ndkInstance = new NDK({
  explicitRelayUrls: relaysOnPlatform,
  enableOutboxModel: true,
  // ...
});
```

**ファイル**: `src/Hooks/useOutboxRelays.js`

```javascript
const useOutboxRelays = () => {
  const [outboxRelays, setOutboxRelays] = useState([]);
  const userFollowingsRelays = useSelector(state => state.userFollowingsRelays)

  useEffect(() => {
    if (!(userFollowingsRelays.length > 0)) return;
    let allRelays = [
      ...new Set(userFollowingsRelays.map((relay) => relay.relays.map(_ => _.url)).flat()),
    ];
    let relaysStats = allRelays.map((relay) => {
      return {
        url: relay,
        pubkeys: userFollowingsRelays.filter((user) => user.relays.find(_ => _.url === relay))
          .map((user) => user.pubkey)
      }
    }).sort((a, b) => b.pubkeys.length - a.pubkeys.length)
    setOutboxRelays(relaysStats);
  }, [userFollowingsRelays]);

  return { outboxRelays };
};
```

## 説明

- NDK の enableOutboxModel が有効
- フォローのリレー情報を収集して outboxRelays を構築
- リレーごとの pubkey 数でソート

## 参考
- https://github.com/nicehashdev/yakihonne-web-app/blob/main/src/Helpers/NDKInstance.js
- https://github.com/nicehashdev/yakihonne-web-app/blob/main/src/Hooks/useOutboxRelays.js

---
← [README](../README.md)
