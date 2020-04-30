# はじめに

本記事は，「jsPsychによる心理学実験作成チュートリアル」の第5回の記事です。[第4回](https://qiita.com/snishym/items/ccbf53e64e313584dd48)では`timelineVariables`の使い方を紹介しました。今回は，実験データを手元のPCに保存する方法を紹介します。残念ながら，**今回紹介する保存方法は，オンライン実験を実施する際には利用できません**。また，このチュートリアルではオンライン実験でのデータの保存方法を紹介しません。利用するサーバー（jsPsychがアップロードされていて，参加者がアクセスする場所）によってその方法は異なるからです。[jsPsychの公式サイト](https://www.jspsych.org/overview/data/)でも一部のサーバーに利用できる保存方法を紹介してあるので，これからサーバーを借りてオンライン実験を実施するという場合は，掲載されている方法をそのまま利用できるサーバーを探してみるといいかもしれません。

オンライン実験でのデータの保存方法もjsPsychの重要な要素ですが，それ以外にもjsPsychでのデータの扱い方について知っておくことはあるので，本記事では合わせてそれらを紹介します。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# データを手元のPCに保存する

データを手元のPCに保存する方法は非常にシンプルです。`jsPsych.init()`の中に新たに`on_finish`に続く3行を足せばいいだけです。

```javascript
jsPsych.init({
  timeline: [{
    timeline_variables: trial_types,
    timeline: [trial],
  }],
  default_iti: 250,
  // ここから
  on_finish: function() {
    jsPsych.data.get().localSave('csv', 'data.csv');
  },
  // ここまで
});
```

前回作成したコードに`on_finish`の3行を追加して実行すると，課題の終了後，`data.csv`という名前のファイルが，ブラウザの設定にしたがって保存されます。デフォルトなら「ダウンロード」という名前のフォルダに自動で保存されていると思います。設定内容を確認する方法は，「ブラウザ名 + ファイル + ダウンロード先」とかでweb検索すると見つかるはずです。

`on_finish`という名前にあるように，終了時に`function(){}`内に入っている処理を実行しています。`jsPsych.data`というところに保存されているデータを取り出し`.get()`手元のPCにcsv形式で「data.csv」という名前で保存するようにしています`.localSave('csv', 'data.csv')`。csvというのは**c**omma **s**eparated **v**alue(s)の略で，値（各データ）がカンマ`,`区切りで並べられて書かれているファイルです。対応しているアプリケーションも多く，エクセルのような表計算ソフトで開くことはもちろん可能ですし，PythonやRといったプログラミング言語でデータ分析をする際にも扱いやすいファイル形式です。

さっそくデータの中身を確認してみましょう。かなり横に長くこのページでは見にくくなるので，ここには載せません。ちなみに，VS codeをお使いの方は「Rainbow CSV」という拡張機能をインストールするとVS Code上でもcsvの内容が確認しやすくなります。

さて，先程追加した3行以外に「〜〜のデータを保存する」ことを指定するコードは書いていませんが，さまざまなデータが保存されています。ただ厄介なのは，`stimulus`と`key_press`です。`stimulus`には，実際に入力された形式のまま保存されます。前回紹介した`timelineVaribles`の使い方のどの方法を用いたとしても，この`<div>`タグ付きの文字列が保存されることになります。これだと見にくいです。また，保存されたデータには各試行の条件を判別できる列がこれ以外にはないため，`stimulus`に保存されているデータから判定していくことになりますが，すこしテクニックが要りそうです。できることなら，（提示された文字と位置と）条件が最初からデータに保存されている状態にしておきたいです。

`key_press`には，入力されたキーが保存されているはずですが，実際に入力したキーの文字（`f`や`j`）ではなく，数字が保存されています。これはバグっているわけではなく，保存されている数字は`f`と`j`に対応するキーコードです。第1回の「キー入力を指定する」というセクションでも軽く紹介しました。このままだと，分析の際にいちいちキーコードと対応する文字を確認する必要があるので放っておくのは不便です。（入力されたキーと）入力の正誤があるとその手間が省けて良さそうです。

ということで，残りの部分では，提示された文字，位置，条件，入力されたキー，入力の正誤の5つを保存する方法を紹介します。

# 提示された文字，位置を保存する

第4回で紹介した，`trial_types`で`letter`と`pos`を分ける方のコードに以下のように4行追加すれば，提示された文字と位置をデータに簡単に保存することができます。

```javascript
var trial_types = [
  {letter: 'L', pos: 'left'},
  {letter: 'R', pos: 'left'},
  {letter: 'L', pos: 'right'},
  {letter: 'R', pos: 'right'},
];

var trial = {
  type: 'html-keyboard-response',
  stimulus: function(){
    return '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
  },
  choices: ['f', 'j'],
  trial_duration: 1000,
  // ここから
  data: {
    letter: jsPsych.timelineVariable('letter'),
    pos: jsPsych.timelineVariable('pos')
  },
  // ここまで
}
```

この`data: {}`の中には`timelineVariable`以外も入れることができます。以下のように`hogehoge: "gehogeho"`を追加すると，サイモン課題のデータすべてに"gehogeho"という文字列が保存された列ができます。`hogehoge: "gehogeho"`は実用的ではありませんが，複数の課題を遂行させるような実験を実施する場合には，分析の際にどのデータがどの課題のものかを判別できるように，`task: "simon"`のようなデータを保存しておく必要がありますので，そのような際に活用してください。

```javascript
data: {
  letter: jsPsych.timelineVariable('letter'),
  pos: jsPsych.timelineVariable('pos'),
  hogehoge: "gehogeho",
}
```

# 各試行の条件を保存する

データに追加した文字`letter`と`pos`情報から，各試行の条件を判定して，データとして保存されるようにしましょう。条件の判定には`if`文を使用します。試行ごとに`if`文を実行してデータに保存するには，`trial`変数にも`on_finish:`という項目を追加します。

## if文

if文は以下のように書いて，ifの直後に指定されている条件が真`true`なら`{}`内の処理を実行します。

```javascript
if (条件式) {
  何らかの処理
}
```

例えば，以下のように書けます。

```javascript
var a = 1;
if (a == 1) {
  処理A
}
var b = 2;
if (b == 1) {
  処理B
}
```

この例は，`a`と`b`が`1`であればそれぞれ処理A，処理Bが実行されるというものです。`==`は等価演算子と呼ばれるもので，左右の値が等しいかどうかを判定し，等しければ`true`，そうでなければ`false`を返します。今回，`a`には1が入っているので，`a == 1`は`true`になり，処理Aは実行されます。一方で，`b`には2が入っているので，`b == 1`は`false`となり処理Bは実行されません。

`if`に合わせて`else if`，`else`というものも利用できます。まず`else`について紹介します。

```javascript
var b = 2;
if (b == 1) {
  処理B
} else {
  処理C
}
```

`else`を使うと，直前の条件式が`false`の場合に必ず実行する処理を記述することができます。上記の例であれば，`b == 1`は`false`なので，処理Cが実行されます。

そして，`else if`は以下のように使うことができます。

```javascript
var b = 2;
if (b == 1) {
  処理B
} else if (b == 3){
  処理C
}
```

`else if`は見た目の通り，`else`に`if`を組み合わせたもので，直前の条件式が`false`の場合に`()`内の条件式の真偽を判定し，`true`なら`{}`内の処理を実行します。上記の例であれば，`b == 3`は`false`になるので，処理Cも実行されません。

`if`，`else`，`else if`は組み合わせて使うことができます。以下の例では，`b == 1`, `b == 3`のどちらも`false`になるので，処理Dが実行されます。

```javascript
var b = 2;
if (b == 1) {
  処理B
} else if (b == 3){
  処理C
} else {
  処理D
}
```

今回のサイモン課題であれば，「『文字がL かつ 位置が左』または『文字がR かつ 位置が右』」なら一致条件，そうでなければ不一致条件ということになるので，それを判定する条件式は以下のように書けます。`cond`は条件conditionの最初の4文字です。

```javascript
if ((letter == 'L' && pos == 'left') || (letter == 'R' && pos == 'right')) {
  cond = 'cong'
} else {
  cond = 'incong'
}
```

最初のifの条件式がかなり長いですが，「『文字がL かつ 位置が左』または『文字がR かつ 位置が右』」を変換しただけです。「かつ」が`&&`，「または」が`||`になります。条件式を囲む`()`と，「Lかつ左」「Rかつ右」を囲む`()`があるので注意してください。横に長いのが嫌な人は，今回の例であれば，`else if`を以下のように利用して横方向の長さを回避することができます。

```javascript
if (letter == 'L' && pos == 'left') {
  cond = 'cong'
} else if (letter == 'R' && pos == 'right') {
  cond = 'cong'
} else {
  cond = 'incong'
}
```

## 試行ごとに`if`文を実行してデータに保存する

今回の記事の一番初めに，実験データを保存するために`jsPsych.init()`の中に`on_finish`という項目を設けて，実験終了時にデータを保存する処理が実行されるようにしました。実は，試行変数`trial`にも`on_finish`の項目は設け，各試行の最後に実行する処理を指定することができます。

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  data: {
    letter: jsPsych.timelineVariable('letter'),
    pos: jsPsych.timelineVariable('pos')
  },
  // ここから
  on_finish: function() {
    何らかの処理
  }
  // ここまで
```

そして，`on_finish: function(data){...}`とすることで，`{}`内の処理でその試行変数の`data`にアクセスできるようになります。今回は，各試行に提示された文字と位置を元にして条件を判定することが目標なので，`{}`内に先程作成した`if`文を代入すればいいわけです。`data`に保存された文字`letter`と位置`pos`には`data.letter`，`data.pos`でアクセスすることができます[^1]。

ということで，それらを組み合わせると以下のようになります。

[^1]: `function(){}`の`()`内はdataでなくても，`d`とか`hogehoge`でも構いません。`{}`内の処理を実行するときに，`()`内で指定した名前で，試行変数の`data`にアクセスできます。したがって，`function(hoge){}`とした場合は`hoge.letter`で各試行で提示された文字を参照できます。

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  data: {
    letter: jsPsych.timelineVariable('letter'),
    pos: jsPsych.timelineVariable('pos')
  },
  // ここから
  on_finish: function(data) {
    if (data.letter == 'L' && data.pos == 'left') {
      data.cond = 'cong'
    } else if (data.letter == 'R' && data.pos == 'right') {
      data.cond = 'cong'
    } else {
      data.cond = 'incong'
    }
  }
  // ここまで
}
```

if文の例で`cond`だった部分も`data.cond`に変更しています。このように，`data`に新しいデータを追加したい場合は，`data.データ名 = データ`で追加することができます。

## if文を使わなくても良い

そもそも，上記のような面倒なことをしなくても，今回のサイモン課題程度なら`trial_types`配列内の連想配列に`cond: "cong"`などを追加してしまうほうが，データに条件を追加するときにも`jsPsych.timelineVariable('cond')`と1行で済むので簡単です。

```javascript
var trial_types = [
  {letter: 'L', pos: 'left', cond: 'cong'},
  {letter: 'R', pos: 'left', cond: 'incong'},
  {letter: 'L', pos: 'right', cond: 'incong'},
  {letter: 'R', pos: 'right', cond: 'cong'},
];

var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  data: {
    letter: jsPsych.timelineVariable('letter'),
    pos: jsPsych.timelineVariable('pos'),
    cond: jsPsych.timelineVariable('cond'),
  }
  // この場合はon_finishは要らない
};
```

# 入力されたキーの文字，正誤を保存する

入力されたキーはデータファイルの`key_press`の列に保存されていますが，それが実際の文字ではなく，キーコードで保存されていることは今回の記事の序盤に述べたとおりです。入力されたキーを文字で保存したい場合はこのキーコードを文字に変換しなければなりません。大変そうですが，実は，jsPsychには`jsPsych.pluginAPI.convertKeyCodeToKeyCharacter()`という関数が用意されており，`()`内にキーコードを入れるだけで，対応する文字に変換してくれます。便利ですね。`key_press`の列に保存されるデータには，`on_finish: function(data){}`の`{}`内で`data.key_press`とすることでアクセスすることができます[^2]。また，`data.任意の名前 = データ`で新しくデータを追加できることは直前のセクションで述べたとおりです。これらをまとめると，以下のようにして，入力されたキーの文字列を保存することができます。

[^2]: 保存されたデータに表示された列名を使って，`data.列名`とすれば，その列のデータにアクセスすることができます。例えば，`data.rt`とすれば，反応時間を取り出すことができます。`data: {letter: ..., pos: ...}`でデータを追加した場合は，`data.letter`などでアクセスすることができ，それらはデータファイル内で`letter`，`pos`という列に保存されるというところからも，`data.列名`でデータにアクセスすることができるということがわかると思います。

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  // ここから
  on_finish: function(data) {
    data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)
  }
  // ここまで
}
```

この例だと，`key`という列名でファイルに保存されることになります。

このままの勢いで，入力されたキーの正誤判定までやってしまいましょう。あまりちゃんと説明していませんでしたが，今回のサイモン課題では，Lが提示されたら左手（人差し指）で`f`キーを，Rが表示されたら右手（人差し指）で`j`キーを押すことが求められます。つまり，「提示された文字列が`L`のときは`f`,`R`のときは`j`」が入力されていたら正反応ということになり，「そうでない場合」は誤反応ということになります。これまでの内容で実装することができそうです。コード例は下に記しますが，ぜひご自身でまずは挑戦してみてください。ちなみに，以下のコード例では，正なら1,誤なら0で保存しています。こうすれば，分析の際にデータの変換をせず，平均を取るだけで正反応率を算出できるからです。

<details><summary>コード例その1</summary><div>

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  on_finish: function(data) {
    data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)
    if (data.letter == 'L') {
      if (data.key == 'f') {
        data.correct = 1
      } else {
        data.correct = 0
      }
    } else {
      if (data.key == 'j') {
        data.correct = 1
      } else {
        data.correct = 0
      }
    }
  }
}
```

一番地道なコードです。if elseを繰り返し使うことになって`{}`が増えて読みにくくなってしまうのが難点です。

</div></details>

<details><summary>コード例その2</summary><div>

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  on_finish: function(data) {
    data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)

    data.correct = 0
    if (data.letter == 'L' && data.key == 'f') {
      data.correct = 1
    } else if (data.leter == 'R' && data.key == 'j') {
      data.correct = 1
    }
  }
}
```

どうせ`else`で`0`を保存するのなら，初めに0を`data.correct`に入れておいて，正反応と判定されたら`1`に書き換えようという発想のコードです。elseだと3行使うのが，1行にまとまるので，省スペースですね。例その1と比べると，`data.letter == 'L' && data.key = 'f'`などを使ったことのほうが省スペースに貢献していますが。。。

</div></details>

<details><summary>コード例その3</summary><div>

```javascript
var trial = {
  type: 'html-keyboard-response',
  // stimulus: などは省略
  on_finish: function(data) {
    data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)
    if (data.letter == 'L') {
      data.correct = Number(data.key == 'f')
    } else {
      data.correct = Number(data.key == 'j')
    }
  }
}
```

たぶん，この書き方が一番行数が少なくなると思います。`data.key == 'f'`は`true`か`false`を返しますが，`Number()`で数字に変換すると，それぞれ`1`，`0`になります。

</div></details>

いくつか例を載せましたが，他にも書き方はいくつかあると思います。とりあえず回ればどんなコードでもいいと思います。

# おわりに

今回の記事の内容をこれまでのコードに追加すると以下のようになります。

<details><summary>コード例</summary><div>

```html:save-data.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(-50%);
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(50%);
    }
  </style>
</head>
<body></body>
<script>

  var trial_types = [
    {letter: 'L', pos: 'left', cond: 'cong'},
    {letter: 'R', pos: 'left', cond: 'incong'},
    {letter: 'L', pos: 'right', cond: 'incong'},
    {letter: 'R', pos: 'right', cond: 'cong'},
  ];

  var trial = {
    type: 'html-keyboard-response',
    stimulus: function(){
      return '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
    },
    choices: ['f', 'j'], // 入力キーの指定
    trial_duration: 1000, // 試行の持続時間
    data: {
      letter: jsPsych.timelineVariable('letter'),
      pos: jsPsych.timelineVariable('pos'),
      cond: jsPsych.timelineVariable('cond'),
    },
    on_finish: function(data) {
      data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)

      if (data.letter == 'L') {
        data.correct = Number(data.key == 'f')
      } else {
        data.correct = Number(data.key == 'j')
      }
    }
  }

  var trial_types = jsPsych.randomization.repeat(trial_types, 2);

  jsPsych.init({
    timeline: [{ // [ と { 2つあります
      timeline_variables: trial_types,
      timeline: [trial],
    }], // } と ] 2つあります
    default_iti: 250,
  });
</script>

</html>
```

</div></details>

今回は実験データの追加・保存方法について紹介しました。気づいたらif文についても紹介していました。for文と合わせてjavascript（やその他多くのプログラミング言語）の基本構文になるので，ぜひ慣れ親しんでいただければと思います。

今回の記事では，例として，文字・位置・入力されたキーを追加でデータとして保存できるようにしましたが，分析で必要なのは，条件と正誤（とRT）なので，文字などを保存する必要はないかもしれません。なにはともあれ実際に実験が実施できそうなプログラムになってきました。あとは教示と参加者情報の取得になります。引き続き頑張りましょう。

[【第6回】参加者情報の取得](https://qiita.com/snishym/items/e0f82fa972970cda632c)