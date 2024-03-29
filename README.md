---
marp: true
size: 16:9
paginate: true
headingDivider: 2
header: wikidata を簡単なデータベースとして使う
footer: © 2022 wonox
---

# wikidata を簡単なデータベースとして使う

## Query のサンプル

library_wikidata.ipynb には以下の Query サンプルを置いた

- 機関リポジトリ一覧
- 大学図書館一覧
- 日本の学術雑誌
- 日本の大学の Q 番号と rorID の対照表
- 紀要一覧
- 日本の大学図書館特殊文庫一覧

## wikidata とは

- ウィキデータ（英: Wikidata）はウィキメディア財団が提供する共同編集型のデータベース（知識基盤）
- トピック、概念、オブジェクトなどを表す項目に焦点を当てたドキュメント指向データベース
- 各項目は、「QID」と呼ばれる文字 Q が先頭に付いた番号で一意に識別される。たとえば「政治」は Q7163 である。これにより、項目に必要な基本情報を言語を問わずに入力することができる。

## データの例

![w:600](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ae/Datamodel_in_Wikidata.svg/660px-Datamodel_in_Wikidata.svg.png)

https://www.wikidata.org/wiki/Wikidata:Introduction/ja

情報の科学と技術 特集：「RDF と SPARQL ～検索とデータ可視化」
2020 年 70 巻 8 号 https://www.jstage.jst.go.jp/browse/jkg/70/8/_contents/-char/ja

## Query の例

```
SELECT ?item ?itemLabel ?isil
WHERE {
?item wdt:P31 wd:Q856234;
    wdt:P17 wd:Q17.
    OPTIONAL {?item wdt:P791 ?isil.}
SERVICE wikibase:label { bd:serviceParam wikibase:language "ja,en".}
}
ORDER BY(?itemLabel)
```

分類 (P31) が 大学図書館 (Q856234) の ?item
; は、次の主語（この場合?item）を省略できる。
国 (P17) が 日本 (Q17)
オプションで、?item が ISIL 識別子 (P791) を持っていれば ?isil に代入

## Wikidata Query Service

- クエリを簡単に試すことができる。
  - https://query.wikidata.org/
- 上記の例：https://w.wiki/5dUk
- 乃木坂 46: https://w.wiki/5dhw

## Python から SPARQL

```python
# 日本の大学図書館
import datetime
import pandas as pd
from SPARQLWrapper import SPARQLWrapper, JSON
sparql_wikidata = SPARQLWrapper(
    "https://query.wikidata.org/sparql", returnFormat='json')
sparql_wikidata.setQuery("""
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?item ?itemLabel ?isni ?isil ?venue ?address ?geocode
WHERE {
?item wdt:P31 wd:Q856234;
    wdt:P17 wd:Q17.
    OPTIONAL {?item wdt:P213 ?isni.}
    OPTIONAL {?item wdt:P791 ?isil.}
    OPTIONAL {?item wdt:P131 ?venue.}
    OPTIONAL {?item wdt:P6375 ?address.}
    OPTIONAL {?item wdt:P625 ?geocode.}
SERVICE wikibase:label { bd:serviceParam wikibase:language "ja,de,en".}
}
ORDER BY(?itemLabel)
""")
sparql_wikidata.setReturnFormat(JSON)
library_results = sparql_wikidata.query().convert()

# pandasでJSON文字列・ファイルを読み込み
library_results_df = pd.io.json.json_normalize(
    library_results['results']['bindings'])
# EXCELに書き出し
now = datetime.datetime.now()
filename = './library_results_' + now.strftime('%Y%m%d_%H%M%S') + '.xlsx'
with pd.ExcelWriter(filename) as writer:
    library_results_df.to_excel(writer, sheet_name=now.strftime('%Y%m%d'))
```

## まとめ

- wikidata から簡単な SPARQL で、データを取り出せる
- データは、csv や EXCEL で取り出せる
- PANDAS で処理できる
- （やろうと思えば）他の Linked Data と連携できる（そのための Linked Data）
- お手軽なデータ置き場として重宝

ソース：　https://github.com/wonox/sparqltraining
