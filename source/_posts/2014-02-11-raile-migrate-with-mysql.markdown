---
layout: post
title: "HerokuのClearDBでdb:migrateを実行する"
date: 2014-02-11 20:47:00 +0900
comments: true
categories: 
---

Heroku上のPostgreSQLやMySQLでは、rails generate migrationで生成したスクリプトが下記のエラーで実行できない場合があります。

        PG::Error: ERROR:  column "number" cannot be cast to type integer
    : ALTER TABLE "tweets" ALTER COLUMN "number" TYPE integer

その場合は、migrateスクリプトのadd_columnメソッドをコメントアウトし、SQLでDDL文を直接実行することで解決できます。

    # def change
      #add_column :line_items, :quantity, :interger, default: 1 #コメントアウト
      connection.execute(%q{
        alter table line_items
        add column quantity integer default 1
      })
    end

### 参考

* [postgresql - Rails Migration Error w/ Postgres when pushing to Heroku - Stack Overflow](http://stackoverflow.com/questions/12603498/rails-migration-error-w-postgres-when-pushing-to-heroku)
