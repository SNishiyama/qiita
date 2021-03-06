# 2021/03/15 補足 (2021/07/08追記)

2021/03/15 時点での jspsych の最新バージョンは `v6.3.0` です。本チュートリアルのコードは`v6.1.0`が最新だった 2020 年 4 月頃に作成し，それから更新しておりません。チュートリアル内で紹介されているコードの大体は互換性があります（＝バージョンに関係なくそのまま利用できる）が，現在のところ，次の 3 点については互換性がないことを把握しております（コード自体は未修正）。

1. `v6.3.0`では，`survey-text`など`survey-`系のプラグインで取得したデータを取り出す際に，`JSON.parse`を使う必要がなくなった（第 6 回に関連）。
2. `v6.3.0`では，ある試行で得られた反応を取り出す際に，どのプラグインでも`response`で参照できるようになった（いろんな回に関連）。
3. `v6.3.0`では，関数内で`jsPsych.timelineVariables()`を使用する時に，引数に`true`と指定する必要がなくなった（特に第 4 回）。←**互換性ありました。古いバージョンで作成したコードでもこの点についてはそのままで動作します(2021/07/08)**

2 に関して，例えば，チュートリアルの中で一番よく使っている`html-keyboard-response`で得られたキー反応を参照するために，`v6.2.0`までは`変数名.key_press`で取得していましたが，`v6.3.0`（以降）では`変数名.response`に変わります。

また時間ができたら対応したいと思います。ひとまずアナウンスだけになりますが，ご了承ください。jsPsych の更新履歴は[公式 github のリリース情報](https://github.com/jspsych/jsPsych/releases)から確認できます。

# 2020/12 頃 補足

このチュートリアルのつぎのステップとして読むといいかもしれない記事を zenn というサイトで公開しています（[zenn の マイページ](https://zenn.dev/snishiyama)）。今後は zenn で記事を公開していくつもりなので，そちらもたまに見に来ていただけると幸いです。

# はじめに

本シリーズは，サイモン課題の作成を通して，jsPsych で心理学実験を行うために最低限必要なスキルを身につけることを目標としています。

具体的には，以下の 4 つです。

1. 参加者情報の入力（[第 6 回](https://qiita.com/snishym/items/e0f82fa972970cda632c)）
2. 課題の教示（[第 7 回](https://qiita.com/snishym/items/2296fc1aff6d711aaa0b)）
3. 刺激のランダム提示（[第 1 回](https://qiita.com/snishym/items/45cfd220f7af8c7a13e0)，[第 2 回](https://qiita.com/snishym/items/a121a4d6a02c71b69f3e)，[第 3 回](https://qiita.com/snishym/items/bec56308c67922c3b3df)，[第 4 回](https://qiita.com/snishym/items/ccbf53e64e313584dd48)）
4. データの保存（[第 5 回](https://qiita.com/snishym/items/be23aa7cbeeffa49d13a)）

想定しているターゲットは，どの言語のプログラミング経験もない人です。javascript（や，HTML, css）の基本を知っている必要はありません。シリーズでは, jsPsych で心理学実験を作成するのに必要なテクニックを javascript の基本的な構文も織り交ぜながら順番に説明していきます。もちろん，javascript は知っているが jsPsych はこれからという人にも有用な資料になっているはずです。

なお，このチュートリアルは私が以前作成した「[PsychoPy Coder による心理学実験作成チュートリアル](https://qiita.com/snishym/items/8b52db0d901cf5744463)」を jsPsych 用に書き換えたものです。

# jsPsych とは

jsPsych は，javascript というプログラミング言語で書かれた心理学実験・調査作成ツール（ライブラリ・プラグイン）です。そして，javascript は Google Chrome や Firefox, Safari といったウェブブラウザ上で動作するプログラミング言語です。したがって，jsPsych もウェブブラウザ上で動作します。つまり，ウェブブラウザ上で心理学実験を動作させることができます。ウェブブラウザは，インターネットを介して世界の誰かの記事・ブログを読むためのものです。それらウェブページと同様に，jsPsych で作成した実験をウェブ上にアップロードしておけば，世界のどこかの誰かさんがインターネットを介して自分の作成した心理学実験に参加することができます。つまり，「オンライン実験」を実施することができます。もちろん，jsPsych で作成した実験を PC にダウンロードしておけば，インターネットがなくてもその PC 上で実験を動作させることができるので，jsPsych でも実験室実験を実施することができます。

オンライン実験作成が jsPsych の主な特徴になると思うのですが，他にも同様のツールは結構あるようです（[Anwyl-Irvine ら(2020)](https://link.springer.com/article/10.3758/s13428-019-01237-x)の Table 1）。その中でも私が jsPsych を使おうと思った理由は以下の３つです。

1. 無料で使える[^1]
2. スクリプトを書いて作成する
3. 日本語の資料がある

[^1]: ただし，オンライン実験を実施するためにはアップロードしておくサーバーが必要で，多くの場合，レンタルサーバーを借りることになるので，その実施には謝礼とは別にお金がかかります。

2 つ目の点が個人的には結構大事でした。無料で使えて，日本語の資料があるツールとしては，PsychoPy Builder や lab.js が挙げられます。これらのツールは jsPsych と違って，基本的に GUI でマウスでポチポチ実験を作成することができるツールです。GUI のツールは、最初の垣根は低いのですが、工夫し始めると一気に手順がややこしくなったり、コードを書いたりする必要が出てきたりするという印象があります。コードを書くとなった場合にはツールのベースにあるプログラミング言語（PsychoPy であれば python, lab.js であれば javascript）について結局勉強する必要があります。

だったら，初めからコードでの実験の作り方を覚えたほうが長期的にはいいんじゃないかと思います。コーディングは最初の垣根が GUI より高いかもしれませんが、慣れてしまえば、GUI よりも楽に自由に実験を組めるようになります。その知識は他のプログラミング言語を勉強するときにも応用できるので、その際の学習コストも低くなります。私の場合は，PsychoPy を使ってコードを書いて心理学実験を作ってきたので，jsPsych にはすぐに慣れることができました。

とはいえ，この辺は作成する予定の実験の複雑さに左右されます。GUI ツールでも全然事足りる場合はあるので，全く心理実験を作ったことがないという方は，まずは GUI ツールでの作成から始めてみるのがいいと思います。

# 参考資料

本チュートリアルの作成に際し，以下の資料を参考にいたしました。作成者のみなさまには感謝申し上げます。

- [jsPsych 公式サイト](https://www.jspsych.org/)
- ダウンロードした jsPsych に同梱されているサンプルコード（examples フォルダ内）
- [黒木先生の「ウェブブラウザで心理学実験と調査 jsPsych」](https://sites.google.com/site/webdeshinri/home)
- [高橋先生の「キソジオンライン」](https://github.com/kohske/KisojiOnline)
- [国里先生の「jsPsych を用いた認知課題の作成」](https://kunisatolab.github.io/main/code_tips.html)
- [小林先生の「jsPsych チュートリアル」](https://www.notion.so/jsPsych-73cade0a2e044217aedf01b5845e8d4e)

黒木先生のサイトについては掲載のコードを見る限り，現在のバージョンと互換性がなさそうなので，注意が必要かもしれません。高橋先生のキソジオンラインは，jsPsych の解説資料ではありませんが，jsPsych で作成された心理学実験のコードがたくさんアップされています。jsPsych がある程度読み書きできるようになったら，とても参考になると思います。国里先生のページでは Rstudio という開発環境上で jsPsych ベースの心理実験を作成する方法が紹介されています。Rstudio に馴染みのある方はここから始めてみてもいいかもしれません。小林先生のページにはこれら 3 つのサイトとは少し種類の違う実験課題のコードが掲載・解説されています。

# おわりに

本記事は，本シリーズのまとめページとして概要を説明しました。正直に言うと，サイモン課題は，GUI ツールで簡単に作成できる課題です（[PsychoPy Builder での作成](http://www.s12600.net/psy/python/ppb/html/chapter03.html)，[lab.js での作成](https://www.notion.so/5-8cad725e57ac4a3382cc0e9d3c82d407)）。

それでも，本シリーズが，なにかのきっかけに jsPsych で書きたい・書かないといけないという方のお役に立てれば大変幸いです。
