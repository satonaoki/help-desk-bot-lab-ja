# 演習 8: バックチャネルを通したイベントの送受信 (Node.js)

## 概要

バックチャネル メカニズムを使用すると、ユーザーからは見えない情報をクライアント アプリとボットがやり取りできます。たとえば、クライアントのタイム ゾーンの要求や GPS の位置情報の読み取り、Web ページでのユーザーの操作内容などがあります。ボットがユーザーをガイドして、Web フォームの一部に自動的に記入することなどもできます。バックチャネルは、JavaScript クライアント アプリとボットの隔たりを埋める役割を果たします。このメカニズムは、Direct Line API を使用して実装します。この API を使用すると、クライアントとボットの間でアクティビティを送受信できます。

この演習では、アプリに Web ページを追加します。ボットと Web ページはバックチャネル メカニズムを使用して通信を行います。ボットはユーザーのチケットを Web ページに送信し、Web ページではそのチケットに関連するサポート技術情報の記事を表示します。目標は、人間のスーパーバイザー エージェントが会話を監視し、Web ページを使用してユーザーに役立つ記事があるかどうかを判断し、チケットの作成を回避できるようにすることです。ユーザーが記事を見つけたら、スーパーバイザー エージェントはその記事をクリックし、ユーザーとボットの会話で表示します。

Bot Framework のドキュメントでは、[バックチャネルの仕組み](https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-backchannel) の詳細を説明しています。

[こちらのフォルダー](./exercise8-BackChannel) 内には、この演習のステップの完了結果として得られるコードを含むソリューションが入っています。このソリューションは、演習を進めるにあたってさらにヒントが必要な場合に、ガイダンスとして使用できます。これを使用するには、まず `npm install` を実行して、`.env` ファイルに値を入力しておく必要があることを忘れないでください。

この図は、この演習のコンポーネントを簡単に示しています。

![](media/2e3ecb90036d37aeafeec4e9ec5913d7.png)

## 前提条件

この演習を完了するには、以下のソフトウェアが必要です。

-   [最新の Node.js と NPM](https://nodejs.org/ja/download/)

-   [Visual Studio Code](https://code.visualstudio.com/download) (推奨) や Visual Studio 2017 Community 以上などのコード エディター

-   [Bot Framework Emulator](https://emulator.botframework.com/) (en-US ロケールで構成されていることを確認してください)

-   ローカル開発用 [ngrok](https://ngrok.com/)

## タスク 1: ボットの Web チャット チャネルへの新しいサイトの追加

1.  [Bot Framework ポータル](https://dev.botframework.com/) にサインインします。

2.  [My bots] ボタンをクリックし、編集するボットをクリックします。
    
    > **注:** この演習では、Bot Framework ポータルにボットが既に登録されていることを前提としています。登録していない場合は、[演習 5](./exercise5-Deployment.md) の説明を参照してください。

3.  Web チャット チャネルの [編集] ( ![](media/aa9a2da6246391b3e1b7334f24ede1c9.png) ) リンクをクリックします。開いたウィンドウで [Add new site] をクリックします。サイト名 (例: ヘルプ デスク チケット検索) を入力します。

    ![](./media/8-3.png)

4.  [完了] をクリックすると、次のページが表示されます。**秘密鍵**が 2 つあることに注意してください。後で使用できるように、いずれか 1 つを保存しておきます。[完了] をクリックします。

    ![](./media/8-4.png)

## タスク 2: 埋め込み済み Web チャットによる HTML ページの追加

このタスクでは、Web Chat コントロールと、ボットに event メッセージを送受信するコードが含まれるアプリに HTML ページを追加します。次に、event アクティビティをボットから Web ページに送信する BackChannel 機能を追加します。

1.  前の演習から得られたアプリを開きます。または、[exercise7-HandOffToHuman](./exercise7-HandOffToHuman) フォルダーにあるアプリを使用することもできます。

    > **注:** あらかじめ提供しているソリューションを使用する場合は、必ず以下の値を置き換えてください。
    >   - **{LuisModelEndpointUrl}** プレースホルダーを、使用しているモデル URL に置き換えます
    >   - **{textAnalyticsKey}** を、自分が使用している Text Analytics キーに置き換えます (演習 6 で説明しています)。
    >   - **{searchIndexName}** と **{searchIndexKey}** を、自分の Search インデックス名とキーに置き換えます (演習 4 で説明しています)。
    >   - **MICROSOFT\_APP\_ID** と **MICROSOFT\_APP\_PASSWORD** を、Bot Framework ポータルからのボットの値に置き換えます

2.  ソリューションのルート フォルダーに web-ui という名前の新規フォルダーを作成します。そのフォルダーに、[assets](../assets) フォルダーの[default.htm](../assets/exercise8-BackChannel/default.htm) ファイルをコピーします。

3.  [botchat.js script element](../assets/exercise8-BackChannel/default.htm#L52) の下に、以下のコードを含む新しいスクリプト要素を追加します。このコードでは、Web Channel Secret で **DirectLine** オブジェクトを作成し、WebChat コントロールをそのページに登録します。{DIRECTLINE\_SECRET} プレースホルダーを、これまでに取得した秘密鍵で置き換え、{BOT\_ID} プレースホルダーをボット処理 ID (例: *help-desk-bot*) で置き換えます。

    ```html
    <script>
        var botConnection = new BotChat.DirectLine({
            secret: '{DIRECTLINE_SECRET}'
        });
        var resPanel = document.getElementById('results');

        BotChat.App({
            botConnection: botConnection,
            user: { id: 'WebChatUser' },
            bot: { id: '{BOT_ID}' },
            locale: 'en-us',
        }, document.getElementById('bot'));
    </script>
    ```

    > **注:** [オープンソースの Web Chat コントロール](https://github.com/Microsoft/BotFramework-WebChat)は、[Direct Line API](https://docs.botframework.com/en-us/restapi/directline3/#navtitle) を使用してボットと通信を行います。Direct Line API を使用することで、`activities` をクライアントとボットの間で送受信できます。最も一般的な型のアクティビティは `message` ですが、その他の型もあります。たとえば、`typing` 型のアクティビティは、ユーザーが入力していること、またはボットが応答のコンパイル作業中であることを示します。

4.  同じスクリプト要素で、`event` アクティビティを受信するためのボット アクティビティ リスナーを追加し、記事リストを表示します。

    > **注:** Web Chat コントロールは、`type="event"` のアクティビティを自動的に無視します。これにより、ページはボットと、ボットはページと直接通信を行うことができます。

    ```javascript
    botConnection.activity$
        .filter(function (activity) {
            return activity.type === 'event' && activity.name === 'searchResults';
        })
        .subscribe(function (activity) {
            updateSearchResults(activity.value)
        });

    function updateSearchResults(results) {
        resPanel.innerHTML = '';
        results.forEach(function (result) {
            resPanel.appendChild(createSearchResult(result));
        });
    }

    function createSearchResult(result) {
        var el = document.createElement('div');
        el.innerHTML = '<h3>' + result.title + '</h3>' +
            '<p>' + result.text.substring(0, 140) + '...</p>';

        return el;
    }
    ```

    > **注:** わかりやすくするため、ユーザーとの会話を含む Web Chat コントロールと検索結果は同じページに表示します。ただし、この 2 つはそれぞれ別々に扱うことが理想的です。エージェントが監視と推奨記事の送信ができるように、スーパーバイザー Web サイトには進行中の会話のリストを表示する必要があります。

## タスク 3: ボットを更新して `event` アクティビティを Web アプリに送信

このタスクでは `event` メッセージをボットと送受信する機能を追加します。

1.  **app.js** を開きます。次の `require` ステートメントをファイルの冒頭に追加します。

    ```javascript
    const path = require('path');
    ```

2.  `var bot = new builder.UniversalBot(...);` の行の上に、次のコードを追加します。このコードでは、*Restify* で `web-ui/default.htm` ファイルを既定の Web ページとして使用するように指定します。

    ```javascript
    server.get(/\/?.*/, restify.serveStatic({
        directory: path.join(__dirname, 'web-ui'),
        default: 'default.htm'
    }));
    ```

3.  **SubmitTicket** ダイアログのウォーターフォールの最初のステップで、`session.dialogData.description` でメッセージを保存した場所のすぐ下に、次のコードを追加します。このコードでは、**サポート技術情報**を検索して `event` アクティビティと検索結果を Web ページに送信します。

    ```javascript
    azureSearchQuery(`search=${encodeURIComponent(session.message.text)}`, (err, result) => {
        if (err || !result.value) return;
        var event = createEvent('searchResults', result.value, session.message.address);
        session.send(event);
    });
 
4.  `createEvent` 関数を追加します。この関数は、`event` 型のバックチャネル メッセージを作成し、値としてクエリを実行します。
    
    ```javascript
    const createEvent = (eventName, value, address) => {
        var msg = new builder.Message().address(address);
        msg.data.type = 'event';
        msg.data.name = eventName;
        msg.data.value = value;
        return msg;
    };
    ```

## タスク 4: ボットから Web アプリへのバックチャネル メッセージのテスト

1.  コンソールからアプリを実行します (`nodemon app.js`)。

2.  *ngrok* をダウンロードした新しいコンソール ウィンドウが開き、そのウィンドウで `ngrok http 3978` と入力します。`3978` は、ボットを実行しているポート番号であることに注意してください。別のポート番号を使用している場合は、変更します。次に、転送先の **https** URL も保存しておきます。

    ![](./media/8-5.png)

    > **注:** ngrok を使用することで、ローカル コンピューター上で実行している Web サーバーをインターネットに公開できます。ngrok で、Web サーバーがリッスンしているポートを指定します。開始すると、ターミナルの UI と、トンネルのパブリック URL およびトンネルを通じて行われる接続についてのその他のステータスとメトリック情報が表示されます。

3.  [Bot Framework ポータル](https://dev.botframework.com/) にサインインします。

4.  [My bots] ボタンをクリックし、次に、編集するボットをクリックします。[設定] タブをクリックして、メッセージのエンドポイント URL を更新します
    (`/api/messages` を忘れずに保持してください)。[変更の保存] ボタンをクリックします。

5.  Web ブラウザでボットの URL に移動します (いつもどおり <http://localhost:3978/> です)。Web Chat コントロールで、`「I need to reset my
    password, this is urgent」` (急いでパスワードを変更する必要があります) と入力します。Web Chat コントロールで、チケット確認メッセージが通常どおりに表示されます。入力した説明に応じて、右側に記事リストが表示されることを確認してください。

    ![](./media/8-6.png)

## タスク 5: Web ページを更新して `event` メッセージをボットに送信

1.  **default.htm** ファイルを開きます。ファイル冒頭の `<style>` セクションで、`#results h3` セレクタ―を次の CSS で置き換えます。

    ``` css
    #results h3 {
        margin-top: 0;
        margin-bottom: 0;
        cursor: pointer;
    }
    ```

2.  `createSearchResult` 関数を次のコードで更新します。このコードは、ユーザーが記事のタイトルをクリックすると、`event` アクティビティをボットにポストします。

    ```javascript
    function createSearchResult(result) {
        var el = document.createElement('div');
        el.innerHTML = '<h3>' + result.title + '</h3>' +
            '<p>' + result.text.substring(0, 140) + '...</p>';

        el.getElementsByTagName('h3')[0]
            .addEventListener('click', function () {
                botConnection
                    .postActivity({
                        type: 'event',
                        value: this.textContent.trim(),
                        from: { id: 'user' },
                        name: 'showDetailsOf'
                    })
                    .subscribe(function (id) {
                        console.log('event sent', id);
                    });
            });

        return el;
    }
    ```

## タスク 6: ボットを更新して `event` アクティビティを受信

1.  **app.js** ファイルを開き、以下のイベント リスナー登録を追加します。これは、ユーザーが記事のタイトルをクリックすると呼び出されます。この関数は、要求された文字列でサポート技術情報の記事のタイトルを検索し、結果を **Web Chat コントロール**でユーザーに送信します。

    ``` javascript
    bot.on(`event`, function (event) {
        var msg = new builder.Message().address(event.address);
        msg.data.textLocale = 'en-us';
        if (event.name === 'showDetailsOf') {
            azureSearchQuery('$filter=' + encodeURIComponent(`title eq '${event.value}'`), (error, result) => {
                if (error || !result.value[0]) {
                    msg.data.text = 'Sorry, I could not find that article.';
                } else {
                    msg.data.text = result.value[0].text;
                }
                bot.send(msg);
            });
        }
    });
    ```

    > **注:** [`on` イベント リスナー](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#on) の詳細については、Bot Framework のドキュメントを参照してください。

## タスク 7: アプリからボットへのバックチャネル メッセージのテスト

1.  ボットが引き続き実行されていることを確認します (`nodemon app.js`)。ngrok についても同様に確認します (n`grok http 3978`)。

2.  Web ブラウザでボットの URL に移動します (いつもどおり <http://localhost:3978/> です)。Web Chat コントロールで、`「My computer is not working.」` (コンピュータは動いていません) と入力します。

3.  右側のいずれかの記事のタイトルをクリックすると、Web Chat コントロールに記事のコンテンツが表示されます。

    ![](./media/8-7.png)

## その他の課題

* エージェントが、ボットと話している現在のすべてのユーザーを確認できるようにするために、ボットを、バックチャネルを備えたヒューマン ハンドオフに統合し、それから、エージェントが特定のユーザーと接続できるようにします。
