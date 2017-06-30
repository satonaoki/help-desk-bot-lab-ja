# 演習 4: Azure Search と Cosmos DB によるヘルプ デスク ナレッジ ベースの実装(Node.js)

## 概要

ボットは、ユーザーが大量のコンテンツをナビゲートする支援を行い、ユーザーのためにデータ駆動型の検索エクスペリエンスを実現することもできます。この演習では、検索機能をボットに追加し、ユーザーがナレッジベースを検索する支援を行う方法について学習します。これを行うには、Azure Cosmos DB に保管されている KB の記事のインデックスを作成する Azure Search サービスにボットを接続します。

[Azure Cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/)
は、マイクロソフトが提供する、ミッションクリティカルなアプリケーション向けのグローバル分散型マルチ モデル データベースサービスです。Azure Cosmos DB は、さまざまなデータモデルをサポートします。この演習では、Azure Cosmos DB の DocumentDB API を使用します。これを使用することで、ナレッジ ベースの記事を JSON ドキュメントとして保管できます。

[Azure Search](https://azure.microsoft.com/en-us/services/search/) は、カスタム
アプリケーションで充実した検索エクスペリエンスを実現する完全管理型のクラウド検索サービスです。Azure Search は、さまざまなソース (Azure SQL DB、Cosmos DB、BLOB ストレージ、テーブルストレージ) のコンテンツのインデックスを作成でき、その他のデータソースに対応する「プッシュ型」のインデックス作成をサポートします。また、PDF、Office ドキュメント、および非構造化データが含まれるその他の形式のドキュメントを開くことができます。コンテンツカタログが Azure Search インデックスに取り込まれることで、ボットのダイアログからクエリを行えるようになります。

次の図は、コンポーネントどうしがどのようなやりとりをするかを示したものです。

![exercise4-diagram](media/6002c05ae7223b25b34d254ca6a7e53c.png)

 **注:** このラボでは Azure Search と Azure Cosmos DB を使用しますが、どの検索エンジンおよびバッキングストアを使用してももちろんかまいません。

[こちらのフォルダー](./exercise4-KnowledgeBase)内には、この演習のステップの完了結果として得られるコードを含むソリューションが入っています。このソリューションは、演習を進めるにあたってさらにヒントが必要な場合に、ガイダンスとして使用できます。これを使用には、まず `npm install` を実行し、`.env` ファイルで LUIS モデルの値、および Azure Search インデックスの名前とキーを指定しておく必要があることを覚えておいてください。

**前の演習** [演習3 言語理解の機能 (LUIS) によるボットのスマート化](./exercise3-LuisDialog.md)

## 前提条件

この演習を完了するには、以下のソフトウェアが必要です。

* [最新の Node.js と NPM](https://nodejs.org/en/download)
* [Visual Studio Code](https://code.visualstudio.com/download) (推奨) や Visual Studio 2017 Community 以上などのコード エディター
* [Azure](https://azureinfo.microsoft.com/us-freetrial.html?cr_cc=200744395&wt.mc_id=usdx_evan_events_reg_dev_0_iottour_0_0) サブスクリプション
* [Bot Framework Emulator](https://emulator.botframework.com/) (`en-US` ロケールで構成されていることを確認してください)
* [LUIS ポータル](https://www.luis.ai/)のアカウント

* [このドキュメントの原文](https://github.com/GeekTrainer/help-desk-bot-lab/blob/master/Node/exercise4-KnowledgeBase.md)

<!-- ドライラン時に補足追加 -->
> 補足: このドキュメントの動作確認はそれぞれ下記バージョンにて実施しています。
>* node-v6.11.0-x64
>* botframework-emulator-Setup-3.5.29
>* Windows 10 Version 1703 (Build 15063.413)
> package.json ファイルのバージョンが下記になっていることを確認してください。

```json
 "dependencies": {
    "botbuilder": "^3.8.4",
    "dotenv": "^4.0.0",
    "restify": "^4.3.0"
  };
```

もし、なっていない場合は 下記のコマンドでコンポーネントの再インストールを行います。
```
    npm install --save botbuilder@3.8.4 restify@4.3.0 dotenv@4.0.0 
    npm install -g nodemon
```

<!-- ドライラン時に補足追加 -->


## タスク 1: Cosmos DB サービスを作成し、ナレッジ ベースをアップロードする

このタスクでは、Cosmos DB データベースを作成し、ボットによって使用されるいくつかのドキュメントをアップロードします。Azure Cosmos DB の仕組みについてよくわからない場合は、[こちらのドキュメント](https://docs.microsoft.com/en-us/azure/cosmos-db/)を参照してください。

1.  [Azure ポータル](https://portal.azure.com/)にアクセスしてサインインします。左側のバーにある **新規** ボタン (![exercise4-new](media/fce1f762ced67b48945c5d79ec825659.png)) をクリックし、次に **Database** をクリックして、**Azure Cosmos DB** を選択します。

2.  ダイアログ ボックスで一意のアカウント ID (例: _help-desk-bot_) を入力し、*API*で **SQL (DocumentDB)** を選択します。新しいリソースグループ名を入力し、**作成** をクリックします。

   ![exercise4-createdocumentdb](./media/4-3.png)
   <!--- ドライラン補足--->
   **補足** この演習で指定するデーターベースIDは全世界でユニークになる必要があります。すでに help-desk-bot は使用されていますので、ユニークになるように任意の文字列を付加するなどしてください。
   なお、Document DB は東日本、西日本の両方のリージョンでもご利用可能ですのでお好みの場所に作成してください。

3.  展開が完了するまで待ちます。先ほど作成した *Cosmos DB アカウント*を開き、**概要** セクションに移動します。**コレクションの追加** ボタンをクリックします。ダイアログ ボックスの *コレクション ID* で *knowledge-base* と入力し、*ストレージ容量* で *固定（10 GB)* を選択して、データベース ID として *knowledge-base-db* と入力します。*OK* をクリックします。

   ![exercise4-documentdb-addcollection](./media/4-4.png)
    <!--- ドライラン補足　-->
    **補足**：初期スループット容量、RU/分、パーティション キー　についてはそのままの値を使用します。

4.  左側で **ドキュメント エクスプローラー** を選択し、次に **アップロード** ボタンをクリックします。

5.  開いたウィンドウで、[assets/kb](../assets/kb) フォルダーのファイルをすべて選択します。各ファイルは、ナレッジ ベースの 1 つの記事に相当します。**アップロード** をクリックします。ブラウザーを閉じないでください。

   ![exercise4-documentdb-uploadfiles](./media/4-5.png)

> **注:** 記事の "ドキュメント" にはそれぞれ 3 つのフィールド (タイトル、カテゴリ、およびテキスト) が含まれています。

## タスク 2: Azure Search サービスを作成する

このタスクでは、Cosmos DB にアップロードされたコンテンツのインデックスを作成するための Azure Search サービスを作成します。Azure Search は、検索に最適化されたインデックス付きのデータのコピーを作成します。

1.  Azure ポータルの左側のバーにある **新規** ボタン (![exercise4-new](media/fce1f762ced67b48945c5d79ec825659.png)) をクリックし、次に **Web + モバイル** をクリックして、**Azure Search** を選択し、**作成** ボタンをクリックします。一意の URL (例:_help-desk-bot-search_) を入力します。Cosmos DB で使用したものと同じリソースグループを選択します。*価格レベル* を **Free** に変更し、**作成** をクリックします。

   ![exercise4-createsearchservice](./media/4-6.png)

  <!--- ドライラン補足--->
   **補足** この演習で指定するデーターベースIDは全世界でユニークになる必要があります。すでに help-desk-bot は使用されていますので、ユニークになるように任意の文字列を付加するなどしてください。
   なお、Document DB は東日本、西日本の両方のリージョンでもご利用可能ですのでお好みの場所に作成してください。

2.  サービスのプロビジョニング後、*概要* に移動してから **データのインポート** ボタン (![exercise4-import-data-button](media/18699592dc18fddc6fb9e63e78a5591f.png)) をクリックします。

3.  **データに接続します** ボタン、**DocumentDB** の順にクリックします。データソース名として_knowledge-base-datasource_と入力します。先ほど作成した Cosmos DB のアカウント、データベース、およびコレクションを選択します。**OK** をクリックします。

   ![exercise4-azuresearch-createdatasourcex](./media/4-8.png)

4.  **対象インデックスをカスタマイズします** ボタンをクリックします。**インデックス名** で _knowledge-base-index_ と入力します。インデックスの定義が以下の図と一致するように各列のチェックボックスを更新します。**OK** をクリックします。
    
カテゴリ フィールドの **フィルター可能** と **ファセット可能** にチェックマークが付いていることを確認します。これにより、カテゴリが一致するすべての記事を取得できると共に、各カテゴリの記事の数も取得できるようになります。これは、Azure Search の専門用語で **ファセット ナビゲーション** と呼ばれます。ファセット ナビゲーションは、ユーザーのガイドを支援する強力なボットのユーザー エクスペリエンス ツールです。

   ![exercise4-faq-index-facets-matrix](./media/4-9.png)

 **注:** インデックスの詳細については、[こちらの記事](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-an-index)を参照してください。

5.  最後に **インデクサー - データのインポート** ボタンをクリックします。*名前*で**knowledge-base-indexer**と入力します。*スケジュール* で **1 度** が選択されていることを確認します。**OK** をクリックします。

   ![exercise4-azuresearch-createindexer](./media/4-10.png)

 <!--- ドライラン補足--->
   **補足** この演習では詳細は設定しません。

6.  再び **OK** をクリックし、*データのインポート* ダイアログを閉じます。

7.  左側で **キー** をクリックし、次に **クエリ キーの管理** をクリックします。次のタスクで使用するために、既定の Azure Search キー(**\<空\>** という名前で示されています) を保存します。

   ![exercise4-azuresearch-managekeys](./media/4-11.png)

> **注:** クエリ キーは、管理キーとは異なり、Search インデックスの読み取り専用の操作 (例: ID によるドキュメントのクエリと検索) でしか使用できません。プライマリ管理キーとセカンダリ管理キーは、すべての操作 (例: サービスの管理、インデックス、インデクサー、データ ソースの作成と削除) を行える完全な権限を付与します。

## タスク 3: ExploreKnowledgeBase インテントが含まれるように LUIS モデルを更新する

このタスクでは、ナレッジ ベースを検索するために LUIS に新しいインテントを追加します。

1.  [LUIS ポータル](https://www.luis.ai/) にサインインします。演習 3 で作成したアプリを編集します。

2.  左側のメニューで **Intents** をクリックし、次に **Add Intent** ボタンをクリックします。インテント名として _ExploreKnowledgeBase_ と入力してから、以下の発話を追加します。

* _explore knowledge base_
* _explore hardware articles_
* _find me articles about hardware_

   ![exercise4-new-intent](./media/4-12.png)

3.  **Save** をクリックします。

4.  左側にある **Publish App** リンクをクリックします。**Train** ボタンをクリックし、完了したら **Publish** ボタンをクリックします。

## タスク 4: Azure Search API を呼び出せるようにボットを更新する 

このタスクでは、先ほど作成したインテントに対応し、*Azure Search* サービスを呼び出すためのダイアログを追加します。

1.  前の演習から得られたアプリを開きます。または、[exercise3-LuisDialog](./exercise3-LuisDialog)
    フォルダーのアプリを使用することもできます。その場合は、`.env` ファイルを編集し、**LUIS\_MODEL\_URL** キーをご使用のモデル URL に置き換えます。

2.  **azureSearchApiClient.js** という名前の新しい空のファイルを追加し、REST API を介して Azure Search からデータを取得する以下のコードを追加します。

```javascript
    const restify = require('restify');

    module.exports = (config) => {
        return (query, callback) => {
            const client = restify.createJsonClient({ url: `https://${config.searchName}.search.windows.net/` });
            var urlPath = `/indexes/${config.indexName}/docs?api-key=${config.searchKey}&api-version=2015-02-28&${query}`;

            client.get(urlPath, (err, request, response, result) => {
                if (!err && response && response.statusCode == 200) {
                    callback(null, result);
                } else {
                    callback(err, null);
                }
            });
        };
    };
```

3.  以下の行を追加し、`.env` ファイルを更新します。*AZURE\_SEARCH\_ACCOUNT* の値に
 Azure Search のアカウント名 (例: _help-desk-bot-search_ ) を指定し、*AZURE\_SEARCH\_KEY* にキー値を指定します。

```javascript
    AZURE_SEARCH_ACCOUNT=
    AZURE_SEARCH_INDEX=knowledge-base-index
    AZURE_SEARCH_KEY=
```

4.  **app.js** の上のセクションに以下のコードを追加し、検索クライアントをインスタンス化します。

```javascript
    const azureSearch = require('./azureSearchApiClient');

    const azureSearchQuery = azureSearch({
        searchName: process.env.AZURE_SEARCH_ACCOUNT,
        indexName: process.env.AZURE_SEARCH_INDEX,
        searchKey: process.env.AZURE_SEARCH_KEY
    });
```
5.  *SubmitTicket* ダイアログの直後に、カテゴリの記事を取得するための *ExploreKnowledgeBase* ダイアログ ハンドラーを追加します。

```javascript
    bot.dialog('ExploreKnowledgeBase', [
        (session, args) => {
            var category = builder.EntityRecognizer.findEntity(args.intent.entities, 'category');
            if (!category) {
                return session.endDialog('Try typing something like _explore hardware_.');
            }
            // search by category
            azureSearchQuery('$filter=' + encodeURIComponent(`category eq '${category.entity}'`), (error, result) => {
                if (error) {
                    console.log(error);
                    session.endDialog('Ooops! Something went wrong while contacting Azure Search. Please try again later.');
                } else {
                    var msg = `These are some articles I\'ve found in the knowledge base for the _'${category.entity}'_ category:`;
                    result.value.forEach((article) => {
                        msg += `\n * ${article.title}`;
                    });
                    session.endDialog(msg);
                }
            });
        }
    ]).triggerAction({
        matches: 'ExploreKnowledgeBase'
    });
```

## タスク 5: この時点でボットをテストする

1.  コンソール (`nodemon app.js`) からアプリを実行し、エミュレーターを開きます。ボットの URL (`http://localhost:3978/api/messages` ) をいつもどおり入力します。

2.  *explore hardware* と入力します。そのカテゴリに属する記事がボットにより一覧表示されることを確認します。他の *category* 値 (例: networking、software ) で試してもかまいません。

   ![exercise4-testbit-explorehardware](./media/4-13.png)

## タスク 6: カテゴリと記事を表示できるようにボットを更新する

このタスクでは、ナレッジベースをカテゴリで検索できるようにボットのコードを更新します。

1.  ユーザーが入力したテキストを使用して単純な検索を行うための以下のダイアログを **app.js** に追加します。この場合、ダイアログは、ユーザーの入力テキスト内で _'search about'_ というフレーズを検出する正規表現によって起動されます。`matches` メソッドが正規表現やレコグナイザーの名前を取ることができるのを確認します。

```javascript
    bot.dialog('SearchKB', [
        (session) => {
            session.sendTyping();
            azureSearchQuery(`search=${encodeURIComponent(session.message.text.substring('search about '.length))}`, (err, result) => {
                if (err) {
                    session.send('Ooops! Something went wrong while contacting Azure Search. Please try again later.');
                    return;
                }
                session.replaceDialog('ShowKBResults', { result, originalText: session.message.text });
            });
        }
    ])
    .triggerAction({
        matches: /^search about (.*)/i
    });
```

> **注:** Azure Search では、`search=...` クエリは、インデックス内のすべての検索可能フィールドの用語を 1 つ以上検索し、Google や Bing などの検索エンジンと同様に機能します。`filter=...` クエリは、インデックス内のすべてのフィルター可能フィールドでブール式を評価します。検索クエリとは異なり、フィルター クエリは、フィールドの正確なコンテンツのマッチングを行います。つまり、文字列フィールドの大文字と小文字が区別されます。

2.  カテゴリを取得し、それらを一覧表示できるように **ExploreKnowledgeBase** ダイアログを置き換えます。`facet=category` クエリを使用し、インデックスのクエリを行って、これが行われることを確認します。これは、すべての記事に適用できるすべての "category filters" をインデックスから取得します (この場合は、software、hardware、networking など)。また、Azure Search は、各ファセットの記事の数を返します。

```javascript
    bot.dialog('ExploreKnowledgeBase', [
        (session, args, next) => {
            var category = builder.EntityRecognizer.findEntity(args.intent.entities, 'category');

            if (!category) {
                // retrieve facets
                azureSearchQuery('facet=category', (error, result) => {
                    if (error) {
                        session.endDialog('Ooops! Something went wrong while contacting Azure Search. Please try again later.');
                    } else {
                        var choices = result['@search.facets'].category.map(item=> `${item.value} (${item.count})`);
                        builder.Prompts.choice(session, 'Let\'s see if I can find something in the knowledge base for you. Which category is your question about?', choices, { listStyle: builder.ListStyle.button });
                    }
                });
            } else {
                if (!session.dialogData.category) {
                    session.dialogData.category = category.entity;
                }

                next();
            }
        },
    ]).triggerAction({
        matches: 'ExploreKnowledgeBase'
    });
```

3.  `$filter=...` クエリを使用して記事カテゴリで検索を行うための 2 つ目のウォーターフォール手順を追加します。

```javascript
    (session, args) => {
        var category;

        if (session.dialogData.category) {
            category = session.dialogData.category;
        } else {
            category = args.response.entity.replace(/\s\([^)]*\)/,'');
        }

        // search by category
        azureSearchQuery('$filter=' + encodeURIComponent(`category eq '${category}'`), (error, result) => {
            if (error) {
                session.endDialog('Ooops! Something went wrong while contacting Azure Search. Please try again later.');
            } else {
                session.replaceDialog('ShowKBResults', { result, originalText: category });
            }
        });
    }
```

> **注:** `session.replaceDialog()` メソッドにより、呼び出し元に返さずに、現在のダイアログを終了し、これを新しいものに置き換えることができます。

4.  **DetailsOf** ダイアログを追加するために、**app.js** ファイルの最後に以下のコードを追加します。このダイアログは、タイトルに基づいて特定の記事を取得します(`$filter='title eq ...'` クエリ フィルターを確認します)。

```javascript
    bot.dialog('DetailsOf', [
        (session, args) => {
            var title = session.message.text.substring('show me the article '.length);
            azureSearchQuery('$filter=' + encodeURIComponent(`title eq '${title}'`), (error, result) => {
                if (error || !result.value[0]) {
                    session.endDialog('Sorry, I could not find that article.');
                } else {
                    session.endDialog(result.value[0].text);
                }
            });
        }
    ]).triggerAction({
        matches: /^show me the article (.*)/i
    });
```

> **注:** わかりやすくするために、記事のコンテンツは Azure Search から直接取得されます。しかしながら、本番のシナリオでは、Azure Search はインデックスとしてのみ機能し、記事の全文は Cosmos DB から取得されます。

5.  **ShowKBResults** ダイアログに対応するための以下のダイアログを追加します。このダイアログは、ThumbnailCard のカルーセルを使用して、記事の結果の一覧をユーザーに示します。通常、カードには、1 つの大きな画像、1つ以上のボタン、およびテキストが含まれています。ユーザーに対してリッチなカードを示す方法の詳細については、[こちらの記事](https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-send-rich-cards)を参照してください。

```javascript
    bot.dialog('ShowKBResults', [
        (session, args) => {
            if (args.result.value.length > 0) {
                var msg = new builder.Message(session).attachmentLayout(builder.AttachmentLayout.carousel);
                args.result.value.forEach((faq, i) => {
                    msg.addAttachment(
                        new builder.ThumbnailCard(session)
                            .title(faq.title)
                            .subtitle(`Category: ${faq.category} | Search Score: ${faq['@search.score']}`)
                            .text(faq.text.substring(0, Math.min(faq.text.length, 50) + '...'))
                            .images([builder.CardImage.create(session, 'https://bot-framework.azureedge.net/bot-icons-v1/bot-framework-default-7.png')])
                            .buttons([{ title: 'More details', value: `show me the article ${faq.title}`, type: 'postBack' }])
                    );
                });
                session.send(`These are some articles I\'ve found in the knowledge base for _'${args.originalText}'_, click **More details** to read the full article:`);
                session.endDialog(msg);
            } else {
                session.endDialog(`Sorry, I could not find any results in the knowledge base for _'${args.originalText}'_`);
            }
        }
    ]);
```

 >**注:** ポストバックのアクション タイプは、ボットにメッセージをプライベートでポストするため、そのメッセージがポストされたことは会話内の他の参加者にはわかりません。

6.  最後に、ナレッジ ベースの機能が含まれるように `Help` ダイアログのテキストを更新します。

```javascript
    bot.dialog('Help',
        (session, args, next) => {
            session.endDialog(`I'm the help desk bot and I can help you create a ticket or explore the knowledge base.\n` +
                `You can tell me things like _I need to reset my password_ or _explore hardware articles_.`);
        }
    ).triggerAction({
        matches: 'Help'
    });
```

## タスク 7: エミュレーターからボットをテストする

1.  コンソール (`nodemon app.js`)からアプリを実行し、エミュレーターを開きます。ボットの URL (`http://localhost:3978/api/messages` ) をいつもどおり入力します。

2.  `explore knowledge base`と入力します。Cosmos DB にアップロードした記事カテゴリの一覧、および各カテゴリの記事の数が表示されます。

   ![exercise4-emulator-explorekb2](./media/4-14.png)

3.  一覧表示されたカテゴリのいずれかをクリックすると、そのカテゴリの記事が表示されます。

   ![exercise4-emulator-showkbresults](./media/4-15.png)

4.  記事の **More Details** ボタンをクリックすると、記事の全文が表示されます。

   ![exercise4-emulator-detailsofarticle](./media/4-16.png)

5.  特定のカテゴリを検索してみてもかまいません。`explore software articles`と入力すると、そのカテゴリに属する記事がいくつか表示されます。

   ![exercise4-emulator-explorecategory2](./media/4-17.png)

6.  同様に、特定のトピックに関する記事を検索してみてもかまいません。たとえば、`search about OneDrive` と入力します。

   ![exercise4-emulator-search](./media/4-18.png)

 > **注:** 検索によって返されるドキュメントごとにスコアが返されることを確認してください。

## その他の課題

自主的に学習を続ける場合は、次のタスクを利用できます。

* カルーセルで使用される記事 `ThumbnailCard` をアダプティブカードで変更できます。例として[こちら](../assets/exercise4-KnowledgeBase/FurtherChallenge/articlesCard.js)で提供されているコードを使用できます。
* 記事 `ThumbnailCard` で既定の画像を表示する代わりに、[Bing Image Search API](https://azure.microsoft.com/en-us/services/cognitive-services/bing-image-search-api/)
を使用して、記事のカテゴリに関連する画像を表示できます。ハンズオン ラボの[assets](../assets)
フォルダーの[こちらのモジュール](../assets/exercise4-KnowledgeBase/FurtherChallenge/imageSearchApiClient.js)を使用できます。

**次の演習** [演習5 クラウドへのボットの展開](./exercise5-Deployment.md)
