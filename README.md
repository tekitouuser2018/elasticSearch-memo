# elasticSearch-memo


### Elasticsearchの特徴
----
Apach Luceneをベースとした、Javaで書かれた全文検索ソフトウェア。
ライセンス： Apache License2.0(オープンソースソフトウェア)
サポートポリシー： 各リリースの一般提供開始日から１８ヶ月間がサポート期間
メンテナンスポリシー： 最新２世代のメジャーバージョンに対してのみメンテナンスが行われる

- 分散配置による高速化と高可用性の実現
- シンプルなREST APIによるアクセス
- JSONフォーマットに対応した柔軟性の高いドキュメント指向データベース
- ログ収集・可視化などの多様な関連ソフトウェアとの連携


### レイヤー
----

- indexer


- searcher


- crawler


- 検索ライブラリレイヤー
インデクサとサーチャーの機能を提供するレイヤとなるソフトウェア形態
例：Apach Lucine

- 検索サーバレイヤー
インターフェースや管理機能を提供するレイヤ
例：Elasticsearch, Apache Solr

- 検索システムレイヤー(エンタープライズサーチ)
クローラやWeb UI機能を付加したレイヤｂｇ


### 論理的な概念
----

- Document:　 
Elasticsearchに格納する１つの文章の単位でJSONオブジェクト。RDBの１行のレコードに該当。

- Fiels:
ドキュメント内のkey,valueの組。

- Index:
ドキュメントの保存場所。RDBのテーブルに該当

- Document type:
ドキュメントの構造。１つのインデックスの中に格納できるドキュメントタイプは１つのみ

- Mapping:
ドキュメント内の各フィールドのデータ構造やデータ型を記述した情報。
事前定義しなくても、ドキュメント格納時に自動的に定義してくれる。

### 物理的な概念
----

- Node:
Elasticsearchが動作する各サーバのこと。(=JVMインスタンス)
１つのOS上に１ノードだけ起動するのが典型

- Cluster:
複数のノードを起動すると、お互いにメッセージを送信しあい、自律的に形成するノードのグループ

- Shard:
インデックスのデータを一定数に分割した各部分のこと。
初期設定数は「５」。事前に設定しておく必要があり、後から変更はできない。

- Replica:
各シャードが自動的に複製されたのも。インデックスが更新される際は、必ず最初にプライマリシャーとに反映されてから、逐次レプリカシャードを複製するようになっている。
インデックス作成後も設定数は変更可能。


### システム構成
----

- ノード種別
 - Master(Master-eligible)ノード:
   Elasticsearchクラスタには、必ず１台のMasterノードが必要。
   - ノードの参加と離脱の管理
   - クラスタメタデータの管理
   - シャードの割り当てと再配置
 - Dataノード: Elasticsearchにおけるデータの格納、クエリへの応答、そして内部的にLuceneインデックスファイルのマージなどの管理を行う役割を持ちます。また、クライアントから受け付けたリクエストのハンドリングも行います。
 - Ingestノード: ver5.0から登場。Elasticsearchノードの内部でデータの変換や加工を実行できる機能。
 - Coordinatingノード: クライアントからのリクエストのハンドリングのみを実行。


### スプリットブレイン問題について
----

スプリットブレインとは、Elasticsearchクラスタが２つに分断されて、かつ、それぞれのクラスタ上でMasteerノードが独立して稼働してしまう状態のこと。これを回避するためにMasterノードを高可用性構成にする場合、Master-eligibleノードは3ノード以上の奇数台で構成する必要がある。

- 必須対処方法
 - Master-eligibleノードの数を３以上の奇数にすること
 - elasticsearch.yml設定ファイルのパラメータ「discovery.zen.minimum_master_nodes」の数をMaster-eligibleノード数の過半数に設定すること((N/2)+1)。このパラメータはデフォルト値が「１
 」であるため、事前設定が必要。

 ### メモリ要件
----
- 必要メモリは8~64GB程度とするのが理想的。Elasticsearchでは、データアクセスの際のキャッシュ用途でメモリを多量に使います。これらの処理はJVMのヒープ領域を使うため、JVMのヒープサイズは十分に確保する必要があります。ヒープサイズは、OS全体のメモリ量の半分を上限として割り当てます。そして残りの半分は、OSがファイルシステムキャッシュに使うメモリとして残しておきます。なお、Elasticsearchに割り当てるJVMヒープサイズの上限を32GBとしないと性能上の問題が起きると言われています。

 ### CPU要件
----
- 2~8Core程度で十分と言われている

 ### ディスク要件
----
- インデックス、クエリともにディスクI/Oを必要とします。ディスクは性能上のボトルネックになりやすいポイントであるため、HDDよりSSDが好ましい。

### ネットワーク要件
----
- 1GbE,10GbEといった高速なネットワーク構成が望ましい。シャードの複製やリバランスを行う際にネットワーク帯域が特に必要となる。ノードの追加や削除、故障などが起きた場合に、一時的に大量のデータがノード缶を移動することになる。この間にも各ノードとMasterノードとの間では生存確認パケットが行き来している。もしシャードの複製やリバランスの影響により生存確認パケットに伝送遅延が起きてしまうと、あるノードが誤って障害と判定される可能性がある。そうするとノード障害により更なるシャードのリバランスが発生する、といった悪循環が発生する恐れがある。同様の理由により、クラスタを構成するノードを地理的に離れた場所に分散配置することも推奨されない。

### REST APIによる操作について
----

- ボディを付加する場合、ヘッダ情報として「-H 'Content-Type : application/json'」を指定する必要がある。Elasticsearch6.0以降はこれを省略するとエラーになる。

- ステータスコード409 : Conflict

### Mappingの変更について
----
- データストリームに新しくmappingを追加する：

putで新しい項目を追加する

```
PUT /test/_mapping
{
  "properties": {
    "host": {
      "properties": {
        "addadd": {
          "type": "keyword"
        }
      }
      }
    }
  }
}
```

結果：

```
{
  "acknowledged" : true
}
```

- データストリームに存在するmappingのフィールドを変更する：

❌putで既存の項目を更新する

```
PUT /test/_mapping
{
  "properties": {
    "host": {
      "properties": {
        "addadd": {
          "type": "text"
        }
      }
      }
    }
  }
}
```

結果：

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "mapper [host.addadd] cannot be changed from type [keyword] to [text]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "mapper [host.addadd] cannot be changed from type [keyword] to [text]"
  },
  "status" : 400
}
```

・ReIndexを行う:

- データストリームで動的にインデックスをセッティングする：

- データストリームで静的にインデックスをセッティングする：



### 基本クエリ - 全文検索クエリ
----

- match_all : 特殊なクエリ。検索条件を指定しなくても必ず全件を返す。格納されたドキュメントの確認などに使用。

```
{
  "query" : {
    "match_all" : {}
  }
}
```

- match : 典型的なクエリ。match句の中に、フィールド名及び検索キーワードを指定する。

```
{
  "query" : {
    "match" : {
      "message" : "word1 word2"
    }
  }
}
```
- operator : オペレータで一致条件を指定

```
{
  "query" : {
    "match" : {
      "message" : "word1 word2",
      "operator" : "and"
    }
  }
}
```

- minimum_should_match : 検索条件に含む単語の最低件数を指定

```
{
  "query" : {
    "match" : {
      "message" : "word1 word2 word3 word4",
      "minimum_should_match" : "2"
    }
  }
}
```

- sentence : 指定された語順のドキュメントのみを検索する

```
{
  "query" : {
    "match_phrase" : {
      "sentence" : "momotaro killed ogre"
    }
  }
}
```

- query_string : 特殊な記述。検索条件部分にLuceneシンタックスでの検索式を直接記載できる。低レベル検索用のクエリとなります。

```
{
  "query" : {
    "query_string" : {
      "default_field" : "message",
      "query" : "message:Elasticsearch^2 + message:world -user_name:Smith"
    }
  }
}
```
### 基本クエリ - Termベースクエリ
- Termベースクエリは、指定した検索キーワードに完全一致したフィールドを探すときに使うクエリ種別。keyword型のフィールドを探索するために使うクエリ。
----

```
{
  "query" : {
    "terms" : {
      "prefecture" : ["Tokyo", "Kanagawa", "Chiba", "Saitama"]
    }
  }
}
```

- Range : 主に数値型や日付型のフィールドを対象とする
----

```
{
  "query" : {
    "range" : {
      "stock_price" : {
        "gte": "15.00",
        "lte": "25.00"
    }
  }
}
```
日付型のフィールドを対象とした範囲探索では特別な日付計算用の式を使うことができる
```
{
  "query" : {
    "range" : {
      "manufactured_date" : {
        "gte": "now-1w"
    }
  }
}
```
### 複合クエリ - Boolクエリ
- Boolクエリは、これまで紹介してきた基本クエリを複数組み合わせて、複合クエリを構成するための記法
----

```
{
  "query" : {
    "bool" : {
      "must" : [<基本クエリ>],      // AND
      "should" : [<基本クエリ>],    // OR
      "must_not" : [<基本クエリ>],  // NOT
      "filter" : [<基本クエリ>]     
    }
  }
}
```

### クエリ結果のソート
----

```
{
  "query" : {
    "sort" : [
      {"stock_price" : {"order" : "desc"}},
      {"founded" : {"order" : "asc"}},
      "_score"
    ]
  }
}
```

### インポート - csvファイルなど
----
Logstash - .confのfilterに型を記載。

※ 参照資料：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/*, 「Elasticsearch実践ガイド」(惣道 哲也 著)

