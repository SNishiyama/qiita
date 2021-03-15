# 2021/03/15 補足

2021 年 2 月末くらいにリリースされた`v6.3.0`になって`survey-text`や`survey-multi-choice`など`survey-`系のプラグインで得られた反応を利用するための処理が少し変わりました。

1. `v6.3.0`では，`survey-text`など`survey-`系のプラグインで取得したデータを取り出す際に，`JSON.parse`を使う必要がなくなりました。
2. また，`変数名.responses`ではなく，`変数名.response`で得られたデータを参照するようになりました（複数形の s がいらない）。

まとめると，以下のサンプルコードで`par_info.id = JSON.parse(data.responses).participantID`となっていたものが，`par_info.id = data.response.participantID`で良くなります（※未検証）。最新版をダウンロードしてこのチュートリアルに望んでいる場合は，その点に留意して以下の内容を読んでください。そのうちちゃんと修正しますが，ひとまずアナウンスにてご了承ください。その他重大な変更点については[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)を参照してください。

# はじめに

本記事は，「jsPsych による心理学実験作成チュートリアル」の第 6 回の記事です。[第 5 回](https://qiita.com/snishym/items/be23aa7cbeeffa49d13a)では実験データの保存方法を紹介しました。今回は，参加者情報の取得について紹介します。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# 参加者情報の取得

さっそく参加者情報を取得していきましょう。今回のチュートリアルで収集する情報は，参加者 ID，年齢，性別の 3 つとします。参加者 ID と年齢は`jspsych-survey-text`，性別は`jspsych-survey-multi-choice`というプラグインを使って収集していきます。まずは以下のコードを実行してみてください。

```html:get_participant_info.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8"> <!-- これを足す -->
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-survey-text.js"></script>
  <script src="../plugins/jspsych-survey-multi-choice.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var par_id = {
    type: 'survey-text',
    questions: [
      {prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID'},
    ],
    button_label: '次へ',
  };

  var age = {
    type: 'survey-text',
    questions: [
      {prompt: '年齢を入力してください', columns: 3, required: true, name: 'age'},
    ],
    button_label: '次へ',
  };

  var gender = {
    type: 'survey-multi-choice',
    questions: [
      {prompt: "性別を選択してください", options: ['男性', '女性'], required: true, horizontal: true, name: 'gender'},
    ],
    button_label: '次へ',
  };

  jsPsych.init({
    timeline: [par_id, age, gender],
    on_finish: function() {
      jsPsych.data.get().localSave('csv', 'data.csv');
    },
  });
</script>

</html>
```

全部で 3 ページの実験（？）プログラムが実行されたはずです。まず，本題とは逸れますが，質問文に日本語を使っているので，`<head>`の直後に`<meta charset="utf-8">`が挿入されています（第 1 回参照）。

さて，本題です。`survey-text`と`survey-multi-choice`のどちらも`questions:`がメインの設定項目になります。どちらも`questions:[]`と`[]`内に質問の内容や解答欄・選択肢を設定するための連想配列を持ってきますが，その連想配列内で設定できる内容が`text`と`multi-choice`で少し異なります。

`survey-text`の場合，`columns`という設定ができます。これは，回答を入力するためのテキストボックスの幅を決める項目で，数字が大きいほど幅が広くなります。適宜，想定される回答の長さに応じて変更してください。
`survey-multi-choice`の場合，`options`という設定ができます。これは，選択肢を指定する項目で，例のように，配列を使います。また，`horizontal`という設定もあります。これは選択肢を横(`horizontal`)に並べるか，縦に並べるかどうかを指定できます。`horizontal: true`で横に並べることができます。また，この項目を設定しなければ，jsPsych は自動的に選択を**縦に並べます**。

残りは共通の設定で，`prompt`には質問の内容，`required`では回答を必須とするかどうか，`name`には回答がデータに保存されるときの名前を指定しています。`required`で回答を必須にしていないと無回答で次に進めてしまうので，参加者情報を集める場合に限っては常に`true`でいいでしょう。

なお，`questions`は複数形になっている通り，`[]`内に質問用の連想配列を複数持ってくることができます。その場合，1 つのページに 2 つ以上の質問を提示することができます。適宜利用してください。本チュートリアルでは参加者 ID と年齢を尋ねる質問は別々のページで提示することにします。

```javascript
var parID_age = {
  type: 'survey-text',
  questions: [
    { prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID' },
    { prompt: '年齢を入力してください', columns: 3, required: true, name: 'age' },
  ],
  button_label: '次へ',
};
```

## 回答の保存方法をアレンジする

次に，回答が保存されたファイルを確認してみましょう。`responses`という列に回答が保存されていますが，困ったことに`{"participantID":"kla"}`のように連想配列の形式で保存されてしまっています。これでは分析しにくいので，回答の部分だけを取り出して別の列に保存する方法を考えていきましょう。

個人的に一番シンプルだと思う方法は，回答が終了したタイミングで反応を取り出しておくというものです。前回，`on_finish`という項目を設けて，`on_finish: function(data) {...}`とすることで，各試行の終了時にその試行で得られたデータにアクセスし，回答の正誤などを判定する方法を紹介しました。これと同様の方法を用います。具体的には，以下のようにします。

```javascript
var par_info = {};

var par_id = {
  type: 'survey-text',
  questions: [{ prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID' }],
  // ここから
  on_finish: function (data) {
    par_info.id = JSON.parse(data.responses).participantID;
  },
  // ここまで
};
```

まず，`JSON.parse(data.responses).participantID`について説明します。`data.列名`で保存された特定のデータにアクセスできることは前回紹介したとおりです。先ほど保存されたファイルを確認した際に，回答は`responses`という名前の列に保存されていたので，`data.responses`で回答を取り出すことができます。しかしながら，この時点では`data.responses`に保存されている回答は json という形式になっています。これを javascript の連想配列に変換するために，`JSON.parse()`を使います。ここでようやく`{participantID:"kla"}`という連想配列が得られます。`連想配列.キー名`でそのキーに対応する値を取り出すことができるので，`.participantID`で`"kla"`という文字列を得ることができます。このキー名は，質問項目の設定`name:`で指定した文字列に一致します。

次に，`par_info.id =`について説明します。ここでは，参加者 ID を事前に作った連想配列`var par_info = {}`に一時保存しています。前回の記事では，`on_finish: function(data) {...}`の中で`data.新しい列名 = 値`とすることで，任意の値を新しい列に保存できることを紹介しました。同様に今回も，`data.id = JSON.parse(...)`として保存しても構わないのですが，その場合参加者 ID はたった 1 行にしか保存されません（試してみてください）。参加者 ID，性別などの参加者情報はデータのすべての行にあるほうが分析がしやすい場合が多いので，実験の最後に「すべての行に参加者情報を保存する」という処理が実行できるように，ここでは別の連想配列に一時保存しています。

上記のコードを少しずつ変更して，他の参加者情報も一時保存するコードは以下のようになります。ここでは合わせて，最後に一時保存した参加者情報をすべての行に追加して保存しています。

```html:save_participant_info.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8"> <!-- これを足す -->
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-survey-text.js"></script>
  <script src="../plugins/jspsych-survey-multi-choice.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var par_info = {};

  var par_id = {
    type: 'survey-text',
    questions: [
      {prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.id = JSON.parse(data.responses).participantID // 一時保存
    }
  };

  var age = {
    type: 'survey-text',
    questions: [
      {prompt: '年齢を入力してください', columns: 3, required: true, name: 'age'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.age = JSON.parse(data.responses).age // 一時保存
    }
  };

  var gender = {
    type: 'survey-multi-choice',
    questions: [
      {prompt: "性別を選択してください", options: ['男性', '女性'], required: true, horizontal: true, name: 'gender'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.gender = JSON.parse(data.responses).gender // 一時保存
    }
  };

  jsPsych.init({
    timeline: [par_id, age, gender],
    on_finish: function() {
      jsPsych.data.addProperties(par_info); // すべての行に追加
      jsPsych.data.get().localSave('csv', 'data.csv');
    },
  });
</script>

</html>
```

終盤にある`jsPsych.data.addProperties()`という関数が，すべての行に参加者情報を追加している部分になります。保存された csv ファイルを確認すると，`par_info.id`, `.age`, `.gender`のそれぞれ`id`, `age`, `gender`が列名になっていて，それらの列のすべての行に，回答が保存されていると思います。

# timeline_variables を使ってみる（余談）

先ほどのコードでは，参加者情報を集めるために`par_id`，`age`，`gender`と 3 つの試行変数を作りました。しかし，それぞれの中身の共通部分は多いので，`timeline_variables`を使ってまとめてしまいましょう。紹介はしますが，必須ではないです。

<details><summary>コード例</summary><div>

```html:get_info_timeline_variables.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8"> <!-- これを足す -->
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-survey-text.js"></script>
  <script src="../plugins/jspsych-survey-multi-choice.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var question_settings = [
    {type: 'survey-text', colname: 'ID', question: {prompt: '参加者IDを入力してください', columns: 10, required: true}},
    {type: 'survey-text', colname: 'age', question: {prompt: '年齢を入力してください', columns: 3, required: true}},
    {type: 'survey-multi-choice', colname: 'gender', question: {prompt: "性別を選択してください", options: ['男性', '女性'], required: true, horizontal: true}},
  ];
  var par_info = {};

  var get_info = {
    type: jsPsych.timelineVariable('type'),
    questions: [jsPsych.timelineVariable('question')],
    button_label: '次へ',
    on_finish: function(data) {
      var colname = jsPsych.timelineVariable('colname', true);
      par_info[colname] = JSON.parse(data.responses).Q0
    }
  };

  jsPsych.init({
    timeline: [{
      timeline_variables: question_settings,
      timeline: [get_info],
    }],
    on_finish: function() {
      jsPsych.data.addProperties(par_info); // すべての行に追加
      jsPsych.data.get().localSave('csv', 'data.csv');
    },
  });
</script>

</html>
```

`type`，`questions`の中身，`colname`（保存時の列名）が質問内容によって変わるので，各ページでそれらが変更されるように，`question_settings`という`timeline_variables`を作っています。一つ前の例のコードと見比べてどのように`jsPsych.timelineVariable`を使っているかを確認してください。

2 点，これまで説明していなかったものがあるので，それについて説明します。どちらも`par_info[colname] = JSON.parse(data.responses).Q0`に関わるものです

1. `par_info[colname] =`とあるように，連想配列への要素の追加は`par_info.キー名`だけでなく，`par_info[キー名]`でも行うことができます。
2. 各質問内容の設定（`question_settigns`の`question:`）で`name:`を指定していません。指定しない場合，デフォルトの設定が反映され回答は`{Q0: 回答}`と保存されます。そうすれば，どの質問でも，`JSON.parse(data.responses).Q0`で回答を取り出せるようになります。`name:`を指定してしまうと，この`.Q0`の部分が指定された名前に変更されてしまうため，ここにも`jsPsych.timelineVariable()`を使う必要がでてきます。

</div></details>

# これまでのコードに取り入れる

最後に，今回の記事で紹介したコードをこれまでのコードと合体させます。基本的には付け足していけばいいですが，`jsPsych.init({})`内にある`timeline:[]`の部分には少し変更が必要です。前回までの記事では，以下のようにしていました。

```javascript
jsPsych.init({
  timeline: [
    {
      timeline_variables: trial_types,
      timeline: [trial],
    },
  ],
  // 以下省略...
});
```

ここに，`par_id`, `age`, `gender`を追加していく必要があります。以下のようにすればいいのですが，

```javascript
jsPsych.init({
  timeline: [
    par_id,
    age,
    gender,
    {
      timeline_variables: trial_types,
      timeline: [trial],
    },
  ],
  // 以下省略...
});
```

このままだと外側の`timeline:[]`の中身が見にくくなってしまいます。実は，`{timeline_variables: ..., timeline: ...}`の部分は事前に別のところで変数として作っておくことができます。この部分は，サイモン課題に対応しているので，下の例では`simon`という名前をつけています。

```javascript
var simon = {
  timeline_variables: trial_types,
  timeline: [trial],
};

jsPsych.init({
  timeline: [par_id, age, gender, simon],
  // 以下省略...
});
```

これで，実験の流れがかなり見やすくなったと思います。すべてまとめたコードは以下を参照してください。

<details><summary>コード例</summary><div>

```html:get_participant_info_all.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <script src="../plugins/jspsych-survey-text.js"></script>
  <script src="../plugins/jspsych-survey-multi-choice.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(-50%);
      -webkit-transform: translateY(-50%) translateX(-50%);
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(50%);
      -webkit-transform: translateY(-50%) translateX(50%);
    }
  </style>
</head>
<body></body>
<script>

  // 参加者情報の取得
  var par_info = {};

  var par_id = {
    type: 'survey-text',
    questions: [
      {prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.id = JSON.parse(data.responses).participantID // 一時保存
    }
  };

  var age = {
    type: 'survey-text',
    questions: [
      {prompt: '年齢を入力してください', columns: 3, required: true, name: 'age'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.age = JSON.parse(data.responses).age // 一時保存
    }
  };

  var gender = {
    type: 'survey-multi-choice',
    questions: [
      {prompt: "性別を選択してください", options: ['男性', '女性'], required: true, horizontal: true, name: 'gender'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.gender = JSON.parse(data.responses).gender // 一時保存
    }
  };

  // サイモン課題
  var trial_types = [
    {letter: 'L', pos: 'left', cond: 'cong'},
    {letter: 'R', pos: 'left', cond: 'incong'},
    {letter: 'L', pos: 'right', cond: 'incong'},
    {letter: 'R', pos: 'right', cond: 'cong'},
  ];

  var trial = {
    type: 'html-keyboard-response',
    stimulus: function() {
      return '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
    },
    choices: ['f', 'j'], // 入力キーの指定
    trial_duration: 1000, // 試行の持続時間
    data: {
      letter: jsPsych.timelineVariable('letter'),
      pos: jsPsych.timelineVariable('pos'),
      cond: jsPsych.timelineVariable('cond')
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

  var simon = {
    timeline_variables: trial_types,
    timeline: [trial],
  }

  jsPsych.init({
    timeline: [par_id, age, gender, simon],
    default_iti: 250,
    on_finish: function() {
      jsPsych.data.addProperties(par_info);
      jsPsych.data.get().localSave('csv', 'data.csv');
    },
  });
</script>

</html>
```

</div></details>

# おわりに

今回は参加者情報の取得について紹介しました。次回は課題の教示（と注視点）です。

[【第 7 回】教示と練習課題](https://qiita.com/snishym/items/2296fc1aff6d711aaa0b)
