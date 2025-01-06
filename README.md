# Hadoop & Hive クラスタ構築 Docker Compose

このリポジトリは、Docker Compose を使用して Hadoop (HDFS) と Hive の分散環境を構築するための設定ファイルを提供します。この環境では、Hive メタストアとして PostgreSQL を利用します。

## 前提条件

- Docker がインストールされていること
- Docker Compose がインストールされていること

## 構成

この環境は、以下の Docker コンテナで構成されています。

- **namenode:** Hadoop HDFS のネームノード。ファイルシステムのメタデータを管理します。
- **datanode:** Hadoop HDFS のデータノード。実際のデータを格納します。
- **hive-server:** Hive のサーバ。クライアントからのクエリを受け付け、実行します。
- **hive-metastore:** Hive のメタストア。テーブル定義やパーティション情報などを管理します。
- **hive-metastore-postgresql:** Hive メタストア用の PostgreSQL データベース。

## ファイル構成

- `docker-compose.yml`: Docker Compose の設定ファイル
- `hadoop-hive.env`: 環境変数ファイル
- `hdfs/`: Hadoop HDFS の永続化データ格納ディレクトリ
    - `hdfs/namenode`: ネームノードのメタデータ
    - `hdfs/datanode`: データノードのデータ
- `metastore-postgresql/`: PostgreSQL の永続化データ格納ディレクトリ
    - `metastore-postgresql/postgresql/data`: PostgreSQL のデータベースデータ
- `employee/`: Hive で利用するサンプルデータ格納ディレクトリ
    - (例) `employee/employee.csv` など
- `README.md`: このドキュメント

## 使い方

1. このリポジトリをクローンまたはダウンロードします。
2. `hadoop-hive.env` を必要に応じて編集します (特に、パスワードやポート番号)。
3. `docker-compose.yml` があるディレクトリで、以下のコマンドを実行します。
   ```bash
   docker-compose up -d
   ```
   これにより、バックグラウンドでコンテナが起動します。
4. コンテナが起動するまでしばらく待ちます。以下のコマンドでコンテナの状態を確認できます。
   ```bash
   docker-compose ps
   ```
   すべてのコンテナが `Up` と表示されれば起動完了です。
5. Hive に接続するには、以下のコマンドで `hive-server` コンテナ内に入り、Beeline を起動します。
   ```bash
   docker exec -it hive-server /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
   ```

## 環境変数 (`hadoop-hive.env`)

以下の環境変数が設定可能です。`hadoop-hive.env` ファイルを編集することでカスタマイズできます。

```env
# 例
# HADOOP
HADOOP_USER=hadoop
HADOOP_GROUP=hadoop
# HIVE
HIVE_SERVER2_THRIFT_PORT=10000
HIVE_METASTORE_PORT=9083
# POSTGRES
POSTGRES_USER=hive
POSTGRES_PASSWORD=hive
POSTGRES_DB=metastore
```

## HDFS の Web UI

HDFS の Web UI は、以下の URL でアクセスできます。

[http://localhost:50070](http://localhost:50070)

## Hive の Beeline クライアント

Hive の Beeline クライアントは、以下のコマンドで `hive-server` コンテナ内から起動できます。

```bash
docker exec -it hive-server /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
```

## サンプルデータ (`employee/`)

`employee` ディレクトリに CSV ファイルを配置することで、Hive でテーブルを作成し、利用することができます。例えば、`employee/employee.csv` というファイルを作成し、Hive でテーブルを作成してデータをロードする手順は以下の通りです。

1. `employee.csv` ファイルを以下のように作成 (例)
   ```csv
   id,name,age,salary
   1,John,30,5000
   2,Alice,25,6000
   3,Bob,35,7000
   ```
2. Beeline で以下のクエリを実行
   ```sql
   CREATE EXTERNAL TABLE employee (
      id INT,
      name STRING,
      age INT,
      salary INT
   )
   ROW FORMAT DELIMITED
   FIELDS TERMINATED BY ','
   LOCATION '/employee';

   LOAD DATA LOCAL INPATH '/employee/employee.csv' OVERWRITE INTO TABLE employee;
   SELECT * FROM employee;
   ```
   これで、`employee` テーブルが作成され、データがロードされます。

## コンテナの停止

以下のコマンドで、コンテナを停止できます。
```bash
docker-compose down
```

## その他

- 環境変数の設定変更後にコンテナを再起動する場合は、`docker-compose restart` コマンドを利用してください。
- 各コンテナのログを確認するには、`docker logs <container_name>` コマンドを利用してください。
   例: `docker logs namenode`
- この構成は開発環境を想定しています。本番環境での利用には、よりセキュアな構成と設定が必要になります。
