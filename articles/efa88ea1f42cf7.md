---
title: "sqlboilerで自動生成するGoの型を任意の物に起きかえる"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["db","mysql","sqlboiler"]
published: true
---

例えば

```sql
CREATE TABLE `Post` (
  id VARCHAR(255) NOT NULL,
  title VARCHAR(255) NOT NULL,
  text TEXT NOT NULL,
  created_at DATETIME NOT NULL,
  updated_at DATETIME NOT NULL,
    PRIMARY KEY (id),
);
```

のようなテーブルが存在していた場合、生成されるGoの構造体は以下のようになる。

```go:Post.go
type Post struct {
    ID        string    `boil:"id" json:"id" toml:"id" yaml:"id"`
    Title     string    `boil:"title" json:"title" toml:"title" yaml:"title"`
    Text      string    `boil:"text" json:"text" toml:"text" yaml:"text"`
    CreatedAt time.Time `boil:"created_at" json:"created_at" toml:"created_at" yaml:"created_at"`
    UpdatedAt time.Time `boil:"updated_at" json:"updated_at" toml:"updated_at" yaml:"updated_at"`

    R *postR `boil:"-" json:"-" toml:"-" yaml:"-"`
    L postL  `boil:"-" json:"-" toml:"-" yaml:"-"`
}
```

設定ファイルに下記のように追記

```toml:sqlboiler.toml
[[types]]
  [types.match]
    tables = ["Post"]   # 対象のテーブル、省略した場合は全てのテーブル
    name = "id"         # 対象のカラム
  [types.replace]
    type = "uuid.UUID"  # 設定したい型
  [types.imports]
    third_party = ['"github.com/google/uuid"'] # 設定したい型に必要なパッケージ
```

sqlboilerによって生成される構造体は以下のようになる

```go:Post.go
type Post struct {
    ID        uuid.UUID `boil:"id" json:"id" toml:"id" yaml:"id"`
    Title     string    `boil:"title" json:"title" toml:"title" yaml:"title"`
    Text      string    `boil:"text" json:"text" toml:"text" yaml:"text"`
    CreatedAt time.Time `boil:"created_at" json:"created_at" toml:"created_at" yaml:"created_at"`
    UpdatedAt time.Time `boil:"updated_at" json:"updated_at" toml:"updated_at" yaml:"updated_at"`

    R *postR `boil:"-" json:"-" toml:"-" yaml:"-"`
    L postL  `boil:"-" json:"-" toml:"-" yaml:"-"`
}
```

`ID`の型が`uuid.UUID`に変わっていることが分かる。

他にもsqlboilerのmatchには他にも様々な設定が出来る

```toml:sqlboiler.toml
[[types]]
  [types.match]
    tables = ["Post"]   # 対象のテーブル、省略した場合は全てのテーブル
    name = "id"         # 対象のカラム
    type = "int"        # 生成後の型
    db_type = "int"     # 元の型
    nullable = false    # nullを許容しているか否か
```
