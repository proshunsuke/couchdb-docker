# 実行手順

_7つのデータベース 7つの世界_ より、「第6章 CouchDB」の実行例を実際に試した時の手順

## 6.2 1 日 目： CRUD・Futon・cURL Redux

### 寝心地のいいFuton

http://localhost:5984/_utils/ に移動する

`Create Database` で `music` を追加

以下を追加

```json
"name": "The Beatles"
```

以下を追加

```json
"albums": ["Help!""Sgt. Pepper’s Lonely Hearts Club Band","Abbey"]
```

albumsの中身を以下に変更

```json
[{ "title": "Help!", "year": 1965 },{ "title": "Sgt. Pepper’s Lonely Hearts Club Band", "year": 1967 },{ "title": "Abbey Road", "year": 1969 }]
```

### cURLを使ったRESTfulなCRUD操作

以下を実行

```bash
curl http://localhost:5984/
```

以下を実行

```bash
curl http://localhost:5984/music/
```

### GETでドキュメント参照

以下を実行。_idの部分は適宜変更すること

```bash
curl http://localhost:5984/music/7c2047269d93292148d5443e34000d8f
```

### POSTでドキュメント作成

以下を実行

```bash
curl -i -X POST "http://localhost:5984/music/" \
-H "Content-Type: application/json" \
-d '{ "name": "Wings" }'
```

### PUTでドキュメント更新

以下を実行。_idの部分は適宜変更すること

```bash
curl -i -X PUT \
"http://localhost:5984/music/7c2047269d93292148d5443e34005fb7" \
-H "Content-Type: application/json" \
-d '{ "_id": "7c2047269d93292148d5443e34005fb7", "_rev": "1-2fe1dd1911153eb9df8460747dfe75a0", "name": "Wings", "albums": ["Wild Life", "Band on the Run", "London Town"] }'
```

ここで再度↑を実行する

### DELETEでドキュメント削除

以下を実行

```bash
curl -i -X DELETE \
"http://localhost:5984/music/7c2047269d93292148d5443e34005fb7" \
-H "If-Match: 2-17e4ce41cd33d6a38f04a8452d5a860b"
```
