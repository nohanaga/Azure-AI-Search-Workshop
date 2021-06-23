# はじめに
組織内に貯まっている大量な構造化・非構造化データから、新たな価値を見出すためのフルマネージド全文検索サービスである [Azure Cognitive Search](https://www.youtube.com/watch?v=jOzA48ZDyC4) を使えば、誰でも簡単に AI 搭載検索エンジンを開発することができます。今回は Azure Cognitive Search の豊富な検索機能を使いこなすためのハンズオンをご紹介します。

# 目次
1. [豊富な検索機能を使いこなす](#豊富な検索機能を使いこなす)
1. [前提条件](#前提条件)
1. [選択($select)](#選択select)
1. [ソート($orderby)](#ソートorderby)
1. [フィルター($filter)](#フィルターfilter)
1. [ファセット(facet)](#ファセットfacet)
1. [まとめ](#まとめ)  


# 豊富な検索機能を使いこなす
Azure Cognitive Search には、フリーテキスト検索から高度に指定されたクエリ パターンまで、さまざまなシナリオをサポートする豊富なクエリ言語が用意されています。このハンズオンでは、クエリ要求と、作成できるクエリの種類について説明します。

検索クエリの発行は、REST API で行います。検索クエリの送信には前回導入した Postman を使ってもよいですし、今回のハンズオン用に私が作成したクエリテスター GUI を利用することもできます。

本ハンズオンでは、Simple Cognitive Search Tester を使用します。

# 前提条件

 - [フルテキスト検索](FullTextSearch.md) ハンズオンを修了済みであること
 - Simple Cognitive Search Tester か Postman が導入済みであること

# 選択($select)
結果セットに含めるコンマ区切りのフィールドの一覧。この句には、取得可能としてマークされているフィールドのみを含めることができます。指定しない場合、または * に設定した場合は、スキーマで取得可能とマークされているすべてのフィールドが含まれます。

$select パラメータに以下のように指定します。

```
metadata_storage_name
```
結果の value 配列に @search.score と metadata_storage_name しか含まれなくなります。Simple Cognitive Search Tester をお使いの場合、右側タブの「Response Json」に REST API の結果が表示されます。

[Azure Cognitive Search での OData $select 構文](https://docs.microsoft.com/ja-jp/azure/search/search-query-odata-select)

# ソート($orderby)
評価や場所などの値によって結果を並べ替える場合に使用します。 それ以外の場合、既定では、関連性スコア（@search.score）を使用して結果が順位付けされます。 このパラメーターの候補とするために、このフィールドには 並べ替え可能 の属性を付ける必要があります。

$orderby パラメータに以下のように指定します。

```
metadata_storage_last_modified desc
```
ドキュメントの最終更新が新しいもの順にソートされます。

[Azure Cognitive Search での OData $orderby 構文](https://docs.microsoft.com/ja-jp/azure/search/search-query-odata-orderby)

# フィルター($filter)
フィルター構文は、単独で使用することも、"search" と一緒に使用することもできる OData 式です。一緒に使用した場合、"filter" が最初にインデックス全体に適用され、次にフィルター処理の結果に対して検索が実行されます。フィルターはクエリのパフォーマンス向上に役立つ手法です。フィルターを使うと、検索クエリで処理が必要なドキュメントの数が減ります。フルテキスト検索とは異なり、フィルター値または式では厳密な一致のみが返されます。

$filter パラメータに以下のように指定します。

## 文字列のフィルター

```
metadata_storage_file_extension eq '.pdf'
```
ドキュメントの拡張子が PDF のものを検索します。


## 数値フィルター

```
metadata_storage_size lt 102400
```
ドキュメントのファイルサイズが 102,400 byte より小さいものを検索します。

## 日付フィルター

```
metadata_storage_last_modified gt 2021-06-23T00:00:00Z
```
ドキュメントの最終更新が 2021-06-23T00:00:00Z 以降のファイルを検索します。

[Azure Cognitive Search での OData $filter 構文](https://docs.microsoft.com/ja-jp/azure/search/search-query-odata-filter)

## コレクション型のフィルタ
Collection(Edm.String) タイプのフィールドの中から、一致する値を検索したい場合は以下の構文を使用します。
```
imageTags/any(t: t eq 'person')
```
imageTags フィールドに person を含むドキュメントを検索します。

[Azure Cognitive Search での OData コレクション演算子 - any と all](https://docs.microsoft.com/ja-jp/azure/search/search-query-odata-collection-operators)


# ファセット(facet)
ファセット ナビゲーションは、検索アプリケーションで自律型のドリルダウン ナビゲーションを提供するフィルター処理メカニズムです。 「ファセット ナビゲーション」という用語は聞き慣れないかもしれませんが、気づかずに使っていることもあります。 次の例に示すように、ファセット ナビゲーションは結果のフィルター処理に使用されるカテゴリです。

<img src="./media/search/011.jpg" />

ファセットは検索結果に応じて動的に生成されます。

facet パラメータに以下のように指定します。

```
"keyphrases,count:5", "organizations,count:5", "locations,count:5"
```

上記はファセットとして、keyphrases、organizations、locations フィールドから生成し、それぞれ上位 5 位を出力します。"count" の他に有効な値は、"sort"、"values"、"interval"、および "timeoffset" です。

```
"metadata_storage_size,interval:10000"
```

このように指定すると、ドキュメントのファイルサイズを 10,000 バイトごとに区分サマリーして表示します。
より詳細な解説は、[クエリ パラメーター](https://docs.microsoft.com/ja-jp/rest/api/searchservice/Search-Documents#query-parameters)の facet 項目を参照してください。

[Azure Cognitive Search へのファセット ナビゲーションの実装方法](https://docs.microsoft.com/ja-jp/azure/search/search-faceted-navigation)









# まとめ
本ハンズオンでは、Simple クエリパーサーと Full Lucene クエリ パーサーの違いを理解していただき、それぞれで可能な検索方法について実践しました。次回は応用的な検索方法として、フィルターやファセット、並び替えなどを解説します。