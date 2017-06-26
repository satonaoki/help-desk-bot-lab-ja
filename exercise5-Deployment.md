# 演習 5: クラウドへのボットの展開

この演習では、自分のボットを登録し、Azure
に展開して、他の人々がそのボットを使用できるようにする方法を学習します。

## 目標

この演習を完了するためには、以下の操作を実行できなければなりません。

-   [Bot Framework Portal](https://dev.botframework.com/) へのボットの登録

-   Azure Web アプリの作成、およびアプリでのボット コードの発行

## 前提条件

-   前の演習を完了していること、あるいは
    [C\#](./CSharp/exercise4-LuisDialog)
    または
    [Node.js](./Node/exercise4-LuisDialog)
    用の開始点を使用できることが必要です。

-   [Azure](https://azureinfo.microsoft.com/us-freetrial.html?cr_cc=200744395&wt.mc_id=usdx_evan_events_reg_dev_0_iottour_0_0)
    サブスクリプション。

-   Node.js で [Git コマンド ライン
    インターフェイス](https://git-scm.com/downloads)を保持している必要があります。

-   [Skype](https://www.skype.com/) アカウント (オプション)。

## Bot Framework へのボットの登録

自分のボットを他の人々が使用できるようにするためには、そのボットを Bot Framework
に登録する必要があります。登録は簡単に行えます。そのボットに関する情報を入力するように求められます。入力すると、そのボットが
Bot Framework での認証に使用するアプリ ID
とパスワードがポータルによって生成されます。

![](media/fcb92a16375bee8d386fde934483f6e0.png)

## Azure へのボットの発行

自分のボットを他の人々が使用できるようにするためには、そのボットをクラウドに展開する必要があります。Azure
または他の任意のクラウド サービスに展開できます。

### Node.js

次のような**アプリ設定**キーを Web アプリに追加する必要があります。

| **キー**                 | **説明**                                                                         |
|--------------------------|----------------------------------------------------------------------------------|
| AZURE\_SEARCH\_ACCOUNT   | **Azure Search アカウント名**を使用します。                                      |
| AZURE\_SEARCH\_INDEX     | **Azure Search インデックス名**を使用します。                                    |
| AZURE\_SEARCH\_KEY       | **Azure Search キー**を使用します。                                              |
| MICROSOFT\_APP\_ID       | **ボット アプリ ID** を使用します。                                              |
| MICROSOFT\_APP\_PASSWORD | **ボット パスワード**を使用します。                                              |
| LUIS\_MODEL\_URL         | **LUIS モデル URL** を使用します。                                               |
| TICKET\_SUBMISSION\_URL  | Web アプリの URL (例: <https://help-desk-bot.azurewebsites.net/>) を使用します。 |

ローカルの git リポジトリからの展開をサポートするように Web
アプリを構成し、展開の資格情報をセットアップする必要があります。次に、そのローカル
git
リポジトリをプロジェクトに追加する必要があります。変更内容をリポジトリにプッシュすると、コードが自動的に
Web アプリに発行されます。

### C#

Visual Studio から Web アプリに直接ボットを発行する必要があります。まだ何も作成していない場合は、作成します。または、既存のものを選択します。

![](media/6591781d7291d3a21fa1a8ca7169940b.png)

次のような**アプリ設定**キーを Web アプリに追加する必要があります。

| **キー**             | **説明**                                                                         |
|----------------------|----------------------------------------------------------------------------------|
| AzureSearchAccount   | **Azure Search アカウント名**を使用します。                                      |
| AzureSearchIndex     | **Azure Search インデックス名**を使用します。                                    |
| AzureSearchKey       | **Azure Search キー**を使用します。                                              |
| MicrosoftAppId       | **ボット アプリ ID** を使用します。                                              |
| MicrosoftAppPassword | **ボット パスワード**を使用します。                                              |
| TicketsAPIBaseUrl    | Web アプリの URL (例: <https://help-desk-bot.azurewebsites.net/>) を使用します。 |

## ボット構成の更新

**Bot Framework Portal** でボットのアプリ サービス URL を Web アプリの URL
によって更新する必要があります。URL の末尾には必ず /api/messages
を付け、またプロトコルは必ず **https** にしてください。

![](media/b73673b2b414ffb9d144f3b1dbd800a5.png)

**注:** この時点で、**Bot Framework Portal** の **Web Channel Control**
で、発行したボットをテストできます。また Skype でもテストできます (Skype
をインストールする必要があります)。

## その他の課題

-   Bot Emulator
    でボットをテストします。[ngrok](https://docs.microsoft.com/en-us/bot-framework/debug-bots-emulator#a-idngroka-install-and-configure-ngrok)
    を使用して、ローカル
    コンピューターへの応答方法をボットに知らせる必要があります。

-   Skype または Web
    チャットを介したボットのテスト中、[コードをローカルで実行します](https://blogs.msdn.microsoft.com/jamiedalton/2016/07/29/ms-bot-framework-ngrok/)。

-   [Application Insights を使用した Bot
    Analytics](https://docs.microsoft.com/en-us/bot-framework/portal-analytics-overview)
    の追加も試してみてください。

-   [Slack](https://slack.com/) などの別のチャネルにボットを登録します。

## 参考資料

-   [Register a bot with the Bot Framework (Bot Framework
    へのボットの登録)](https://docs.microsoft.com/en-us/bot-framework/portal-register-bot)

-   [Deploy a bot to Azure from a local git repository (ローカルの git
    リポジトリから Azure
    へのボットの展開)](https://docs.microsoft.com/en-us/bot-framework/deploy-bot-local-git)

-   [Deploy from Visual Studio (Visual Studio
    からの展開)](https://docs.microsoft.com/en-us/bot-framework/deploy-bot-visual-studio)

-   [Connect to Skype Channel (Skype
    チャネルへの接続)](https://dev.skype.com/bots)
