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

### インスタンス数の見積もりについて
----
https://aws.typepad.com/sajp/2017/01/get-started-with-amazon-elasticsearch-service-how-many-data-instances-do-i-need.html


### シャードのサイズについて
> ヒント： 小さなシャードは小さなセグメントとなり、結果としてオーバーヘッドが増えます。そのため、平均シャードサイズは最小で数GB、最大で数十GBに保つようにしましょう。時間ベースのデータを使用するケースでは、シャードサイズを20GBから40GBにするのが一般的です。
----
https://www.elastic.co/jp/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster

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

### Analyzer
- Elasticsearchでは、全文検索を行うために文章を単語の単位に分割する処理機能を、Analyzerと呼ぶ。Elasticsearchの標準的なAnalyzerでは、日本語の形態素解析処理が行えないため、OSSの日本語形態素解析ソフトウェアkuromojiのプラグインを追加インストールする必要がある。
----

- Analyzerの指定を含んだマッピング定義の作成例
```
curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d
'
{
  "mappings" : {
    "my_type" : {
      "properties" : {
        "blog_message" : {
          "type": "text",
          "analyzer": "standard"
        }
      }
    }
  }
}
'

```
- Analyzerの構成要素
  - Char filter : 入力されたテキストを前処理するためのフィルタ機能
  - Tokenizer : Char filterのsyつうりょくてきすとを入力として、テキストの分割処理を行い、トークン（単語）の列を生成する。
  - Token filter : Tokenizeが分割したトークンに対して、トークン単位での変換処理を行うためのフィルタ機能

- Analyzer構成をカスタム定義できる

- Analyzerの動作確認の例
```
curl -XPOST 'http://localhost:9200/_analyze_' -H 'Content-Type: application/json' -d
'
{
  "analyzer" : "standard",
  "text" : "Hello Elasticsearch" 
}
'

```

### Aggregation
- 検索結果の集合に対して、分類や集計を行うことができる機能
----

### Aggregationの構成
- 分類や集計を行うための多くのタイプをカテゴリ分けすると４つに分類できる。個々のタイプをうまく組み合わせることで非常に柔軟な分析が行える。
----

- Metrics : 分類された各グループに対して、最小、最大、平均などの統計値を計算する。書籍のジャンル別に件数をカウントする、平均価格を集計するなど。

- Buckets : ドキュメントをそのフィールド値に基づいてグループ化するための分類方法を表す。書籍のジャンルを表すフィールドを使ってジャンル名ごとのグループに分類するなど。

- Pipeline : Bucketsなどの他のAggregationの結果を用いてさらに集計をする機能。月別の平均株価を分類・集計し、さらに前月からの差分を計算する場合など

- Matrix : 複数のフィールドの値を対象に相関や共分散などの統計値を計算することができる機能。


### Aggregationのクエリ記法
----
 - 基本的なクエリ記法
 * 計算した統計量の数値だけ知りたい場合、size : 0 を指定するとレスポンスが簡素化される。

```
{
  "aggregations(aggs)" : {
    "<aggregation_name>" : {
      "<aggregation_type>" : {
        <aggregation_body>
    }
    }
  }
}

```

 - avg Metricsクエリ記法
```
curl -XPOST 'http://localhost:9200/employee/_search_' -H 'Content-Type: application/json' -d
'
{
  "size" : 0,
  "query" : {
    "match" : {
      "office" : "Otemachi"
    }
  },
  "aggs" : {
    "avg_salary" : {
      "avg" : {
        "field" : "salary"
      }
    }
  }
}
'

```

 - min, max Metricsクエリ記法
```
curl -XPOST 'http://localhost:9200/employee/_search_' -H 'Content-Type: application/json' -d
'
{
  "size" : 0,
  "query" : {
    "term" : {
      "group" : "HR section"
    }
  },
  "aggs" : {
    "min_salary" : {
      "min" : {
        "field" : "salary"
      }
    },
    "max_salary" : {
      "max" : {
        "field" : "salary"
      }
    }
  }
}
'

```

 - canrdinality Metricsクエリ記法
 * ElasticsearchではHyperLogLog++と呼ばれるハッシュ値を用いたアルゴリズムを使って、概算値が計算されるようになっている
```
curl -XPOST 'http://localhost:9200/employee/_search_' -H 'Content-Type: application/json' -d
'
{
  "size" : 0,
  "aggs" : {
    "unique_user" : {
      "cardinality" : {
        "field" : "user_name.keyword"
      }
    }
  }
}
'

```

- stats Metricsクエリ記法
 * count,min,max,avg,sumの統計値がまとめて返される
```
curl -XPOST 'http://localhost:9200/employee/_search_' -H 'Content-Type: application/json' -d
'
{
  "size" : 0,
  "aggs" : {
    "unique_user" : {
      "stats" : {
        "field" : "user_name.keyword"
      }
    }
  }
}
'

```

### Bucketsの定義方法
----

### インポート - csvファイルなど
----
Kibana - Indexを指定してcsvファイルを取り込む機能がある。実験的機能。
Logstash - .confにinput,filter,outputの情報を記述。
embulk - fluentdのバッチ版。logstashと同様にymlに設定を記述する必要がある。

※ 参照資料：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/*, 「Elasticsearch実践ガイド」(惣道 哲也 著)

※「Elasticsearch実践ガイド」(惣道 哲也 著)内のソフトウェア環境
Elasticsearch 6.2.1
Kibana 6.2.1
Loagstash 6.2.1
Packetbeats 6.2.1
X-Pack 6.2.1
curator 5.4.1

