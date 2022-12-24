
<!-- 外部参照リンクの定義 -->

<!-- WebClapModoki 標準動作用の Google スプレッドシート（テンプレート） -->
[SS_template]: https://docs.google.com/spreadsheets/d/1j_h4weHihW-uVeTFtHTwrs6QwRPAT36sEBf3DPbo3ms/edit?usp=sharing
<!-- WebClapModoki の GAS 本体 -->
[WebClapModoki_GAS]: /WebClapModoki-GAS.txt
<!-- WebClapModoki を里々から叩く場合のサンプルコード -->
[dic_wcm]: /dic_wcm.txt
<!-- Figure 1 -->
[fig_1]: /Figure1_v2.gif
<!-- Figure 2 -->
[fig_2]: /Figure2_v1.png
<!-- セットアップ手順詳細説明用スライド -->
[ExpSlide]: /ExpSlide_20221225.pdf
<!-- WebClapModoki を里々から叩く場合のサンプルコード, 生テキスト形式表示 -->
[dic_wcm_raw]: https://raw.githubusercontent.com/dullNeko/WebClapModoki_GAS/main/WebClapModoki_GAS.txt
<!-- 伺的 Advent Calendar 2022 -->
[uka_adcurry2022]: https://adventar.org/calendars/8310
<!-- Web拍手公式サイト -->
[WebClapOrg]: https://www.webclap.com/index.html
<!-- GAS の各種制限 -->
[GAS_quotas]: https://developers.google.com/apps-script/guides/services/quotas 
<!--伺的 Advent Calendar 2022 延長戦ハッシュタグ -->
[uka_adcurry2022_extended]: https://twitter.com/hashtag/uka_advent_2022

# WebClapModoki-GAS

WebClap-like system by using Google Apps Script & Google spreadsheet.

# 前書き

[伺的アドベントカレンダー2022][uka_adcurry2022]、25日の記事です。  
[24日の Zichqec 氏の記事](https://zichqec.github.io/s-the-skeleton/advent_calendar_2022_02)に続いて、dullNeko がお送りします。

伺か界隈で良くお見掛けする [Web拍手][WebClapOrg] がアレなことになっていると知り、  
手持ちの技術で代替になりそうな何かを組んでみた、という記事になっております。

長い事ユーザ専門だったので、  
こういった記事を書くのも、人様に使って頂く目的でスクリプトを公開するのも、初めてです…  
何卒、お手柔らかにお願い申し上げます。


# これは何？

- __Web拍手と同じような使い勝手__ で
  - （＝外部サイトに誘導せず、SSP内での \![open,inputbox,...] と \![execute,http-POST,...] の組み合わせだけで使えて）
- ユーザさんから直接「感想」「バグ報告」などコメントを投げてもらえて
- __デべさんのGoogleアカウント内に作成したGoogleスプレッドシート__ に、投稿されたコメントを記録する

…ことができる __Web拍手"もどき"__ です。


# どんなことができるの？

動作イメージは…↓のGIF画像（Fig.1）をご覧頂ければわかりやすいかと思います。  
SSPのインプットボックスから入力した「いいね！」と、付随する情報が Google スプレッドシートに記録されている様子のGIF動画です。  
__画面遷移を伴わず、SSPからWeb拍手と同じ様に扱える__ のが最大の特徴です。

※お借りしているシェルは [にはちびっとさんのフリーシェル「和装のおばあさん01」](https://waka.edo-jidai.com/sub/freeshell.html) です。  
※マウスカーソルは [ゴースト回覧板３ｒｄ「ロスト・ユー・サムウェア」スレの2214氏作ルストリカマウスカーソル](https://jbbs.shitaraba.net/bbs/read.cgi/computer/44300/1432111308/2214) です。

![20221216_WebClapModoki_SampleAnimation_v1][fig_1]
**Fig.1 動作イメージ（こんな感じの動作をします、というデモGIF動画）**


# どういう仕組みなの？

本システムの動作図解…というか概念図のようなものとして、↓の画像（Fig.2）を作ってみました。

お伝えしたい最重要ポイントは、  
「本スクリプトで作れるWeb拍手もどきシステムは、  
　（ここでスクリプトを公開してる dullNeko の Google ドライブではなく）  
 　__デべさんご自身の Google ドライブ上（に設置されたスクリプト）で動くもの__」  
という点です。

なので、ご自分の Google ドライブ上へのスクリプトの導入・初期設定など、  
__デべさんご自身に、手動で実行して頂かなければならない作業__ があります。  

![20221222_illustrated_behavior_v1][fig_2]
**Fig.2 動作の図解というか概念図っぽいもの**


# なんで作ったの？

伺かのゴーストをtkっていて、久々に [Web拍手][WebClapOrg] を利用させて頂こうと思って調べ直したところ、  
かなり前から

- [公式サイト][WebClapOrg] に繋がらない？
- 既知の不具合（絵文字などが入力されると、以降の文字列が読めなくなる等）が複数あるが、更新・対応される気配はない（っぽい）
- SSLへの対応が行われる可能性も…無さそう…

な状態にある、と知って採用を見送った…のですが、  
「SSP からさくらスクリプトの \![execute,http-POST,...] 使って、コミュニケートボックスから __直接__ 一言感想出せるの、ユーザ的には気楽だし手軽でいいのよね…何か他の手はないかな…」などと諦めきれずに考えた結果、  
「そういやラズパイからセンサのデータを POST で投げて、__Google Apps Script(GAS)__ で受けてスプレッドシートに記録するとかやってたな？」と思い出し、  
突貫で試作してみたものがこちらになります。  


# さっきから出てくる GAS…Google Apps Script？って何？

公式サイトは  
https://workspace.google.co.jp/intl/ja/products/apps-script/  
…なんですが、このページを見て何なのか／何をするものかパッと分かる人は…多分、控えめに言っても少数派ですよね…

Googleさんの公式な説明として（まだ）分かりやすいのは  
__「Google プロダクト全体でタスクと統合して自動化できる"クラウドベースの JavaScript プラットフォーム"」__  
でしょうか。

身も蓋もない言い方をしてしまえば、  
__「サーバサイドのプログラムを、サーバの立ち上げ・維持管理不要で組める」__  
代物です。  

Google さんが提供するサービスなので、  
__他の各種 Google サービス（Google スプレッドシート、Gmail、カレンダー、etc...）と連携して処理する機能__ もついてきます。  
（というか、公式の謳い文句的にはこちらがメインですね）


## GAS の良い所は？

前節の繰り返しになってしまいますが、  
__「サーバ側でないと難しい処理」__  
…本プロジェクトで言えば、

- 不特定多数のユーザさん（のSSP）から送られるHTTPリクエスト（※POSTメソッド限定）を受信し
- 受け取ったデータを適当な形の文字列・配列に整形し
- スプレッドシートに記録するなど、様々な後処理を行う

といった動作を、

- 自分でサーバを立ち上げたり維持管理することなく
- JavaScript(ライクなGoogle Apps Scriptという)言語で処理を記述できて
- Googleアカウント（Goodleドライブにアクセスできる環境）さえあれば無償で利用できる

…というのが、GASの良い所です。


## 逆に、GAS の悪い所は？

悪い所…というか、  
（dullNekoが調べた範囲で、 __誰からもアクセスできるWebアプリ__ とした場合に）  
__GASでできないこと／問題点になりそうな所__ を挙げておきますね。

- __アクセス元情報など「サーバ側でないと得られない上に重要な情報」が取得できない__
  - 標準仕様の「ご自身の Google ドライブ上にある Google スプレッドシートに、受け取ったコメントを書き込んでいくだけ」なら、特に問題にならないと思います。
  - …ですが、 __外部のAPIを叩く処理（例：「Mastodon へのトゥート機能」など）を実装し、運用した場合には厳しい点__ です。  
  - 対象のサーバさんから見れば、 __アクセス元はあなた（のGoogleアカウント内のスクリプト）__ であって、大本のHTTPリクエストを送信した方ではありません。
  - 原因が何であろうと（例え、悪意のないスクリプトの誤動作だったにせよ）、結果として相手様から DoS 攻撃と認識される事態が生じた場合、 __責任を追及される可能性が極めて高い__ のは __スクリプトを設置し、動作させたあなた__ です。
  - 2010年に起こった「岡崎市立中央図書館事件（Librahack事件）」と似たケースになる可能性を、私は否定できません…。
  - そのため、__標準仕様では「Mastodonへのトゥート機能」は使えないように細工__ してあります。
  - ご自身に責任が降りかかる危険性を認識した上で、十分な注意を払っての使用をお願いいたします。
- __「一日あたりの処理量」に制限があり、それらのほとんどは有償プランでも緩和されない__
  - 処理量制限の詳細は [こちらのページ][GAS_quotas] にある通りです。
  - [WebClapModoki_GAS.txt][WebClapModoki_GAS] の冒頭、「★ 参考資料」セクション内「★ GAS全体の制限について」でも記述しましたが、結構厳しめの制限です。
  - この制限のため、スクリプトを組む際には「セル参照を行う回数をできるだけ減らし、書き込み・読み出しは一括で行う」などのテクニックを使う必要性が高くなっています。
- __Google さんのサービスなので、唐突にサービス終了して使えなくなる可能性がある__
  - 2009年から続いている古株サービスですし、利用者も少なくない（はず）ですし、何より Google のサービス全体を連携させて使うための仕組みなので、可能性は低め…だとは思いますが…無い、とは言い切れません。
- __Google Apps Script の詳細な全仕様は公開されていない__
  - 大元としては JavaScript1.6 がベースである（らしい）のですが、ECMAScriptの"一部"取り入れ（例：[V8ランタイムは使用可能だが、ES6 モジュールはサポートされてない](https://developers.google.com/apps-script/guides/v8-runtime?hl=ja)等）や、独自実装（例：各 Google ドライブ上にあるデータ内でのみ扱えるプロパティサービス等、Google サービスに紐づいたAPI）等もあり、やや複雑な言語になっています。
  - また、予告なく機能や仕様が変更される（例：上のプロパティサービスに関わる関数群）こともあり、今の所使えている実装が気付いたら動かなくなっていた…というパターンが無きにしも非ず。


# 他のサービスと比べてどう？わざわざ自作する意味あるの？

真剣に比較検討したわけではない…ので恐縮ですが、

- __GAS は Google アカウントさえあれば、すぐに使える__
  - ＝Web拍手等、CGIプログラム設置の為だけにサーバをお借りする…ような手間が無く、手軽に使い始めることができます。
  - Twitter（各種SNS）との連携等も必要なく、単体（GAS＋Googleスプレッドシート）で動作するので、SNSやりたくない派／サービス紐づけとかしたくない派の方にもおススメ。
- __基本的に無償で使用可能__
  - ※同じ日に大量にアクセスがあったりすると制限がかかり、通信を遮断される可能性はあります。
  - これはまあ…他サービスもそんな変わらないかな…。
- __頂いた感想について、スプレッドシート上で一覧・集計・検索したりできる__
  - 特定の条件（例えば「バグ報告だけ」）拾うとか、対応状況（した／してない／できない）の列を右端に足して管理したり、といった工夫ができるのは便利…だと思うのですけれど…
- __感想の回収・保全が簡単__
  - 頂いた感想・バグ報告等、コメントの記録先が __自分の Google ドライブ上のスプレッドシート__ なので、「サービスの終了で突然データが消えちゃう」とか「まとめてダウンロードできない」とか、そういう類の心配はなさげです。
  - 定期的にダウンロードしてローカルに保存する、コピーを作って別の場所に保存する、程度の単純なバックアップ操作がし易いのは良い点だと思います。
  - …逆に __うっかり記録先シートを削除しちゃうような、致命的な操作ミスもできてしまう__ あたりは痛し痒し。
- __自分用にカスタマイズする難易度が低め__
  - JavaScriptが全然書けない（アロー関数すらまともに使えない）私でも、本スクリプトを作れる位には低めです。
  - オリジナルのWeb拍手から少しだけ拡張して、__【送信元】;【送信内容（コメント）】;【その他メモ】 の３情報を ;(半角セミコロン) 区切りで取得する__ ようにしたり、位ならできました。
  - 他には…「予め記述したNGワードを弾く関数」を実装してあります。とても原始的な処理ですし、他サービスの高度で自動的なフィルタリング機能に比べると…悲しくなるレベルですが…。
  - GASから更にPOSTメソッドでHTTPリクエストを出す例として、「Mastodon API を利用してトゥートする関数」も実装してあったり。 __（※↑の「悪い所」で前述した問題点／危険性があるので、そのままでは使えないようセーフティをかけてあります）__

…と、「お手軽＆いろいろ自分好みにカスタムできる点」については、  
他のサービスとかと比べてもいいところでは…？と感じたので、  
GitHubの練習がてら公開してみることにした次第です。


# 環境作成・設定方法、使い方の説明は？

## __※作業する前に、下記【注意喚起】を必ずお読みください！__

README.md内にて画像付きで説明…してみようとして挫折したので、  
次の PDFファイル (詳細説明スライド) をご覧くださいまし。

[WebClapModoki-GAS 処理内容・導入手順の詳細説明スライド][ExpSlide]

一応、文章のみの簡易な説明も、  
手順X. として下に記述しておきます。


## 【注意喚起】

本スクリプトを導入後、最初に「実行」を試みた際、  
__「承認が必要です：このプロジェクトがあなたのデータへのアクセス権限を必要としています」__  
というウィンドウが出てきます。  
__ここで要求されるアクセス権限を許可しなければ、本スクリプトは実行できません。__

要求される権限は、以下の２つです。  
__「Google スプレッドシートのすべてのスプレッドシートの参照、編集、作成、削除」__  
__「外部サービスへの接続」__  
…２つだけですが、要求される内容はかなり重いものです。

長くなってしまいますが…大事な所なので、できるだけ詳しく説明しますね。

- __「Google スプレッドシートのすべてのスプレッドシートの参照、編集、作成、削除」__  
  - SPREADSHEET   = SpreadsheetApp.openById(SPREADSHEET_ID); を使用しているため。
  - __本スクリプトでは上記の１シートの参照・編集（書込）しか実行しません__ が、__コードを書き換えれば別のシートを操作できますし、シートの作成・削除も可能__ です。
  - __悪意あるスクリプトの場合（あるいはご自分で組んだスクリプトであっても、処理の記述ミス等があれば）、「ドライブに存在するスプレッドシート全てを列挙し、削除する」「スプレッドシート内の機密情報を、外部に吐き出す」などといった悪辣な動作をさせることも可能です。__
  - ？マークをクリックした際の詳細は、次の通りです。
  - ご覧の通り、__「自分が操作する場合にできること全て」ができます__ し、
  - 同時に __「スクリプトが行ったことであっても、ユーザ本人が操作したものと見做される」__ という、強い文言も入っています。
    - このアプリが次の権限を求めています。
    - スプレッドシートの参照、編集（設定、メタデータを含む）
    - スプレッドシートの新規作成
    - __スプレッドシートのアップロード、ダウンロード__
    - __スプレッドシートの整理、削除__
    - __スプレッドシートを共有しているユーザーの名前、メールアドレスの参照__
    - __他のユーザーとのスプレッドシートの共有、共有停止__
    - __このアプリは、ユーザーが行える操作と同じ操作を行うことができます。このアプリが加えた変更は、ユーザーによるものと見なされます。__
    - __スプレッドシートには、財務記録や個人的リストなどの機密情報が含まれている可能性があります。__
- __「外部サービスへの接続」__  
  - tootByMastodonAPI(toot_content) 関数内で UrlFetchApp.fetch を使用しているため。
  - __本スクリプトではコメントアウト＆セーフティがかけられているため、細工を回避しない限り機能しません（この権限を使用することはありません）__ が、それでも記述がある以上、実行される可能性があるとして、要求されます。
  - ？マークの説明に「外部サービスへのネットワーク接続の作成（例: データの読み取りまたは書き込み）」とある通り、UrlFetchApp.fetch はHTTPリクエストを通じてデータのスクレイピング（収集・抽出）を行うものであるため、場合によっては他サーバへの攻撃・業務妨害と見做される動作をしてしまう危険性があります。

ここは…私からは、  
__「スクリプトのソースを読み、実際の処理を見て、判断してください」__  
としか申し上げられません。  
__「dullNeko の書いたスクリプトを信用できない」と思われた場合は、ここまで__ です。  

もし、この権限を要求される段階まで操作を進めてしまっていた場合は、  
アクセス権の付与をキャンセルし、  
作成したスプレッドシートやスクリプトを削除して下されば、元通りの状態に戻せます。  
お手数をおかけして申し訳ありませんが…この点は、何卒ご容赦ください。


## 手順1.  
__ご自身の Google ドライブ__ に、[このスプレッドシート][SS_template]のコピーを作成してください。  
Googleアカウントでログインしたブラウザ上で開けば、メニューの[ファイル]→[コピーを作成]で簡単にコピーできます。  
また、名前（WebClapModoki (template) のコピー）は好きに変えて大丈夫です。


## 手順2.  
同じくご自分の Google ドライブに「新規 Google Apps Script」を作成し、  
[WebClapModoki.txt][WebClapModoki_GAS] の中身をコピペして、上書き保存してください。  
[RAW表示][dic_wcm_raw] からコピペするのが楽かもです。

【注意】  
1.のスプレッドシート、2. の Google Apps Script の共有設定  
（「右クリックメニュー」→「共有」→「一般的なアクセス」の設定値）は、  
初期状態の __「制限付き」でOK__ です。  
これらのURLを公開する必要もありません。むしろ秘密にしておいて下さいまし。


## 手順3.
1.でコピーしたスプレッドシートの 'id'、すなわち
```
（ブラウザで開いた時のURL）
https://docs.google.com/spreadsheets/d/XXX...XXX/edit#gid=0
```
の __XXX...XXX__ をコピーし、
2.の Google Apps Script 内の 
```
const SPREADSHEET_ID = 'XXX...XXX';
```
の __XXX...XXX__ を  
ご自分のスプレッドシートの 'id' に書き換えて、上書き保存してください。


## 手順4.
3.で修正した WebClapModoki の Google Apps Script を開き、  
doPostTest 関数（動作テスト用関数）を実行してみてください。   

__【注意】__  

この時、警告画面が出て、  
__「この Google Apps Script に Google アカウントへのアクセス権を与えてもいいのか？」__  
と尋ねられます。

本スクリプトを利用する場合は、「許可」を選択して下さい。  
アクセス権の付与を許可した場合は、画面が元に戻り、
doPostTest 関数の実行が継続されます。  
特にエラーが出ず、実行ログに「お知らせ　実行完了」と表示されればOKです。 

もしもエラーが出た場合、  
「3.で置換した 'id' の指定をミスっていないか」  
「きちんと全体を過不足なくコピペできているか」あたりを見直してみるといいかもです。

なお、doPostTest 関数の実行後、
1.でコピーしたスプレッドシートの "posted_comment" シートの「コメント」列に、  
「いいね！ 【NG】」が入力されているのは、正常な動作です。  
（checkContentTextByNGWordsList() 関数が仕事して、  
　"URLらしき文字列" を【NG】に置換した結果です）

使っていて「ちょっとこれは見たくないな…」という文字列が出てきたら、  
"NG_words_list" 上に行を追加して、単語を増やしたり書き換えたりしてみてください。 


## 手順5.  
4.で問題がなければ WebClapModoki_GAS を __Webアプリとしてデプロイ__ し、  
```
https://script.google.com/macros/s/YYY~YYY/exec  
```
形式のURLを取得してください。


## 手順6.
5.で取得したURLを [dic_wcm.txt][dic_wcm] の中の  
```
＄WCM_URL	https://script.google.com/macros/s/YYY~YYY/exec  
```
に上書きして、ご自身のゴーストさんに実装してください。  

サンプルコードは里々での実装例ですが、  
実体はさくらスクリプト  
```
\![execute,http-POST,（WCM_URL）,--param=message_body="（送信元）;（送信内容）;（メモ）",（オプション）]  
```
の実行なので、  
特にSHIORIを選ばず動かせると思います。


## 手順7.  
SSP 上で実行してみて、サンプルGIF画像のように、  
ご自身のスプレッドシートの "posted_comment" シートに POST した内容が追加されれば成功です。  
お疲れ様でした！


## 使用を止めたい時は？

使い始めてから「微妙だな…もう使うの止めたい」と思った時は、  
「Google アカウントのアクセス権の確認・削除」画面  
https://myaccount.google.com/permissions  
にアクセスして、  
「アカウントにアクセスできるアプリ」の一覧から、  
当該スクリプトのアクセス権を削除してください。  
4.で付与されたアクセス権が無くなれば、スクリプトは何もできません。

あと、ゴーストさん側の記述（dic_wcm.txt）を削除した上で、  
Google ドライブ上にある 2. の Google Apps Script のファイルの削除も行えば、  
きれいさっぱりです。

最後に、念のため 5.で取得したURLの  
```
https://script.google.com/macros/s/YYY~YYY/exec
```
に、ブラウザからアクセスしてみて下さい。  

「リクエストされたファイルは存在しません。」(404 Not Found)  
が返ってくれば、スクリプトの動作は完全に停止しています。


# 連絡先

Pull Requests や Issues にコメントを投げて頂くのが、本来の運用方法…だと思うのですが、  
「dullNeko が GitHub に不慣れで、使い方を理解できてない」状態なのと、  
「GitHub アカウントを持ってない方もいらっしゃるはず…」と思ったので、  
↓の Google フォームを用意してみました。お気軽にどうぞ。

https://docs.google.com/forms/d/e/1FAIpQLSflhEpY-kmYyDT87xbUyB0sqA88rkK9oxrXjGw0W2EcDHIg6g/viewform?usp=sf_link

~~ぶっちゃけこのスクリプト使うより Google フォームと Google スプレッドシートの連携の方が手軽で早いし十分なのでは？と思ったのはひみつ~~  
い、一応「どこからでも画面遷移なしで使える」のと「受け取った時点での処理ができる」所は勝ってる…はずなので…（震え声）

あ、  
[Twitter](https://twitter.com/dullNeko) と [Mastodon（うかどん）](https://ukadon.shillest.net/@dullNeko) にもおりますので、  
そちらでお声がけ頂いても大丈夫です。


# ライセンスおよび免責事項

本スクリプト [WebClapModoki_GAS.txt][WebClapModoki_GAS] は NYSL Version 0.9982 に従います。  
ライセンス全文は下記の通りです。

--------------------------------------------------------------------
    NYSL Version 0.9982
    ----------------------------------------
    
    A. 本ソフトウェアは Everyone'sWare です。このソフトを手にした一人一人が、
       ご自分の作ったものを扱うのと同じように、自由に利用することが出来ます。
    
      A-1. フリーウェアです。作者からは使用料等を要求しません。
      A-2. 有料無料や媒体の如何を問わず、自由に転載・再配布できます。
      A-3. いかなる種類の 改変・他プログラムでの利用 を行っても構いません。
      A-4. 変更したものや部分的に使用したものは、あなたのものになります。
           公開する場合は、あなたの名前の下で行って下さい。
    
    B. このソフトを利用することによって生じた損害等について、作者は
       責任を負わないものとします。各自の責任においてご利用下さい。
    
    C. 著作者人格権は dullNeko に帰属します。著作権は放棄します。
    
    D. 以上の３項は、ソース・実行バイナリの双方に適用されます。

--------------------------------------------------------------------

    NYSL Version 0.9982 (en) (Unofficial)
    ----------------------------------------
    A. This software is "Everyone'sWare". It means:
      Anybody who has this software can use it as if he/she is
      the author.
    
      A-1. Freeware. No fee is required.
      A-2. You can freely redistribute this software.
      A-3. You can freely modify this software. And the source
         may be used in any software with no limitation.
      A-4. When you release a modified version to public, you
          must publish it with your name.
    
    B. The author is not responsible for any kind of damages or loss
      while using or misusing this software, which is distributed
      "AS IS". No warranty of any kind is expressed or implied.
      You use AT YOUR OWN RISK.
    
    C. Copyrighted to dullNeko.
    
    D. Above three clauses are applied both to source and binary
       form of this software.
--------------------------------------------------------------------


# 謝辞

- だんでぃ こと りょうすけ氏（オリジナルの「Web拍手」作者様）

[だんでぃ こと りょうすけ氏](https://twitter.com/webclap_dandy) が、Web拍手というアイデアを実装し、公開されていなければ、  
本スクリプトを書くどころか思いつくこともなかったと思います。  
個人Webサイトを持ってた時代はお世話になりました…。  
先人に敬意と感謝を。

- 伺か界隈の皆さま

数々の魅力的なゴーストさん達が公開されていなければ、  
ゴーストさんに、Web拍手経由で気軽に感想を送信できる仕組みが実装された例を見ていなければ、  
[2022年度の伺的アドカレー（伺的 Advent Calendar 2022 - Adventar）][uka_adcurry2022] が開催されておらず、他の方々の記事を読んで刺激を受けていなければ、  
また、 [ぽなさんのトゥート](https://ukadon.shillest.net/@ponapalt/109520146570230914) で背中を押して頂いていなければ、  
本記事を公開することはありませんでした。

ユーザ歴ばかり長い新参者ですが、今後とも何卒よろしくお願い申し上げます…。


# 最後に

こんな長文乱筆の記事を最後までお読み下さり、ありがとうございます…。  
改めて、心より感謝申し上げます。

…あくまで dullNeko 個人の思想ですが、  
```
「記事」や「プログラムコード」も「表現媒体」の一つ、
　そのお方が、この世界をどう捉え、どのようなモノの見方をして、
　どんな "カタチ" が良いもの・好ましいもの・大切なもの・理想と考えておられるのか、  
　如実に物語る創作物だ
```
と感じる勢なので、  

今回、伺的 Advent Calendar 2022 で皆様の記事を拝読し、  
各々方の観点・考え方、意見・失敗談・創作物の数々に触れることができて、  
大変幸せな時間を過ごせました。  

…ところで皆様、  
伺的 Advent Calendar 2022 延長戦ハッシュタグこと、  
[#uka_advent_2022][uka_adcurry2022_extended] はご存じでしょうか？  
既に [古閑未善さんが23日に発表された記事](https://kogami.sakura.ne.jp/uka_adv_2022.html) もございます。  

語りたいこと、 これからやってみたいこと、やってみたこと、やらかしたこと…などなど、  
こういう "機会" がなければ、中々 "自分の外" に出せないこと、多いなあ…  
などと、この記事を書き終わった今、ぼんやり感じております。  
（上でも書きましたが、自分が記事を書いて公開する側になるとは思っていませんでしたし）

もし、本記事を読んで下さった貴方様が、  
何か "自分の外" に出してみたいものをお持ちであるのなら…  
[#uka_advent_2022][uka_adcurry2022_extended] という機会をお借りして、  
記事の形で出してみると、きっと良い経験になりますよ…と、  
お誘いの言葉を投げて、本記事の締めとさせて頂きます。
