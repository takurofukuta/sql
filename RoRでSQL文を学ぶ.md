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
- `#<ActiveRecord::Relation []>`とは、`Hoge.all`で返されたインスタンスのこと


次に、`Hoge.new`してみる
```ruby
Hoge.new



```



## 参考文献
- [【初心者向け】RailsのActive Recordの解説&メソッドまとめ](https://qiita.com/ryokky59/items/a1d0b4e86bacbd7ef6e8)
- [クエリメソッドをまとめてみた[Rails]](https://qiita.com/takuyanin/items/ed2641dcb5fad35f9067)
- [ActiveRecord::Relationとは一体なんなのか](https://spirits.appirits.com/doruby/8831/)