# はじめに
組織内に貯まっている大量な構造化・非構造化データから、新たな価値を見出すためのフルマネージド全文検索サービスである [Azure Cognitive Search](https://www.youtube.com/watch?v=jOzA48ZDyC4) を使えば、誰でも簡単に AI 搭載検索エンジンを開発することができます。今回はノーコード、画面上の操作だけで簡単に検索インデックスを作成する手順をハンズオン形式でご紹介します。

# 目次
1. [Azure Cognitive Search とは](#Azure-Cognitive-Search-とは)
1. [検索サービスの作成](#検索サービスの作成)
1. [Azure Blob Storage の作成](#azure-blob-storage-の作成)
1. [インデックスの作成](#インデックスの作成)
    1. [データに接続](#1-データに接続)
    1. [コグニティブ スキルの追加](#2-コグニティブ-スキルの追加)
    1. [インデックスのカスタマイズ](#3-インデックスのカスタマイズ)
1. [インデクサーの作成](#4-インデクサーの作成)
1. [インデクサーの実行](#インデクサーの実行)
1. [スキルセットの修正](#スキルセットの修正)
1. [デモアプリの作成](#デモアプリの作成)
1. [参考リンク](#参考リンク)


# 検索サービスの作成
[Azure Portal](https://portal.azure.com/) にログインし、新規でリソースグループを作成します。
作成したリソースグループの中に、検索サービスを追加します。マーケットプレイスで、「Azure Cognitive Search」と検索してください。

<img src="./media/001a.jpg" width=450 />

新しい検索サービスウィザードで、必要事項を入力します。価格レベルでは、必要なプランを選択します。こちらの[料金表](https://azure.microsoft.com/pricing/details/search/)も参考にしてください。無料プランで始めることもできます。

<img src="./media/001.jpg" />

デプロイが完了し、作成された検索サービスをクリックすると、以下のようなポータル画面が表示されます。

<img src="./media/002.jpg" />

このポータル画面上でインデックス、インデクサー、データソース、スキルセットの作成や編集を行うことができます。

<BR>

# Azure Blob Storage の作成

ここで一旦リソースグループに戻り、検索対象のデータを保管するためのストレージアカウントを追加します。
マーケットプレイスで、「ストレージ」と入れて検索してください。

<img src="./media/006.jpg" width=450 />

今回は、ドキュメントファイルや画像データを保管できるシンプルな Blob Storage があればよいので、以下のように設定します。

<img src="./media/003a.jpg" />

ストレージアカウントのデプロイが完了しましたら、ストレージアカウントの左メニューから「コンテナー」を選択し、以下のように新しいコンテナーを作成します。

<img src="./media/004.jpg" />

作成したコンテナーの中に入り、検索したいファイルをアップロードします。画像や PDF、ワード、エクセルファイルなどをこちらにアップロードしてください。ファイルが大量にある場合は、[Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) を使ってアップロードすることもできます。
サポートしているファイルフォーマットの一覧は[こちら](https://docs.microsoft.com/azure/search/search-blob-storage-integration#supported-content-types)を参照してください。

今回のワークショップ用に [Github](https://github.com/nohanaga/Azure-Cognitive-Search-Workshop/tree/main/pdf) に検索用の PDF ファイルを用意してありますので、ローカルにダウンロードして、Blob Storage にアップロードします。

<BR>


# インデックスの作成

それでは、先ほど作成した Blob Storage 内を検索するためのインデックスの作成を行います。

## 1. データに接続

<img src="./media/008.jpg" />

検索サービスのポータルに戻り、ツールバーの「データのインポート」をクリックします。

<img src="./media/009.jpg" />

データのインポートウィザードでは、データソースに「Azure BLOB ストレージ」を選択し、データソース名を入力し、「既存の接続を選択します」リンクをクリックします。

ちなみに Azure Cognitive Search が対応しているデータソースには、Azure Cosmos DB や Azure SQL Database などがあり、2021 年 3 月には、[SharePoint Online (Preview)](https://docs.microsoft.com/azure/search/search-howto-index-sharepoint-online) にも対応しました。サポートしているデータソースの一覧は[こちら](https://docs.microsoft.com/azure/search/search-indexer-overview#supported-data-sources)にあります。

<img src="./media/010.jpg" />

先ほどのデータのインポート画面のコンテナー名に、選択したコンテナーが自動で入力されたことを確認し、「次：コグニティブ スキルを追加します（省略可能）」ボタンをクリックします。

## 2. コグニティブ スキルの追加

ここでは、検索インデックスをより豊かにするための、AI エンリッチメントを追加できます。AI エンリッチメントでは、[Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) の機能が使われるため、基本的に[利用料がかかります](https://docs.microsoft.com/azure/search/cognitive-search-attach-cognitive-services)。ただし今回はハンズオンですので、「Cognitive Services をアタッチする」では設定を行いません。そうすると自動で無料 (制限付きのエンリッチメント) リソースがセットされます。ただしインデクサーごとに、1 日あたり 20 ドキュメントまでのエンリッチに制限されます。 

<img src="./media/010a.jpg" />

今回は、上図のようにチェックしてください。サンプル画像から、OCR を使って文字を読み取り、その文字に対して、エンティティの抽出等のテキスト解析を行います。また写真ファイルをアップロードする場合には、画像認識のスキルを有効化しておくことで画像からメタデータを抽出できます。

チェックが完了したら、「次：対象インデックスをカスタマイズします」をクリックします。

## 3. インデックスのカスタマイズ

<img src="./media/012.jpg" />

対象インデックスのカスタマイズウィザードでは、検索インデックスとして格納するフィールドごとに、取得可能、フィルター可能、ソート可能、ファセット可能、検索可能、アナライザー、サジェスターという一般的な検索インデックスとしての設定を行う必要があります。各機能の説明は冒頭の動画でするとして、今回は上図のようにチェックしてください。

アナライザーにはデフォルトで、標準 Lucene アナライザーがセットされていますが、これは西洋語用のもので日本語の解析に対応していませんので、日本語の解析が必要なフィールドには、**日本語アナライザー**を設定する必要があります。日本語のアナライザーには Lucene アナライザーと、Microsoft アナライザーの [2 種類](https://docs.microsoft.com/azure/search/index-add-language-analyzers#comparing-lucene-and-microsoft-analyzers)が用意されていますが、それぞれトークナイズの手法に違いがあるので、要件によって選択するようにしてください。

 * `content`: ドキュメントの本文が格納されます。文字解析可能なドキュメントはここに値が格納されます。中身は日本語として解析される必要があるため、[日本語 Microsoft アナライザー](https://docs.microsoft.com/azure/search/index-add-language-analyzers)をセットします。
 * `metadata_storage_name`: ドキュメントのファイル名です。今回は検索結果に表示するために、取得可能にチェックします。
 * `people`: [人物エンティティ](https://docs.microsoft.com/azure/search/cognitive-search-skill-entity-recognition)が格納されます。日本語で解析できるようにします。
 * `organizations`: [組織エンティティ](https://docs.microsoft.com/azure/search/cognitive-search-skill-entity-recognition)が格納されます。日本語で解析できるようにします。
 * `locations`: [位置エンティティ](https://docs.microsoft.com/azure/search/cognitive-search-skill-entity-recognition)が格納されます。日本語で解析できるようにします。
 * `keyphrases`: [重要語](https://docs.microsoft.com/azure/search/cognitive-search-skill-keyphrases)が格納されます。日本語で解析できるようにします。
 * `language`: [言語検出スキル](https://docs.microsoft.com/azure/search/cognitive-search-skill-language-detection)による言語推定結果が格納されます。ISO 3166-1 alpha-2 の 2 文字の国番号が入るのでアナライザーはデフォルトにしておきます。
 * `merged_content`: ドキュメントの本文と、OCR による読み取り結果をマージした結果が格納されます。日本語で解析できるようにします。Suggester にチェックをいれることで、このフィールドが[オートコンプリートとサジェスト機能](https://docs.microsoft.com/azure/search/index-add-suggesters)対応になります。
 * `text`: [OCR スキル](https://docs.microsoft.com/azure/search/cognitive-search-skill-ocr)の読み取り結果が格納されます。日本語で解析できるようにします。
 * `layoutText`: [OCR スキル](https://docs.microsoft.com/azure/search/cognitive-search-skill-ocr) で抽出されたテキストと､そのテキストが検出された座標を記述した配列が格納されます。日本語で解析できるようにします。
 * `imageTags`: [画像解析](https://docs.microsoft.com/azure/search/cognitive-search-skill-image-analysis)して画像のコンテンツに関係する単語の一覧が格納されます。今回は設定省略します。（日本語にも対応しています）
 * `imageCaption`: [画像解析](https://docs.microsoft.com/azure/search/cognitive-search-skill-image-analysis)して画像のコンテンツを説明するための文が格納されます。今回は設定省略します。（日本語にも対応しています）

 フィールドごとに一つ一つ設定していくのが面倒くさいと思った方はご安心ください。これらの設定はすべて JSON 形式でエクスポートできますので、JSON 形式で編集などをして、[Postman](https://www.postman.com/) を使って REST API 経由でインデックスに登録することができます。こちらは別記事にて紹介いたします。

設定が完了しましたら、「次：インデクサーの作成」をクリックしてください。

## 4. インデクサーの作成

最後にインデクサーの作成を行います。インデクサーは、外部データソースから検索サービスの検索インデックスにドキュメントとコンテンツを転送するための、自動化されたワークフローを提供します。ここでは、データをプルする頻度をスケジューリングすることや、検索対象のファイルに対する処理の各種設定を行うことができます。

<img src="./media/013.jpg" />

今回は、`metadata_storage_path` をドキュメントキーとしているため、`Base-64 エンコード キー` にチェックを入れておきます。
また、スケジュール設定はせず、デフォルトの `1度` にセットしておきます。

設定が完了したら、「送信」ボタンをクリックします。

<br>

# インデクサーの実行

さきほどインデクサーの実行スケジュールを `1度` にしておいたため、作成直後に自動的に 1 度だけ実行されます。

<img src="./media/014.jpg" />

「インデクサー」タブの対象インデクサーのステータスが `実行中` となっていると思います。これが `成功` となればインデキシング完了です。

<br>

# インデックスの検索

作成したインデックスの中身は、ポータル上で検索することができます。

<img src="./media/015.jpg" />

「インデックス」タブをクリックし、インデックス名を選択します。

<img src="./media/015a.jpg" />

すると検索エクスプローラー画面が表示されます。ここで、検索クエリーのテストや、実行結果の確認をすることができます。
ひとまずクエリ文字列に何も入れずに「検索」ボタンをクリックしてみてください。全件が検索されます。

<br>

# スキルセットの修正

OCR スキルはデフォルトで英語の設定になっているため、そのまま検索しようとするとうまく検索できません。OCR を日本語に対応させる場合は一度ポータルに戻り、「スキルセット」タブをクリックし、スキルセット名を選択してください。

<img src="./media/016.jpg" />

スキルセットは、各スキルの定義とその実行順を JSON 形式で定義したものです。ポータル画面から、閲覧と直接編集を行えます。

<img src="./media/017.jpg" />

以下のように、`#Microsoft.Skills.Vision.OcrSkill` の部分の、`defaultLanguageCode` を `ja` に変更し、「保存」ボタンをクリックします。もし OCR したいドキュメントの言語がこの時点で不明の場合、`unk` と設定することで、言語が自動検出されます。

```json
    {
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
       ...
      "defaultLanguageCode": "ja",
       ...
    }
```

保存が完了したら、今度はポータルからインデクサーを開き、「リセット」ボタンを押してから「実行」ボタンを押します。
[リセット](https://docs.microsoft.com/azure/search/search-howto-run-reset-indexers#reset-an-indexer)を行うことで、一度インデキシングされたデータをクリアして、再度インデキシングすることができるようになります。

<img src="./media/018.jpg" />

<br><br>

# デモアプリの作成

Azure Cognitive Search の機能は基本的に REST API での提供となりますが、デモのために簡易的な Web 検索フロントエンドを作成してくれる機能があります。検索エクスプローラー画面の上にある「デモ アプリの作成」ボタンをクリックしてください。

<img src="./media/019.jpg" />


デモ アプリの作成ウィザードでは、検索画面に表示するフィールドの設定を行うことができます。
まずは検索結果画面に表示したいフィールドを選択して「次へ」をクリックします。

<img src="./media/020.jpg" />

サイドバー設定では、検索画面の左側に表示されるフィルター項目を設定することができます。この設定を行えば、[ファセットフィルター](https://docs.microsoft.com/azure/search/search-filters-facets)機能のデモをすることができます。

<img src="./media/021.jpg" />

[インデックスのカスタマイズ](#3-インデックスのカスタマイズ)セクションで、Suggester の設定を行っていたので、この画面でサジェスト対象のフィールド `merged_content` を選択します。

最後に、「デモ アプリの作成」ボタンをクリックすれば、デモ用の HTML ファイルがダウンロードできます。

<img src="./media/022.jpg" />

ローカルでダウンロードした HTML ファイルをブラウザで開き、上部の検索ボックスにキーワードを入れると、このように検索結果が表示されます。簡易的ですが、検索デモとして使うことができます。左ペインは、検索結果に応じて動的に生成される[ファセットフィルター](https://docs.microsoft.com/azure/search/search-filters-facets)です。検索ボックスに何か単語を入力すると、[サジェスト機能](https://docs.microsoft.com/azure/search/search-add-autocomplete-suggestions#suggestions)によるドロップダウンが表示されます。

※注意：HTML ファイル内に接続情報を含んでいるため、このファイルをそのまま運用環境に用いることはできません。
