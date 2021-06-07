# Ruby on RailsでSQL文を学ぶ
## 目的
- コンソール画面等で表示されるSQL文の意味を理解する
- Ruby on Railsで用いられるActiverecordとSQLの対応を理解する


## 準備
### ActiveRecordの理解
- 直接SQL文を記述しなくても、データベースとデータのやりとりをすることができるRuby on Railsの機能のこと
- SQL文の翻訳語
- newメソッド、createメソッド、allメソッド、findメソッド、whereメソッドなど

### サンプルアプリの作成
scaffoldでCRUD機能をもつ簡単なアプリを作成する。
データベースはPostgresqlとし、テストディレクトリは削除しておく。

~~~ruby
rails new sql-sample -d postgresql -T
cd sql_sample
rails g scaffold hoge title:string content:text
rails db:create
rails db:migrate
~~~

## まずはコンソール画面で色々見てみる
まずは、`rails c`でコンソール画面を立ち上げ、
Hogeモデルのインスタンスを確認してみる。

```ruby
Hoge.all

# =>
Hoge Load (0.4ms)  SELECT "hoges".* FROM "hoges" /* loading for inspect */ LIMIT $1  [["LIMIT", 11]]
=> #<ActiveRecord::Relation []>
```

- `SELECT "hoges".* FROM "hoges"`は、hogesテーブルからhoges.*カラムのデータを取得する、ということ。
- `/* loading for inspect */` は謎⭐
- `LIMIT $1`は、最大で$1個のデータを取得する、という上限のこと
- $を用いて引数を用いることができる。$1が第1引数、$2が第2引数...。引数はどこで指定しているの？
- `[["LIMIT", 11]]`とは...?
- `#<ActiveRecord::Relation []>`とは、`Hoge.all`で返されたインスタンスのこと。今のところ、インスタンスはひとつもない状態


次に、`Hoge.new`してみる
```ruby
hoge1 = Hoge.new
=> #<Hoge id: nil, title: nil, content: nil, created_at: nil, updated_at: nil>

hoge1.save
# =>
  TRANSACTION (0.4ms)  BEGIN
  Hoge Create (8.6ms)  INSERT INTO "hoges" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2021-06-07 13:37:23.274497"], ["updated_at", "2021-06-07 13:37:23.274497"]]
  TRANSACTION (6.5ms)  COMMIT
=> true

Hoge.all
# =>
Hoge Load (0.8ms)  SELECT "hoges".* FROM "hoges" /* loading for inspect */ LIMIT $1  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Hoge id: 4, title: nil, content: nil, created_at: "2021-06-07 13:37:23.274497000 +0000", updated_at: "2021-06-07 13:37:23.274497000 +0000">]>
```
- `Hoge.new`すると、中身がnilのインスタンスが生成される（引数を指定していない、initializeメソッドがないので）
- `hoge1.save`でhoge1の情報を保存。
- `TRANSACTION BEGIN`：トランザクション開始の合図
- `TRANSACTION COMMIT`：トランザクション終了の合図
- データの追加・更新・削除についての処理のまとまりをトランザクションという。
- `INSERT INTO テーブル名 (列名1, 列名2, ...) VALUES (値1, 値2, ...)`で、テーブルの各列に値を保存する


newメソッド→saveメソッドではなく、createメソッドでデータの保存をした場合、こんな感じ。

```ruby
Hoge.create(title: "test", content: "YES")
  TRANSACTION (0.4ms)  SAVEPOINT active_record_1
  Hoge Create (0.9ms)  INSERT INTO "hoges" ("title", "content", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["title", "test"], ["content", "YES"], ["created_at", "2021-06-07 14:01:46.739531"], ["updated_at", "2021-06-07 14:01:46.739531"]]
  TRANSACTION (0.2ms)  RELEASE SAVEPOINT active_record_1
=> #<Hoge id: 8, title: "test", content: "YES", created_at: "2021-06-07 14:01:46.739531000 +0000", updated_at: "2021-06-07 14:01:46.739531000 +0000">

Hoge.all
# =>
  Hoge Load (0.5ms)  SELECT "hoges".* FROM "hoges" /* loading for inspect */ LIMIT $1  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Hoge id: 7, title: nil, content: nil, created_at: "2021-06-07 13:57:38.212732000 +0000", updated_at: "2021-06-07 13:57:38.212732000 +0000">, #<Hoge id: 8, title: "test", content: "YES", created_at: "2021-06-07 14:01:46.739531000 +0000", updated_at: "2021-06-07 14:01:46.739531000 +0000">]>
```

- `SAVEPOINT active_record_1`と`RELEASE SAVEPOINT active_record_1`とは
- `RETURNING`とは

id: 7のデータを更新する。

```ruby
hoge1 = Hoge.find(7)
# =>
Hoge Load (0.7ms)  SELECT "hoges".* FROM "hoges" WHERE "hoges"."id" = $1 LIMIT $2  [["id", 7], ["LIMIT", 1]]
=> #<Hoge id: 7, title: nil, content: nil, created_at: "2021-06-07 13:57:38.212732000 +0000", updated_at: "2021-06-07 13:57:38.212732000 +0000">

hoge1.update(title: "test2")
# =>
  TRANSACTION (0.2ms)  SAVEPOINT active_record_1
  Hoge Update (0.5ms)  UPDATE "hoges" SET "title" = $1, "updated_at" = $2 WHERE "hoges"."id" = $3  [["title", "test2"], ["updated_at", "2021-06-07 14:13:09.544111"], ["id", 7]]
  TRANSACTION (0.1ms)  RELEASE SAVEPOINT active_record_1
=> true

Hoge.find(7)
# =>
Hoge Load (0.9ms)  SELECT "hoges".* FROM "hoges" WHERE "hoges"."id" = $1 LIMIT $2  [["id", 7], ["LIMIT", 1]]
=> #<Hoge id: 7, title: "test2", content: nil, created_at: "2021-06-07 13:57:38.212732000 +0000", updated_at: "2021-06-07 14:13:09.544111000 +0000">
```

- `UPDATE`とは...

## 参考文献
- [【初心者向け】RailsのActive Recordの解説&メソッドまとめ](https://qiita.com/ryokky59/items/a1d0b4e86bacbd7ef6e8)
- [クエリメソッドをまとめてみた[Rails]](https://qiita.com/takuyanin/items/ed2641dcb5fad35f9067)
- [ActiveRecord::Relationとは一体なんなのか](https://spirits.appirits.com/doruby/8831/)