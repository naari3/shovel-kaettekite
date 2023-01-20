# 環境を用意

docker-compose.yml を作った

```
$ docker compose up -d
```

http://localhost:8080 で phpMyAdmin にアクセスできるので、基本はそこでクエリを実行する

このディレクトリに frm と idb を置く

# 検索

- https://qiita.com/___uhu/items/74168be48c05638c7ac5

# ↑ の情報を試す

まず database shovel を作成する。phpMyAdmin 上でデフォルトの状態で作成。

- デフォルトで既に shovel の本番環境と同じ状態だった

以降、database shovel のページ内にある SQL タブ上でクエリを実行する

```
$ od -Ax -j 16418 -N4 -t x1 t_words.ibd
004022 00 00 00 a1
004026
```

0xA1 = 161

次に、元あったテーブル定義と同じ番号になるまでテーブルの作成/削除を繰り返す

テーブルの作成 ↓

```sql
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

番号の確認 ↓

```sql
SELECT SPACE,NAME from information_schema.innodb_sys_tablespaces WHERE name LIKE '%t_words';
```

削除 ↓

```sql
DROP TABLE t_words;
```

繰り返す ↓ (5 進める場合)　`クエリボックスを保持する` にチェックを入れておくと Ctrl+Enter を連打するたびに 5 進むようになる

```sql
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
DROP TABLE t_words;
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
DROP TABLE t_words;
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
DROP TABLE t_words;
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
DROP TABLE t_words;
CREATE TABLE `t_words` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'primary id',
  `word` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin UNIQUE NOT NULL COMMENT 'word',
  `reading` varchar(500) NOT NULL COMMENT 'reading',
  `is_uppercase_distinction` char(1) NOT NULL DEFAULT '1' COMMENT '大文字、小文字を区別するか 1:区別する 0:区別しない',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
DROP TABLE t_words;
```

# 上書きしてみる (やんなくていい)

```bash
sudo cp t_words.* data/shovel/
sudo chown -R 999:999 data/shovel
```

## ダメだった！

`table has corrupted!` みたいなことを言われた

# discard を試す

```sql
ALTER TABLE t_words DISCARD TABLESPACE;
```

frm と ibd を上書き

```bash
docker compose down
sudo cp t_words.* data/shovel/
sudo chown -R 999:999 data/shovel
docker compose up -d
```

```sql
ALTER TABLE t_words IMPORT TABLESPACE;
```

なんかエラー出る。index を削除する必要があるっぽい

```
#1815 - Internal error: Drop all secondary indexes before importing table shovel/t_words when .cfg file is missing.
```

https://stackoverflow.com/questions/70470347/internal-error-drop-all-secondary-indexes-before-importing-table-onlinelogistic

word についている index を削除する(phpMyAdmin 上で実行した)

```sql
ALTER TABLE t_words IMPORT TABLESPACE;
```

一応レコードは見れるようになった
