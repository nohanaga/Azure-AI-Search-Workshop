# はじめに
組織内に貯まっている大量な構造化・非構造化データから、新たな価値を見出すためのフルマネージド全文検索サービスである [Azure Cognitive Search](https://www.youtube.com/watch?v=jOzA48ZDyC4) を使えば、誰でも簡単に AI 搭載検索エンジンを開発することができます。今回は Azure Cognitive Search の検索機能をハンズオン形式でご紹介します。

# 目次
1. [Azure Cognitive Search とは](#Azure-Cognitive-Search-とは)

# 検索クエリーハンズオン
Azure Cognitive Search には、フリーテキスト検索から高度に指定されたクエリ パターンまで、さまざまなシナリオをサポートする豊富なクエリ言語が用意されています。このハンズオンでは、クエリ要求と、作成できるクエリの種類について説明します。

検索クエリの発行は、REST API で行います。検索クエリの送信には前回導入した Postman を使ってもよいですし、今回のハンズオン用に私が作成したクエリテスター GUI を利用することもできます。

# 事前準備

## Simple Cognitive Search Tester のダウンロード
Simple Cognitive Search Tester は、今回のハンズオン用に私が用意したクエリテスト用の HTML ベース GUI です。[こちら](https://github.com/nohanaga/Azure-Cognitive-Search-Workshop/raw/89092a48ec9017db84210d3c485b4744d185843b/gui/simple-cognitive-search-tester.zip)からダウンロードできます。ダウンロードしたら、**simple-cognitive-search-tester.zip** ファイルを任意のフォルダに解凍してください。

## 接続情報の設定

simple-cognitive-search-tester ディレクトリ内に移動し、**settings.html** をブラウザで開きます。

<img src="./media/search/002.jpg" />

検索クエリの実行に必要な、以下の情報を入力して「Save」をクリックします。入力した接続情報は、ブラウザの [localStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage) に保存されます。

* `search_service`: Azure Cognitive Search サービスリソースの名前。検索対象の検索サービス名を設定します。
* `index_name`: 検索インデックスの名前。検索対象のインデックス名を指定します。
* `querykey`: Azure Cognitive Search サービスの API キー。検索クエリ用途のみですので、クエリキーのほうを使用します。これは Azure Portal の検索サービスの「設定メニュ→キー」からコピーします。

**注意**：localStorage に API キーを保管するのはセキュリティ上問題があります。今回一時的な使用のためだけに用意しています。必ずデモ終了後、Delete ボタンを押して削除してください。localStorage に API キーを保管したくない方は、各検索 html ページのソースコードの接続情報変数を直接編集してください。


# 1.フルテキスト検索

## 1.1.単純なクエリ パーサー (フルテキスト検索に最適) 
queryType にはパーサーを設定します。設定できるのは、既定の単純なクエリ パーサー (フル テキスト検索に最適) または、正規表現、近接検索、ファジー検索、ワイルドカード検索など高度なクエリ構成で使用される 完全な Lucene クエリ パーサーです。