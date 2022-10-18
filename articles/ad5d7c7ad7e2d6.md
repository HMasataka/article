---
title: "docker-composeでmysql環境を構築する"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker","docker-compose","mysql","DB"]
published: false
---

## docker-compose.yml

```yaml:docker-compose.yaml
version: '3.9'

services:
  mysql:
    image: mysql:8.0.27
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_DATABASE: db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    restart: always
    networks:
      - mysql-network

  cli:
    image: mysql:8.0.27
    networks:
      - mysql-network
    command: mysql -hmysql -uuser -ppassword db

networks:
  mysql-network:
    driver: bridge
```

* `services.mysql.environment`配下でmysqlの設定を行っている。
  * ここではrootのパスワードを`mysql`、デフォルトのDB名を`db`、ユーザー名を`user`、パスワードを`password`に設定しているので適宜修正する。ここで設定した情報を利用してmysqlクライアントからMySQLへ接続する。
* `services.mysql.restart`を`always`に設定することでPCを再起動したとき等もMySQLを起動したままに出来る
* `networks`はmysqlクライアントから接続出来るようにするための設定

## MySQLの起動

下記コマンドにより、MySQLを起動出来る

```bash
docker-compose up -d mysql
```

## MySQL クライアントによる接続

下記コマンドにより、MySQLに接続出来る

```bash
docker-compose run cli
```

以下のような画面が出れば成功

![mysql client image](/images/mysql.png)

## Go言語からの接続

`services.mysql.ports` 部分でポートフォワーディングの設定も行っているので当然アプリケーションからも接続できる。
例えばGo言語から接続したい場合は以下のようにすれば接続出来る

```go
import (
  "database/sql"
  "time"

  "github.com/go-sql-driver/mysql"
)

func connectDB() *sql.DB {
  jst, err := time.LoadLocation("Asia/Tokyo")
  if err != nil {
    // エラーハンドリング
  }

  c := mysql.Config{
    DBName:    "db",
    User:      "user",
    Passwd:    "password",
    Addr:      "localhost:3306",
    Net:       "tcp",
    ParseTime: true,
    Loc:       jst,
  }

  db, err := sql.Open("mysql", c.FormatDSN())
  // 以下省略
}
```
