# WebClapModoki_GAS

WebClap-like system by using Google Apps Script & Google spreadsheet.

# これは何？

- __Web拍手と同じような使い勝手__ で
  - （＝外部サイトに誘導せず、SSP内での \![open,inputbox,...] と \![execute,http-POST,...] の組み合わせだけで使えて）
- ユーザさんから直接「感想」「バグ報告」などコメントを投げてもらえて
- __デべさんのGoogleアカウント内に作成したGoogleスプレッドシート__ に、投稿されたコメントを記録する

…ことができる __Web拍手"もどき"__ です。

# どんなことができるの？

動作イメージは…↓のGIF画像（Fig.1）をご覧頂ければわかりやすいかと思います。  
SSPのインプットボックスから入力した「いいね！」と、付随する情報が Google スプレッドシートに記録されている様子のGIF動画です。  
ご覧の通り、__画面遷移を伴わず、SSPからWeb拍手と同じ様に扱える__ のが最大の特徴です。

※お借りしているシェルは [にはちびっとさんのフリーシェル「和装のおばあさん01」](https://waka.edo-jidai.com/sub/freeshell.html) です。  
※マウスカーソルは [ゴースト回覧板３ｒｄ「ロスト・ユー・サムウェア」スレの2214氏作ルストリカマウスカーソル](https://jbbs.shitaraba.net/bbs/read.cgi/computer/44300/1432111308/2214) です。

![20221216_WebClapModoki_SampleAnimation_v1](https://user-images.githubusercontent.com/120681133/209155366-c6d92596-933c-43fd-bda7-fc407465ceb3.gif)
**Fig.1 動作イメージ**

# どういう仕組み？

本システムの動作図解…というか概念図のようなものとして、↓の画像（Fig.2）を作ってみました。

お伝えしたい最重要ポイントは、  
「本スクリプトで作れるWeb拍手もどきシステムは、  
　（ここでスクリプトを公開してる dullNeko の Google ドライブではなく）  
 　__デべさんご自身の Google ドライブ上（に設置されたスクリプト）で動くもの__」  
という点です。

なので、ご自分の Google ドライブ上へのスクリプトの導入・初期設定など、  
__デべさんご自身に、手動で実行して頂かなければならない作業__ があります…。  

![20221222_illustrated_behavior_v1](https://user-images.githubusercontent.com/120681133/209155615-800e2b75-b31a-446c-ba76-c85c884b5a6d.png)
**Fig.2 動作の図解っぽいもの**

# なんで作ったの？

伺かのゴーストをtkっていて、久々に [Web拍手](https://www.webclap.com/index.html) を利用させて頂こうと思って調べ直したところ、  
かなり前から

- [公式サイト](https://www.webclap.com/index.html) に繋がらない？
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

## GAS の良い所は？

サーバ側でないと難しい処理…本プロジェクトで言えば、

- 不特定多数のユーザさん（のSSP）から送られるHTTPリクエスト（POSTメソッド）を受信し
- それを適当な形の文字列・配列に整形し
- スプレッドシートに記録したり、うかどんにPOSTしたり、色々な後処理を行う

といった動作を、

- 自分でサーバを立ち上げたり維持管理することなく
- JavaScript(ライクなGoogle Apps Scriptという)言語で処理を記述できて
- Googleアカウント（Goodleドライブにアクセスできる環境）さえあれば無償で利用できる

というのが、GASの良い所です。

## 逆に、GAS の悪い所は？

悪い所…というか、  
（dullNekoが調べた範囲で、 __誰からもアクセスできるWebアプリ__ とした場合に）  
GASでできないこと／問題点になりそうな所を挙げておきますね。

- アクセス元情報など「サーバ側でないと得られない上に重要な情報」が取得できない
  - 標準仕様の「ご自身の Google ドライブ上にある Google スプレッドシートに、受け取ったコメントを書き込んでいくだけ」なら、特に問題にならないと思います。
  - …ですが、外部のAPIを叩く処理（例：「Mastodon へのトゥート機能」など）を実装し、運用した場合には厳しい点です。  
  - 対象のサーバさんから見れば、 __アクセス元はあなた（のGoogleアカウント内のスクリプト）__ であって、大本のHTTPリクエストを行った方ではありません。
  - なので、標準仕様では「Mastodonへのトゥート機能」は使えないように細工してあります。ご自身に責任が降りかかることを認識した上で、十分な注意を払っての使用をお願いいたします。
- 「一日あたりの処理量」に制限があり、それらのほとんどは有償プランでも緩和されない
  - 処理量制限の詳細は [こちらのページ](https://developers.google.com/apps-script/guides/services/quotas) にある通りです。
  - [WebClapModoki_GAS.txt](https://github.com/dullNeko/WebClapModoki_GAS/blob/main/WebClapModoki_GAS_v7.txt) の冒頭、「★ 参考資料」セクション内「★ GAS全体の制限について」でも記述しましたが、結構厳しめの制限です。
  - この制限のため、スクリプトを組む際には「セル参照を行う回数をできるだけ減らし、書き込み・読み出しは一括で行う」などのテクニックを使う必要性が高くなっています。
- Google さんのサービスなので、唐突にサービス終了して使えなくなる可能性がある
  - 2009年から続いている古株サービスですし、利用者も少なくない（はず）ですし、何より Google のサービス全体を連携させて使うための仕組みなので、可能性は低め…とは思いますが…。
- Google Apps Script の詳細な全仕様は公開されていない
  - 大元としては JavaScript1.6 がベースである（らしい）のですが、ECMAScriptの"一部"取り入れ（例：[V8ランタイムは使用可能だが、ES6 モジュールはサポートされてない](https://developers.google.com/apps-script/guides/v8-runtime?hl=ja)等）や、独自実装（例：各 Google ドライブ上にあるデータ内でのみ扱えるプロパティサービス等、Google サービスに紐づいたAPI）等もあり、やや複雑な言語になっています。
  - また、予告なく機能や仕様が変更される（例：上のプロパティサービスに関わる関数群）こともあり、今の所使えている実装が気付いたら動かなくなっていた…というパターンが無きにしも非ず。


# 他のサービスと比べてどう？わざわざ自作する意味あるの？

真剣に比較検討したわけではない…ので恐縮ですが、

- GAS は Google アカウントさえあれば、すぐに使える
  - ＝Web拍手というCGIプログラムの為だけにサーバをお借りする…ような手間が無く、手軽に使い始めることができます。
  - Twitter（各種SNS）との連携等が必要なく、単体（GAS＋Googleスプレッドシート）で動作するので、SNSやりたくない派／サービス紐づけとかしたくない派の方にもおススメ。
- 基本的に無償で使用可能
  - ※同じ日に大量にアクセスがあったりすると制限がかかり、通信を遮断される可能性はあります。
  - これはまあ…他サービスもそんな変わらないかな…。
- 頂いた感想について、スプレッドシート上で一覧・集計・検索したりできる
  - 特定の条件、例えば「バグ報告だけ拾う」とかできるの便利そうだなー、という皮算用です…
- 感想のバックアップ取るのが簡単
  - 定期的にダウンロードしてローカルに保存する、コピーを作って別の場所に保存する、程度の単純な操作がし易いのは良いと思います。
  - …逆に __うっかり記録先シートを削除しちゃうような、致命的な操作ミスもできてしまう__ あたりは痛し痒し。
- 自分用にカスタマイズする難易度は低め
  - JavaScriptが全然書けない（アロー関数すらまともに使えない）私でも、本スクリプトを作れる位には低めです。
  - オリジナルのWeb拍手から少しだけ拡張して、【送信元】;【送信内容（コメント）】;【その他メモ】 の３情報を ;(半角セミコロン) 区切りで取得するようにしたり、位ならできました。
  - 他には…「予め記述したNGワードを弾く関数」を実装してあります。とても原始的な処理ですし、他サービスの高度で自動なフィルタリング機能に比べると…悲しくなるレベルですが…。
  - GASから更にPOSTメソッドでHTTPリクエストを出す例として、「Mastodon API を利用してトゥートする関数」も実装してあったり。 __（※↑の「悪い所」で前述した問題点／危険性があるので、そのままでは使えないようセーフティをかけてあります）__

…と、「お手軽＆いろいろ自分好みにカスタムできる点」については、  
他のサービスとかと比べてもいいところでは…？と感じたので、  
GitHubの練習がてら公開してみることにした次第です。


# 環境作成・設定方法、使い方の説明は？

ここで画像付きで説明…してみようとして挫折したので、  
次の PDFファイル (詳細説明スライド) をご覧くださいまし…。

[WebClapModoki_GAS 処理内容・導入手順の詳細説明スライド](https://)

一応、文章での簡易説明も記述しておきます。

<!-- 外部参照リンクの定義 -->
[SS_template]: https://docs.google.com/spreadsheets/d/1j_h4weHihW-uVeTFtHTwrs6QwRPAT36sEBf3DPbo3ms/edit?usp=sharing
[WebClapModoki_GAS]: /WebClapModoki_GAS_v7.txt
[dic_wcm]: /dic_wcm.txt

## 手順1.  
__ご自身の Google ドライブ__ に、[このスプレッドシート](SS_template)のコピーを作成してください。  
Googleアカウントでログインしたブラウザ上で開けば、メニューの[ファイル]→[コピーを作成]で簡単にコピーできます。  
また、名前（WebClapModoki (template) のコピー）は好きに変えて大丈夫です。

## 手順2.  
同じくご自分の Google ドライブに「新規 Google Apps Script」を作成し、  
[WebClapModoki.txt](WebClapModoki_GAS) の中身をコピペして、上書き保存してください。  

【注意】  
1.のスプレッドシート、2. の Google Apps Script の共有設定  
（「右クリックメニュー」→「共有」→「一般的なアクセス」の設定値）は __「制限付き」でOK__ です。  
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
特にエラーが出ず、実行ログに「お知らせ　実行完了」と表示されればOKです。  

エラーが出た場合、  
「3.で置換した 'id' の指定をミスっていないか」  
「きちんと全体を過不足なくコピペできているか」あたりを見直してみるといいかもです。

なお、  
1.でコピーしたスプレッドシートの "posted_comment" シートの「コメント」列に、  
「いいね！ 【NG】」が入力されているのは、正常動作です。  
（checkContentTextByNGWordsList() 関数が仕事して、  
　URLらしき文字列（http://www）を【NG】に置換した結果です）

使っていて「ちょっとこれは見たくないな…」という文字列が出てきたら、  
"NG_words_list" 上に行を追加して、単語を増やしたり書き換えたりしてみてください。 

## 手順5.  
4.で問題がなければ WebClapModoki_GAS を __Webアプリとしてデプロイ__ し、  
```
https://script.google.com/macros/s/YYY~YYY/exec  
```
形式のURLを取得してください。

## 手順6.
5.で取得したURLを [dic_wcm.txt](dic_wcm) の中の  
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
もし、使い始めてから微妙だなー、もう使うの止めたいなー、と思った時は、  
ゴーストさん側の記述（dic_wcm.txt）を削除した上で、  
Google ドライブ上にある 2. の Google Apps Script のファイルを削除しちゃえばOKです。  


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

本スクリプト (WebClapModoki_GAS.txt) は NYSL Version 0.9982 に従います。  
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
ゴーストさんに、Web拍手経由で気軽に感想を送信できる仕組みが実装されている例を見ていなければ、  
[2022年度の伺的アドカレー（伺的 Advent Calendar 2022 - Adventar）](https://adventar.org/calendars/831) が開催されておらず、他の方々の記事を読んで刺激を受けていなければ、  
また、 [ぽなさんのトゥート](https://ukadon.shillest.net/@ponapalt/109520146570230914) で背中を押して頂いていなければ、  
本記事を公開することはありませんでした。

ユーザ歴ばかり長い新参者ですが、今後とも何卒よろしくお願い申し上げます…。
