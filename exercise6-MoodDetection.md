# 演習 6: ユーザーのメッセージの背後にあるセンチメントの判別

ユーザーとボットとの対話はほとんどの場合自由形式で行われるため、ボットは言語を自然に、かつ文脈に応じて理解する必要があります。この演習では、Azure
Text Analytics API を使用してユーザーの感情や気分を検出する方法を学習します。

Azure Cognitive Services オファリングの一部である [Text Analytics
API](https://azure.microsoft.com/ja-jp/services/cognitive-services/text-analytics/)
を使用することによって、センチメント、キー
フレーズ、トピック、および言語をテキストから検出できます。この API は 0 ～ 1
の数字によるスコアを返します。1 に近いスコアは肯定的なセンチメントを、0
に近いスコアは否定的なセンチメントを示します。センチメント
スコアは、分類の手法により生成されます。

こちらの
[C\#](https://github.com/GeekTrainer/help-desk-bot-lab/blob/develop/CSharp/exercise6-MoodDetection)
または
[Node.js](https://github.com/GeekTrainer/help-desk-bot-lab/blob/develop/Node/exercise6-MoodDetection)
のフォルダー内には、この演習のステップで作成するコードを含むソリューションが入っています。このソリューションは、演習を進めるにあたってさらにヒントが必要な場合に、ガイダンスとして使用できます。

## 目標

この演習を完了するためには、ボットが以下の操作を実行できなければなりません。

-   チケットの送信後に、ユーザーにフィードバックを要求する

-   Text Analytics Service
    でフィードバックを分析し、返されたスコアに応じて肯定的または否定的なセンチメントを検出して、それぞれ異なるメッセージを送信する

ボットとの対話のサンプルを次に示します。

![](media/04b2d8c04ebb44d29f60ac886f29209e.png)

![](media/ab14ca08b5087ffb6c7ed534766d29f8.png)

## 前提条件

-   前の演習を完了していること、あるいは
    [C\#](https://github.com/GeekTrainer/help-desk-bot-lab/blob/develop/CSharp/exercise4-KnowledgeBase)
    または
    [Node.js](https://github.com/GeekTrainer/help-desk-bot-lab/blob/develop/Node/exercise4-KnowledgeBase)
    用の開始点を使用できることが必要です。

-   [Azure](https://azureinfo.microsoft.com/us-freetrial.html?cr_cc=200744395&wt.mc_id=usdx_evan_events_reg_dev_0_iottour_0_0)
    サブスクリプション。

## Text Analytics Service の作成

[Cognitive Services
ポータル](https://azure.microsoft.com/ja-jp/try/cognitive-services/)の [言語]
タブから、Text Analytics キーを作成する必要があります。次に、**Text Analytics
Service** を使用する Text Analytics REST API
クライアントを作成します。この記事の説明に従って、URL
<https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/sentiment>
に対する POST 呼び出しを実行します。

## フィードバックを要求してユーザーのセンチメントを分析できるようにするためのボットの変更

ユーザー
エクスペリエンスに関するフィードバックを要求する新しいダイアログを作成して、チケット送信作成の完了後にそのダイアログを呼び出すことができます。そして、作成した
API クライアントを使用して **Text Analytics Service**
を呼び出し、分析結果に応じて異なるメッセージを使用してユーザーに応答します。フィードバックが肯定的な場合
(スコアが 0.5 以上)、ボットは単にユーザーにお礼を言います。そうでない場合
(スコアが 0.5 未満) は、IT
担当者がすぐに連絡する旨のメッセージをユーザーに返します。この後の演習 (7)
では、否定的なフィードバックを受け取ったときに、会話を人間の担当者に引き渡し
(ハンドオフ)、担当者を通じてユーザーを支援する方法について学習します。

## その他の課題

-   Microsoft Cognitive Services
    内の別のサービスを使用して、ボットに音声認識機能を追加できます。[Bing Speech
    API](https://azure.microsoft.com/ja-jp/services/cognitive-services/speech/)
    を試してみてください。

## 参考資料

-   [Text Analytics API Overview (Text Analytics API
    の概要)](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/overview)

-   [Text Analytics API QuickStart (Text Analytics API
    クイックスタート)](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/quick-start#task-2---detect-sentiment-key-phrases-and-languages)
