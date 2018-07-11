# 実行手順

_7つのデータベース 7つの世界_ より、「第6章 CouchDB」の実行例を実際に試した時の手順

## 6.2 1日目： CRUD・Futon・cURL Redux

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

ここで再度↑を実行

### DELETEでドキュメント削除

以下を実行

```bash
curl -i -X DELETE \
"http://localhost:5984/music/7c2047269d93292148d5443e34005fb7" \
-H "If-Match: 2-17e4ce41cd33d6a38f04a8452d5a860b"
```

## 6.3 2日目：問い合わせビューの作成

### ビューでドキュメントにアクセスする

以下を実行

```bash
curl http://localhost:5984/music/_all_docs
```

以下を実行

```bash
curl http://localhost:5984/music/_all_docs?include_docs=true
```

### 最初のビューを作成する

以下の関数をviewに追加する

```javascript
function(doc) {
    emit(null, doc);
}
```


上記の関数を以下に修正する

```javascript
function(doc) {
    emit(doc._id, {rev: doc._rev});
}
```

以下を実行する(※動かない)

```bash
curl -X POST \
http://localhost:5984/music/_temp_view \
-H "Content-Type: application/json" \
-d '{"map":"function(doc){emit(doc._id,{rev:doc._rev});}"}'
```

以下を実行する(※動かない)

```bash
curl -X POST \
http://localhost:5984/music/_temp_view?include_docs=true \
-H "Content-Type: application/json" \
-d '{"map":"function(doc){emit(doc._id,{rev:doc._rev});}"}'
```

## ビューをデザインドキュメントとして保存する

とくになし

## アーティスト名で検索

以下の関数をviewに追加する

```javascript
function(doc) {
    if ('name' in doc) {
        emit(doc.name, doc._id);
    }
}
```

## アルバム名で検索

以下の関数をviewに追加する

```javascript
function(doc) {
    if ('name' in doc && 'albums' in doc) {
        doc.albums.forEach(function(album){
            var key = album.title || album.name,
            value = {by: doc.name, album: album};
            emit(key, value);
        });
    }
}
```

### カスタムビューへの問い合わせ

以下を実行

```bash
curl http://localhost:5984/music/_design/artists/_view/by_name
```

以下を実行

```bash
curl http://localhost:5984/music/_design/albums/_view/by_name
```

以下を実行

```bash
curl 'http://localhost:5984/music/_design/albums/_view/by_name?key="Help!"'
```

### RubyでデータをCouchDBにインポート

以下でcouchdbコンテナに入る

```bash
docker-compose exec db bash
```

以下でファイルをダウンロード

```bash
wget http://img.jamendo.com/data/dbdump_artistalbumtrack.xml.gz
```

以下を実行

```bash
zcat dbdump_artistalbumtrack.xml.gz | ruby import_from_jamendo.rb
```

以下を実行

```bash
curl http://localhost:5984/music/_design/artists/_view/by_name?limit=5
```

以下を実行

```bash
curl http://localhost:5984/music/_design/artists/_view/by_name?limit=5\&startkey=%22C%22
```

以下を実行

```bash
curl http://localhost:5984/music/_design/artists/_view/by_name?startkey=%22C%22\&endkey=%22D%22
```

以下を実行

```bash
curl http://localhost:5984/music/_design/artists/_view/by_name?startkey=%22D%22\&endkey=%22C%22\&descending=true
```

## 6.4 3日目：応用的なビュー・Changes API・レプリケーション

### 応用的なビューとreducer

以下の関数をviewに追加する

```javascript
function(doc){
    (doc.albums || []).forEach(function(album){
        (album.tracks || []).forEach(function(track){
            (track.tags || []).forEach(function(tag){
                emit(tag.idstr, 1);
            });
        });
    });
}
```

以下の関数を↑のviewのreduceに追加する

```javascript
function(key, values, rereduce) {
    return sum(values);
}
```

### reducerの呼び出し

とくになし

### 変更の監視

とくになし

### ChangesをcURLで取得

以下を実行

```bash
curl http://localhost:5984/music/_changes
```

以下を実行(※動かない)

```bash
curl http://localhost:5984/music/_changes?since=99
# 本当は以下のように実行しないといけないっぽい
curl http://localhost:5984/music/_changes?since=110-g1AAAAIjeJzLYWBg4MhgTmHgzcvPy09JdcjLz8gvLskBCjMlMiTJ____PyuDOZEvFyjAbpJkkWpuYYyuGIf2JAUgmWQPNUEIbIJFcpJpiqUlsSY4gEyIh5rADTbB0CzFLCWJaBMSQCbUQ03gB5uQaJyUZmlhQKQJeSxAkqEBSAENmY8wJcko2cg0kVh3QExZADFlP8gUTrApaWmGJilARIopByCm3EfEi2WyWaqlkSlJpjyAmIIUu8lGackGlsnoerIAzdOooQ
```

以下を実行(※動かない)

```bash
curl http://localhost:5984/music/_changes?feed=longpoll\&since=9000
```

### Node.jsで変更をポーリング

コンテナに入って以下を実行

```bash
node watcher_changes_longpolling.js music
# ※本の中で `node watch_changes_longpolling.js music` とあるがこれは間違い
```

### 変更を継続的に監視

以下を実行(※動かない)

```bash
curl 'http://localhost:5984/music/_changes?since=97&feed=continuous'
# 本当は以下のように実行しないといけないっぽい
curl 'http://localhost:5984/music/_changes?since=97-g1AAAAIjeJyVz1kKwjAQANBgBdeKN9AjtOli8mVvolkaSqnJh_7rTfQmehO9ScxSEApCC8MMDDOPmQYAMK0CDkKppOJlIVWlzpfGtEcE0I3Wuq4Csj6ZxiSlqNyhpDv8Z51uTab7Vlg5ATGacYz7CoUVDq2wcEKc85zT3sLRCtdWCJ1AEiowinoKcmwyuJlikLtVlk6hkMGM9L3DKw-vPK0yc4oQccpNDFFeXnlbZe4UzPISw2yQ8vGK_n3EoGARZt2d-gvD26iU&feed=continuous'
```

### 変更のフィルタリング

以下を実行

```bash
curl -X PUT \
http://localhost:5984/music/_design/wherabouts \
-H "Content-Type: application/json" \
-d '{"language":"javascript","filters":{"by_country": "function(doc,req){return doc.country === req.query.country;}" }}'
```

以下を実行

```bash
curl "http://localhost:5984/music/_changes?\
filter=wherabouts/by_country&\
country=RUS"
```

### CouchDBでデータをレプリケート

とくになし

### コンフリクトの発生

以下を実行

```bash
curl -X PUT "http://localhost:5984/music/theconflicts" \
-H "Content-Type: application/json" \
-d '{ "name": "The Conflicts" }'
```

以下を実行

```bash
curl "http://localhost:5984/music-repl/theconflicts"
```

以下を実行

```bash
curl -X PUT "http://localhost:5984/music-repl/theconflicts" \
-H "Content-Type: application/json" \
-d '{ "_id": "theconflicts", "_rev": "1-e007498c59e95d23912be35545049174", "name": "The Conflicts", "albums": ["Conflicts of Interest"] }'
```

以下を実行

```bash
curl -X PUT "http://localhost:5984/music/theconflicts" \
-H "Content-Type: application/json" \
-d '{ "_id": "theconflicts", "_rev": "1-e007498c59e95d23912be35545049174", "name": "The Conflicts", "albums": ["Conflicting Opinions"] }'
```

### コンフリクトの解消

以下を実行

```bash
curl http://localhost:5984/music-repl/theconflicts?conflicts=true
```

以下を実行。_revの部分は適宜変更すること

```bash
curl http://localhost:5984/music-repl/theconflicts?rev=2-cab47bf4444a20d6a2d2204330fdce2a
```

